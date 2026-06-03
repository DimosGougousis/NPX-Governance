# Agentic Harness Design — Building the Factory Around the eIQ Governance Workflow

> **Purpose:** Turn the governance wrappers described in `examples/ex-02-eiq-toolkit-governed-workflow.md` into a living, multi-agent system. The harness *is* the shift-left factory: every model built by any team is governed by construction because the agents that build it cannot emit an artifact that bypasses the governance hooks.
>
> **Reading order:** §1 mental model → §2 architecture → §3 nine agent roles → §4 GovernanceKernel → §5 eIQ stage integration → §6 protocol contracts → §7 runtime topology → §8 failure modes → §9 build sequence.
>
> **Companion files this design produces:** `output/harness/` (Python package), `output/harness/protocols/*.json` (A2A + MCP schemas), `output/harness/governance_kernel/` (the policy engine).

---

## 1. Mental Model — The Factory as a Multi-Agent System

The example is explicit: governance is a property of the **toolchain**, not a downstream audit. The most direct way to make this true is to **be** the toolchain — an agentic system that wraps eIQ and refuses to do work the wrapper hooks would refuse.

Three principles govern the design:

1. **No agent acts without a governance trace.** Every tool call, every file write, every model export passes through the `GovernanceKernel`. If a hook would block the action, the agent receives a `BlockedByPolicy` exception and an obligation to either remediate the missing evidence or escalate.
2. **A2A + MCP are the only inter-agent contracts.** No ad-hoc REST. The same standards the eIQ Agentic AI Framework uses for end-user pipelines (A2A for agent-to-agent, MCP for tool/data invocation) are used internally by the harness. This means the harness is **also** the reference implementation of what Stage 8 of ex-02 requires from a customer pipeline.
3. **Engineers don't talk to agents — they supervise the Orchestrator.** The user-facing surface is a single `Orchestrator` agent that decomposes a project intent ("build a conveyor defect classifier on i.MX 93") into the eIQ stage plan and dispatches stage-specific worker agents. Engineers see the governance manifest evolve in real time and approve gates, just as the example's CI gate approves manifests.

```
            ┌──────────────────────────────────────────────────────┐
            │                  Engineer / AIPO                      │
            │     (intent, stage approvals, role sign-offs)         │
            └────────────────────┬─────────────────────────────────┘
                                 │  A2A
                                 ▼
   ┌─────────────────────────────────────────────────────────────┐
   │                   Orchestrator Agent                         │
   │  - parses intent → eIQ stage plan                           │
   │  - dispatches Stage Workers                                  │
   │  - enforces stage ordering (gates cannot be skipped)         │
   │  - surfaces Governance Manifest to user for approval         │
   └────┬───────┬───────┬───────┬───────┬───────┬───────┬───────┬─┘
        │       │       │       │       │       │       │       │
   A2A  │ S0    │ S1    │ S2    │ S3    │ S4    │ S5    │ S6    │ S7
        ▼       ▼       ▼       ▼       ▼       ▼       ▼       ▼
   ┌────────┐ ...                                                  ┌────────┐
   │Workers │  one per eIQ stage (Stages 0-7)                       │ Deploy │
   │        │                                                      │ Worker │
   └────┬───┘                                                      └────┬───┘
        │ every action                                                │
        ▼                                                             ▼
   ┌────────────────────────────────────────────────────────────────────┐
   │                       GovernanceKernel                            │
   │  policy engine + manifest ledger + hook library + CI gate logic   │
   └────────────────────────────────────────────────────────────────────┘
        │
        ▼
   ┌────────────────────────────────────────────────────────────────────┐
   │                       eIQ Toolchain                                │
   │  (Toolkit, Portal, Neutron Converter, GenAI Flow, Agentic FW)     │
   └────────────────────────────────────────────────────────────────────┘
```

---

## 2. Architecture — Three Layers, Nine Agents, One Kernel

| Layer | Component | Role |
|------|----------|------|
| **L0 — Tooling** | eIQ Toolkit, Neutron Converter, GenAI Flow, Agentic AI Framework, MLflow, Vector DB | The substrate. The harness does not replace them; it drives them. |
| **L1 — Governance** | `GovernanceKernel` (policy engine + manifest ledger + hooks) | The wrapper. Every other layer funnels through it. |
| **L2 — Agentic** | 9 agents (Orchestrator + 7 stage workers + Deploy worker) | The factory. Each is a single-responsibility agent with explicit A2A and MCP interfaces. |
| **L3 — User** | Engineer / AIPO console (CLI, web UI, IDE plugin) | The supervisor. Approves gates, signs evidence, owns RACI outcomes. |

**Why nine, not one mega-agent:** single-responsibility mirrors the eIQ stage decomposition in ex-02 §"The eIQ Workflow Mapped to the Governance Lifecycle". A monolithic agent hides which stage is non-compliant. Stage-scoped agents make the manifest legible: every status update names its stage.

---

## 3. The Nine Agents

Each agent has: a **mission**, an **A2A interface** (the calls it accepts from the Orchestrator and other workers), an **MCP tool registry** (the tools it may invoke, per least-privilege), and a **stage ownership** that maps to manifest controls.

### 3.1 Orchestrator (the conductor)

| Field | Value |
|------|------|
| **Mission** | Translate a project intent into the eight-stage eIQ plan, dispatch workers, gate stage transitions, surface the manifest. |
| **Stage ownership** | None directly; owns the cross-stage `governance-manifest.yaml` and the project plan. |
| **A2A inputs** | `project.intent`, `evidence.submit`, `gate.approve`, `gate.reject`, `worker.report` |
| **A2A outputs** | `plan.created`, `stage.dispatched`, `gate.request`, `manifest.updated`, `project.complete` |
| **MCP tools** | `manifest.read`, `manifest.write`, `policy.evaluate`, `a2a.dispatch`, `notify.user` |
| **Failure mode** | Halts the project on a rejected gate; cannot advance stage N+1 until stage N controls are green. |

The Orchestrator is the **only** agent that talks to the user. Engineers see one conversation, not eight.

### 3.2 EnvironmentSteward (Stage 0)

| Field | Value |
|------|------|
| **Mission** | Provision a governed eIQ baseline (pinned toolchain, AI Hub workspace, RBAC mapping, scaffolded `governance/` tree). |
| **Stage ownership** | L1 + cross-cutting (DG-01, SR-01, RC-01, HO-01 evidence prerequisites). |
| **A2A inputs** | `env.provision` |
| **A2A outputs** | `env.provisioned` (returns `eiq-toolchain.lock`, `governance-roles.yaml`, `governance/` scaffold) |
| **MCP tools** | `eiq_hub.workspace.create`, `eiq_toolkit.pin_version`, `rbac.assign_role`, `git.repo.init`, `cve_feed.subscribe` |
| **Governance hooks fired** | Initializes `governance-manifest.yaml`; sets up the toolchain-fingerprint stamper. |

### 3.3 DataCurator (Stage 1)

| Field | Value |
|------|------|
| **Mission** | Wrap eIQ Portal Dataset Curator: datasheet enforcement, PII/biometric scan, representativeness report, DPIA gate. |
| **Stage ownership** | L2 (DG-02/03/04, DP-01/03, FB-02). |
| **A2A inputs** | `dataset.import` (carries source, license, classes) |
| **A2A outputs** | `dataset.cleared` (carries datasheet path, PII report, `requires_dpia` flag) or `dataset.blocked` (carries reason) |
| **MCP tools** | `eiq_portal.dataset.import`, `pii_scanner.scan`, `governance.write_datasheet`, `dpia.check_signature` |
| **Failure mode** | Refuses to emit `dataset.cleared` if license is non-allow-listed, PII is present without a signed DPIA, or representativeness thresholds are violated. |

### 3.4 ModelRegistrar (Stage 2)

| Field | Value |
|------|------|
| **Mission** | Wrap eIQ Model Zoo / BYOM: registry entry, SBOM generation, license + EU AI Act class check. |
| **Stage ownership** | L1 + L3 (LM-01, SR-04/05, EU-02, ML-01). |
| **A2A inputs** | `model.select` (model_id or BYOM payload) |
| **A2A outputs** | `model.registered` (carries registry entry, SBOM) or `model.blocked` |
| **MCP tools** | `model_zoo.fetch`, `byom.import`, `sbom.generate`, `license.check_allowlist`, `eu_ai_act.classify_use` |
| **Failure mode** | Refuses to register a BYOM model whose `sha256` doesn't match the recorded value, whose license is non-commercial when the intended use is commercial, or whose intended use is on the prohibited registry. |

### 3.5 Trainer (Stage 3)

| Field | Value |
|------|------|
| **Mission** | Wrap eIQ Portal Trainer: enforce DPIA gate, auto-capture run log, draft model card. |
| **Stage ownership** | L3 (ML-03/04, FB-04, DG-07, EU-05, FS-04, XT-02). |
| **A2A inputs** | `train.run` (dataset_id, model_id, hyperparameters) |
| **A2A outputs** | `train.completed` (carries run log path, model card draft path, output artifact hash) |
| **MCP tools** | `eiq_portal.train`, `manifest.read` (DPIA check), `model_card.draft`, `artifact.stamp_fingerprint` |
| **Failure mode** | Refuses to start training if `DP-01` is not `complete` for the dataset. Refuses to emit `train.completed` if reproducibility metadata (seed, framework version, dataset hash) is missing. |

### 3.6 QuantizationGatekeeper (Stage 4 — the highest-leverage worker)

| Field | Value |
|------|------|
| **Mission** | Wrap Neutron Converter with the three mandatory regressions: general accuracy (ML-05), safety vectors (ML-06/FS-05), subgroup fairness (FB-06). |
| **Stage ownership** | L4 (ML-05/06/07, FB-06, FS-05, ES-03). |
| **A2A inputs** | `quantize.run` (model, target, calibration set, test sets) |
| **A2A outputs** | `quantize.passed` (carries three regression reports) or `quantize.blocked` (carries which regression failed) |
| **MCP tools** | `neutron_converter.invoke`, `eval.run_general`, `eval.run_safety`, `eval.run_subgroup`, `report.write` |
| **Failure mode** | **Non-zero exit on any failed regression.** This is the single most important integration point in the harness: the artifact physically cannot advance to validation without a green gate. The hook is the one ex-02 says to do "if you can only do one thing." |

### 3.7 Validator (Stage 5)

| Field | Value |
|------|------|
| **Mission** | Wrap eIQ Portal Validate: subgroup confusion matrices (FB-04), adversarial probes (SR-06 / LM-06), explainability validation (XT-05), and — for safety functions — FSE sign-off gate. |
| **Stage ownership** | L5 (SR-06/07/08, FB-04/07, LM-06/08, XT-04/05, FS-07/08/09, HO-03/04). |
| **A2A inputs** | `validate.run` (model, test sets, safety function classification) |
| **A2A outputs** | `validate.passed` (carries all reports) or `validate.blocked` |
| **MCP tools** | `eiq_portal.validate`, `fgsm.run`, `pgd.run`, `garak.run` (for LLMs), `shap.compute`, `fse.request_signoff` |
| **Failure mode** | Refuses to emit `validate.passed` for any model classified SIL ≥ 1 without FSE sign-off in the manifest. |

### 3.8 Profiler (Stage 6)

| Field | Value |
|------|------|
| **Mission** | Wrap on-device profiling: latency/memory/power capture, power-budget assertion (ES-01/03), determinism check for safety functions (FS-06). |
| **Stage ownership** | L5 + L6 (ES-03, FS-06, ML-09). |
| **A2A inputs** | `profile.run` (model, target hardware, safety-class flag) |
| **A2A outputs** | `profile.passed` or `profile.blocked` |
| **MCP tools** | `eiq_target.profile`, `power.assert_budget`, `determinism.check` (10× same input) |
| **Failure mode** | Refuses to emit `profile.passed` if inference power exceeds the L1 budget, or if determinism variance exceeds the SIL-dependent threshold. |

### 3.9 DeploymentGatekeeper (Stage 7 — the second hard gate)

| Field | Value |
|------|------|
| **Mission** | Wrap the export-to-inference-engine step: read the manifest, refuse to produce a deployable artifact unless L1–L5 critical controls are green, sign and authorize. |
| **Stage ownership** | L6 (ML-10/11/12, SR-09, HO-05/06). |
| **A2A inputs** | `deploy.export` (model, target engine) |
| **A2A outputs** | `deploy.signed` (carries signed artifact, deployment authorization record) or `deploy.blocked` (lists which controls failed) |
| **MCP tools** | `eiq_export.run`, `manifest.read`, `policy.evaluate_gate`, `artifact.sign`, `fleet.register_version` |
| **Failure mode** | This is the **only** agent that may emit a signed, deployable artifact. If `policy.evaluate_gate` returns `BLOCKED`, the export step returns an unsigned test artifact at most. The 4-party authorization for safety-critical models (per ex-03) is enforced here. |

### 3.10 PipelineAssembler (Stage 8, optional, GenAI/Agentic only)

| Field | Value |
|------|------|
| **Mission** | Assemble GenAI Flow / Agentic AI Framework pipelines with MCP tool registry and A2A graph governance. |
| **Stage ownership** | L1 + L3 + L6 (LM-14, HO-01/02, EU-10, FS-12, DG-09/10/11). |
| **A2A inputs** | `pipeline.assemble` (corpus, agents, tools) |
| **A2A outputs** | `pipeline.assembled` (carries `mcp-tool-registry.yaml`, `a2a-graph.yaml`, autonomy classification) |
| **MCP tools** | `genai_flow.compose`, `agentic_framework.deploy`, `mcp_registry.write`, `a2a_graph.write`, `autonomy.classify` |
| **Failure mode** | Refuses to assemble if any tool an agent calls is not in the MCP registry with explicit `scope` and `agents_authorized`, or if any A2A edge lacks an explicit authority level. |

---

## 4. GovernanceKernel — The Policy Engine

The Kernel is the only piece of the harness that has authority. Everything else is a client of it. Five components:

### 4.1 Policy Evaluator

A pure function: `(action, context) → Decision ∈ {ALLOW, BLOCK, REQUIRE_EVIDENCE, REQUIRE_APPROVAL}`. The policy is expressed in Rego (OPA) so it is auditable and version-controlled, but any equivalent DSL works. Policies are evaluated against the **manifest** plus the **proposed action**.

```rego
# Example policy: refuse quantization without Stage 1 datasheet
package harness.policy

deny[msg] {
    input.action == "quantize.run"
    not input.manifest.stages.L2_data.controls.DG_02.status == "complete"
    msg := sprintf("Quantization blocked: DG-02 (datasheet) not complete for dataset %v", [input.dataset_id])
}

deny[msg] {
    input.action == "quantize.run"
    input.manifest.stages.L2_data.controls.DG_02.status == "complete"
    input.dataset.contains_pii == true
    not input.manifest.stages.L1_conception.controls.DP_01.status == "complete"
    msg := "Quantization blocked: dataset contains PII but DP-01 (DPIA) not complete"
}

deny[msg] {
    input.action == "deploy.export"
    some stage, controls in CRITICAL_CONTROLS
    some cid in controls
    not input.manifest.stages[stage].controls[cid].status == "complete"
    msg := sprintf("Deployment blocked: %v/%v not complete", [stage, cid])
}
```

### 4.2 Manifest Ledger

The single source of truth. Implemented as a Git-versioned YAML file (`governance/governance-manifest.yaml`) with optimistic concurrency: every write carries a manifest version hash; a stale write returns `ManifestStale`. The Kernel is the only writer. This is the same pattern ex-01's Automation Playbook uses; the harness formalizes it.

### 4.3 Hook Library

Each eIQ stage hook from ex-02 is implemented as an MCP tool the worker agents call, rather than scripts the engineer runs. The mapping is direct:

| ex-02 hook | Harness MCP tool | Worker |
|-----------|------------------|--------|
| `dataset_intake_gate.py` | `governance.write_datasheet`, `pii_scanner.scan`, `dpia.check_signature` | DataCurator |
| `model_intake_gate.py` | `sbom.generate`, `license.check_allowlist`, `eu_ai_act.classify_use` | ModelRegistrar |
| `capture_training_run.py` | `model_card.draft`, `artifact.stamp_fingerprint` | Trainer |
| `quantization_regression_test.py` | `eval.run_general`, `eval.run_safety`, `eval.run_subgroup` | QuantizationGatekeeper |
| `adversarial_probe.py` | `fgsm.run`, `pgd.run`, `garak.run` | Validator |
| `determinism_check.py` | `determinism.check` | Profiler |
| `check_governance_gate.py` | `policy.evaluate_gate` | DeploymentGatekeeper |
| `mcp-tool-registry.yaml` | `mcp_registry.write` | PipelineAssembler |
| `a2a-graph.yaml` | `a2a_graph.write` | PipelineAssembler |

This is the leverage: every governance hook from ex-02 becomes a typed MCP tool with a JSON schema, a Rego policy, and a manifest update. Engineers write code; the agents call hooks; the Kernel decides.

### 4.4 Stage State Machine

A formal model of stage ordering. The Orchestrator is the only agent allowed to call `stage.advance`, and it is governed by the state machine: `L1 → L2 → ... → L7` (L8 in ex-02 is `PipelineAssembler` or retirement). `L4 → L5` is gated by `quantize.passed`; `L5 → L6` by `validate.passed`; `L6 → L7` (deploy) by `deploy.passed`. The state machine is itself a Rego policy.

### 4.5 Audit Log

Every policy decision, every manifest write, every MCP call, every A2A message is appended to an immutable, hash-chained log (`governance/audit/YYYY-MM-DD.jsonl`). This satisfies D7 (XT-01/02 audit trail), supports post-hoc investigation, and produces the evidence package for the EU AI Act AIMS audit (Tier 1).

---

## 5. eIQ Stage Integration — Where the Hooks Live

The harness is a **wrapper** in the same sense ex-02 describes. The mapping from agent to eIQ component is direct:

| Worker Agent | eIQ Component | What the worker does |
|--------------|---------------|---------------------|
| EnvironmentSteward | eIQ AI Hub / Modular Toolkit | Provisions workspace, pins versions, sets RBAC, scaffolds `governance/` |
| DataCurator | eIQ Portal Dataset Curator | Wraps import, writes datasheet, runs PII scan, gates DPIA |
| ModelRegistrar | eIQ Model Zoo / BYOM import | Wraps model selection, generates registry + SBOM |
| Trainer | eIQ Portal Trainer | Wraps training, captures run log, drafts model card |
| QuantizationGatekeeper | Neutron Converter | Wraps conversion, runs the three regressions, blocks on failure |
| Validator | eIQ Portal Validate | Wraps validation, adds adversarial probes, requests FSE sign-off for safety functions |
| Profiler | eIQ on-device profiler | Wraps profiling, asserts power budget, runs determinism check |
| DeploymentGatekeeper | eIQ export to inference engine | Wraps export, evaluates gate, signs artifact, authorizes deployment |
| PipelineAssembler | eIQ GenAI Flow / Agentic AI Framework | Wraps pipeline assembly, validates MCP and A2A artifacts |

Every worker agent has the same skeleton: it accepts an A2A request from the Orchestrator, calls the appropriate eIQ MCP tool (or shell-out to the CLI for the modular toolkit), calls the GovernanceKernel's hook library to evaluate policy, and emits an A2A response carrying the manifest updates.

---

## 5.1 Agent Quick-Reference — Actions, Rules, Checks, Blocks

One row per agent. Read columns left to right: **what it does** → **the rules it operates under** → **the checks it runs on the way in** → **the checks it runs on the way out** → **what makes it block**. Every agent runs the *Global checks* on every A2A call before and after any action; the *Stage checks* are agent-specific.

### Global checks (apply to every agent)

| When | Check | On failure |
|------|------|-----------|
| Inbound | `manifest_ref` in A2A envelope matches current `HEAD` of `governance-manifest.yaml` | Return `ManifestStale`; re-read manifest; re-evaluate policy |
| Inbound | Caller's `agents_authorized` list contains the calling agent for the requested MCP tool | Return `UnauthorizedCaller`; Orchestrator receives a tamper alert |
| Inbound | `policy.lock` version matches the worker's pinned policy bundle | Return `PolicyVersionMismatch`; refuse until reconciled |
| Outbound | Proposed action is sent to `policy.evaluate` *before* the side effect | If `BLOCK` → abort action, emit `BlockedByPolicy`, attach remediation hint |
| Outbound | Manifest write is performed by the Kernel, not the worker | Worker only emits a *proposed* delta; Kernel validates and commits |

### Per-agent table

| # | Agent | Stage | What it does (1 line) | Rules it operates under | Checks on the way in | Checks on the way out | What blocks it (concrete failure modes) |
|---|-------|------|----------------------|------------------------|----------------------|-----------------------|--------------------------------------|
| 1 | **Orchestrator** | cross-stage | Translates intent into the 8-stage plan, dispatches workers, gates stage transitions, surfaces manifest. | (1) Stage ordering is a state machine; (2) one stage must be `complete` before the next dispatches; (3) only the Orchestrator talks to the user. | (a) `project.intent` carries use case, autonomy level, EU AI Act class hints; (b) `governance-manifest.yaml` exists and is schema-valid; (c) AIPO sign-off on the L1 Decision Record is present. | (a) Every dispatch carries a fresh `manifest_ref`; (b) the manifest is re-snapshotted on every stage transition; (c) user is notified on every `gate.request` and `manifest.updated`. | (i) A previous stage has any 🔴 control with status ≠ `complete`; (ii) `gate.request` receives `reject` from the user; (iii) any worker emits `BlockedByPolicy` that the user does not remediate; (iv) manifest fails schema validation. |
| 2 | **EnvironmentSteward** | 0 | Provisions a governed eIQ baseline: pinned toolchain, AI Hub workspace, RBAC mapping, scaffolded `governance/` tree. | (1) Every toolchain version is pinned in `eiq-toolchain.lock`; (2) RBAC roles match the framework's role definitions exactly; (3) CVE feed subscription is mandatory before workspace is marked provisioned. | (a) Caller has `DSL` or `AIPO` role; (b) workspace name does not collide; (c) target NXP hardware family is in the supported list. | (a) `eiq-toolkit --version`, `neutron-converter --version` and inference-engine versions are recorded; (b) `governance/` tree is created from the canonical template; (c) `governance-manifest.yaml` is initialized with all L1 controls `pending`; (d) CVE feed subscription is live. | (i) Locked file already exists (refuse to overwrite without explicit `force`); (ii) RBAC role names do not match the framework vocabulary; (iii) CVE feed endpoint unreachable; (iv) AI Hub quota exceeded. |
| 3 | **DataCurator** | 1 | Wraps dataset import: datasheet enforcement, PII/biometric scan, representativeness report, DPIA gate. | (1) License must be in the allow-list before import proceeds; (2) any PII above 0% requires a signed DPIA; (3) class imbalance below threshold triggers a warning but is non-blocking unless `is_safety_function=true`. | (a) Caller has `MRO` or `DPO` role; (b) `datasheet.yaml` template fields are all present; (c) PII scan report is < 7 days old; (d) DPIA file path is provided when `contains_pii=true`. | (a) Datasheet is written to `governance/data/datasheet-<id>.yaml`; (b) representativeness report (`min_class_pct`, distribution) is written; (c) manifest's `DG-02`, `DG-04`, `FB-02`, `DP-01` are updated; (d) `requires_dpia` flag is propagated to the dataset registry. | (i) License is not in allow-list; (ii) PII scan finds biometric data with `pii_ratio > 0` and no signed DPIA; (iii) datasheet is missing required fields; (iv) class distribution has a class at < 1% *and* the use case is a safety function. |
| 4 | **ModelRegistrar** | 2 | Wraps model selection: registry entry, SBOM generation, license + EU AI Act class check. | (1) Model Zoo picks inherit NXP provenance; (2) BYOM requires `source_url`, `sha256`, and `license` and they must all be consistent; (3) intended use must not be on the prohibited-use registry. | (a) For Model Zoo: caller specifies `model_id`; (b) for BYOM: caller provides `source_url`, `sha256`, `license`; (c) intended use is provided. | (a) Registry entry written to `governance/model-registry.yaml`; (b) SBOM written to `governance/sbom-<id>.json` (D3 SR-04); (c) manifest's `ML-01`, `LM-01`, `SR-04`, `EU-02` are updated; (d) license-compatibility boolean is recorded. | (i) BYOM `sha256` does not match the value the caller claims; (ii) license is non-commercial but intended use is commercial; (iii) intended use is on the EU AI Act prohibited registry; (iv) framework version in SBOM has an unpatched critical CVE. |
| 5 | **Trainer** | 3 | Wraps training: enforces DPIA gate, auto-captures run log, drafts model card. | (1) Training refuses to start if dataset's `DP-01` is not `complete`; (2) every run is fully reproducible (seed, framework version, dataset hash, hyperparameters); (3) the model card draft is auto-populated, never hand-edited at draft stage. | (a) Caller has `MRO` role; (b) dataset and model IDs both exist in the manifest; (c) `DP-01` is `complete` for the dataset; (d) `seed` is provided; (e) hardware target is set. | (a) `governance/training-runs/run-NNN.yaml` is written with dataset hash, model ID, hyperparameters, seed, toolchain fingerprint, output artifact hash; (b) model card draft written to `governance/model-cards/<model>.md`; (c) manifest's `ML-03`, `ML-04`, `FB-04`, `DG-07`, `EU-05`, `FS-04`, `XT-02` are updated. | (i) Dataset's `DP-01` is not `complete`; (ii) reproducibility metadata is missing; (iii) framework version is unpatched critical CVE; (iv) training run is requested against a model not in the registry. |
| 6 | **QuantizationGatekeeper** | 4 — **hard gate** | Wraps Neutron Converter with three mandatory regressions: general accuracy, safety vectors, subgroup fairness. | (1) Non-zero exit on any failed regression; (2) all three regressions are mandatory, not configurable; (3) calibration set is required; (4) the converted artifact physically cannot advance without `quantize.passed`. | (a) Caller is the Orchestrator (not a user); (b) training run artifact exists and is signed; (c) general, safety, and subgroup test sets are all provided and non-empty; (d) target NPU is specified; (e) calibration set is provided. | (a) `governance/reports/quant-regression-<model>.md` is written with three pass/fail rows; (b) quantization config (scheme, excluded layers, calibration set hash) is recorded in the model card; (c) manifest's `ML-05`, `ML-06`, `ML-07`, `FB-06`, `FS-05`, `ES-03` are updated; (d) signed converted artifact is produced only on `PASS`. | (i) **Any** of: `general_drop > 2%`, `safety_discrepancies > 0`, `subgroup_disp > 2%`; (ii) safety test set is empty; (iii) calibration set is missing; (iv) target NPU is not in the supported list. **Any of these blocks the artifact from advancing to Stage 5.** |
| 7 | **Validator** | 5 | Wraps validation: subgroup confusion matrices, adversarial probes, explainability validation, FSE sign-off for safety functions. | (1) Confusion matrix is disaggregated by subgroup, not just overall; (2) adversarial probe suite runs to completion; (3) any model with `sil ≥ 1` requires FSE sign-off before `validate.passed` may be emitted. | (a) Caller has `MRO` role (or `FSE` for safety review); (b) quantized artifact exists; (c) test sets are present; (d) for safety functions: `FS-04` is `complete` in the manifest. | (a) `governance/reports/validation-<model>.md` written with per-subgroup confusion matrix; (b) adversarial / prompt-injection registry is updated (D3 SR-06, D2 LM-06); (c) manifest's `SR-06/07/08`, `FB-04/07`, `LM-06/08`, `XT-04/05`, `FS-07/08/09`, `HO-03/04` are updated; (d) FSE sign-off row is written for safety functions. | (i) Per-subgroup disparity exceeds the threshold; (ii) adversarial probe produces a successful prompt injection; (iii) safety function without FSE sign-off; (iv) test set is older than the model artifact. |
| 8 | **Profiler** | 6 | Wraps on-device profiling: latency/memory/power capture, power-budget assertion, determinism check for safety functions. | (1) Inference power is asserted against the L1 power budget; (2) determinism check runs N=10 times for any safety function; (3) peak memory is asserted against the target's memory budget. | (a) Quantized + validated artifact exists; (b) target hardware unit is available and registered; (c) power budget is set in `L1_conception.controls.ES_01`; (d) `is_safety_function` flag is set if determinism check is required. | (a) `governance/reports/profiling-<model>.json` written (latency p50/p95/p99, peak memory, NPU avg power); (b) for safety functions: `governance/reports/determinism-<model>.json` with max output variance; (c) manifest's `ES-03`, `FS-06`, `ML-09` are updated; (d) variance row is recorded against the SIL-dependent threshold. | (i) Inference power exceeds the L1 power budget; (ii) determinism variance exceeds the SIL-dependent threshold; (iii) peak memory exceeds the target's memory budget; (iv) hardware unit is not in the registered device set. |
| 9 | **DeploymentGatekeeper** | 7 — **hard gate** | Wraps export-to-inference-engine: reads the manifest, refuses to produce a signed deployable artifact unless L1–L5 critical controls are green, signs and authorizes. | (1) This is the **only** agent that may emit a signed, deployable artifact; (2) 4-party authorization (AIPO + MRO + DSL + FSE) is required for safety-critical models; (3) per-device version registration is mandatory. | (a) Caller is the Orchestrator; (b) all L1–L5 critical controls are `complete`; (c) deployment authorization form is committed to the repo; (d) for safety functions, 4-party sign-off rows are present. | (a) Artifact is signed with the fleet signing key (D3 SR-09); (b) `governance/deployment-auth-<version>.md` is written (D9 ML-10, ML-11); (c) each deployed artifact version is registered per device in the fleet system (D9 ML-12); (d) manifest's `ML-10`, `ML-11`, `ML-12`, `SR-09`, `HO-05`, `HO-06` are updated; (e) at most an **unsigned** test artifact is produced when the gate fails. | (i) **Any** L1–L5 🔴 control is not `complete` (this is the single most important integration point); (ii) deployment authorization form is missing or unsigned; (iii) for safety functions: any of the four sign-offs missing; (iv) kill-switch test (`HO-05`) has not passed. |
| 10 | **PipelineAssembler** | 8 (GenAI/Agentic) | Assembles GenAI Flow / Agentic pipelines with MCP tool registry and A2A graph governance. | (1) Every tool an agent may call is in the MCP registry with explicit `scope` and `agents_authorized`; (2) every A2A edge has explicit authority and escalation rules; (3) RAG corpus governance (ex-01 pattern) and autonomy classification (ex-03 pattern) apply. | (a) Caller has `AIPO` role; (b) corpus curation policy exists; (c) MCP tool registry schema is valid; (d) A2A graph schema is valid; (e) autonomy level is classified. | (a) `governance/agentic/mcp-tool-registry.yaml` and `a2a-graph.yaml` are written; (b) manifest's `LM-14`, `HO-01/02`, `EU-10`, `FS-12`, `DG-09/10/11` are updated; (c) prohibited-decisions registry check passes. | (i) An MCP tool an agent calls is not in the registry; (ii) an A2A edge has no explicit authority; (iii) a tool with `scope: prohibited` is invoked by any agent; (iv) RAG corpus contains expired documents; (v) autonomy classification is missing. |

### Cross-agent rules (apply to multiple agents)

| Rule | Agents it applies to | Concretely |
|------|---------------------|-----------|
| **Manifest is the only ledger.** | All 10 | Workers never edit the manifest directly; the Kernel does, in response to a worker's *proposed* delta. |
| **`manifest_ref` on every A2A call.** | All 10 | Stale-ref calls return `ManifestStale`; no worker acts on outdated gate state. |
| **One signed-artifact path.** | DeploymentGatekeeper only | All other workers produce *test* artifacts at most. The signed path is policy-gated. |
| **DPIA gate is non-bypassable.** | DataCurator, Trainer | `DP-01` must be `complete` in the manifest before the Trainer starts. |
| **Two hard gates cannot be skipped.** | QuantizationGatekeeper, DeploymentGatekeeper | The Stage state machine refuses to advance past L4 or L6 without their `*.passed` events. |
| **Reproducibility metadata is mandatory.** | Trainer, QuantizationGatekeeper, Profiler | Every output artifact carries a toolchain fingerprint, dataset hash, and (for Trainer) seed. |
| **Safety-function trigger is explicit.** | Validator, Profiler, DeploymentGatekeeper | A model with `sil ≥ 1` flips on FSE-required checks at L5, determinism at L6, and 4-party authorization at L7. |
| **Adversarial robustness applies to GenAI too.** | Validator, PipelineAssembler | CV models get FGSM/PGD; LLM models get the `garak` prompt-injection suite; the same `SR-06` control covers both. |

### How to read this table in practice

- **Designing a new agent?** Copy the row shape: *what it does → rules → in-checks → out-checks → blocks*. If you cannot fill the *blocks* column with concrete failure modes, the agent's policy is not specific enough.
- **Debugging a stuck project?** Walk down the relevant agent's column. If the project's manifest shows a control `in_progress`, check whether the agent's *in-checks* (e.g. DPIA signed) are met. If all in-checks pass but the agent still blocks, look at the *out-checks* (e.g. manifest write conflict).
- **Auditing?** Every "blocks" entry should be traceable to a Rego policy in the Kernel. The table is a 1:1 map from human-readable rules to machine-readable policy.

---

## 6. Protocol Contracts — A2A and MCP Schemas

The harness consumes its own dogfood: the same A2A and MCP contracts the eIQ Agentic AI Framework requires of customer pipelines, the harness uses internally. Two schema files define the surface area:

### 6.1 A2A message envelope (JSON Schema)

```json
{
  "$id": "harness.protocols.a2a.envelope",
  "type": "object",
  "required": ["envelope_version", "from", "to", "intent", "payload", "manifest_ref"],
  "properties": {
    "envelope_version": {"const": "1.0.0"},
    "from":             {"type": "string", "description": "sender agent id"},
    "to":               {"type": "string", "description": "recipient agent id"},
    "intent":           {"type": "string", "description": "A2A verb, e.g. quantize.run"},
    "correlation_id":   {"type": "string", "description": "UUID for tracing"},
    "manifest_ref":     {"type": "string", "description": "Git ref of manifest at send time"},
    "payload":          {"type": "object"},
    "policy_decision":  {"enum": ["ALLOW", "BLOCK", "REQUIRE_EVIDENCE", "REQUIRE_APPROVAL"]},
    "evidence":         {"type": "array", "items": {"type": "string"}, "description": "paths to evidence files"}
  }
}
```

`manifest_ref` is what makes the harness safe under concurrent workers: a worker receiving an A2A request with a stale manifest ref must re-read the manifest before acting. This is the optimistic-concurrency primitive for governance.

### 6.2 MCP tool manifest

Each MCP tool the harness exposes to its workers (or to a customer pipeline built with the same kernel) carries a Rego policy binding:

```yaml
# governance/agentic/mcp-tool-registry.yaml  (excerpt — internal-facing)
mcp_tools:
  - tool: "eiq_neutron.convert"
    agents_authorized: ["QuantizationGatekeeper"]
    scope: "action_with_quantization_gate"
    policy: "package harness.policy; deny[msg] { not has_quant_reports(input) }"
    governed_by: "D9 ML-05/06, D5 FB-06, D12 FS-05"
  - tool: "manifest.write"
    agents_authorized: ["GovernanceKernel"]   # not a worker
    scope: "kernel_only"
    policy: "package harness.policy; deny[msg] { not is_kernel(input.caller) }"
```

Note the asymmetry: `manifest.write` is restricted to the Kernel itself, not to workers. Workers call `policy.evaluate` and `manifest.read`; the Kernel writes the manifest in response. This is the **only** way to make the manifest tamper-evident.

---

## 7. Runtime Topology — How the Pieces Run

A reference deployment for one project:

```text
┌─ per-project (one per active project) ──────────────────────────────────┐
│ Orchestrator (stateful: holds the manifest in memory + writes to git)    │
│ Stage workers (stateless; one process per active stage, scale to 0)      │
│ GovernanceKernel (stateful: OPA sidecar + manifest store)                │
└──────────────────────────────────────────────────────────────────────────┘
┌─ per-cluster (shared) ─────────────────────────────────────────────────┐
│ eIQ AI Hub control plane                                                  │
│ Git repo (manifest + evidence) — single source of truth                   │
│ Vector DB for RAG corpora                                                 │
│ Object store for model artifacts + audit log                              │
│ Message bus (A2A: NATS or Kafka; MCP: HTTP/gRPC)                          │
└──────────────────────────────────────────────────────────────────────────┘
┌─ per-engineer ─────────────────────────────────────────────────────────┐
│ Console (web/CLI/IDE) — talks to Orchestrator only                       │
└──────────────────────────────────────────────────────────────────────────┘
```

**Scaling rule:** workers scale horizontally per stage (a single project may have multiple Training jobs across datasets). The Orchestrator and Kernel are vertically scaled but not horizontally — they are the consistency boundaries. The audit log is append-only and sharded by date.

**State:** Orchestrator holds the in-memory manifest + plan, persists on every transition. Workers are stateless; their context is the A2A envelope + the manifest ref. Kernel is the only writer to the manifest.

**Failure isolation:** a worker crash loses no state (the A2A envelope is durable on the bus). A Kernel crash loses the in-memory policy cache only — the persisted manifest and policies on disk are the source of truth. Recovery: replay in-flight A2A envelopes from the bus, re-evaluate against the persisted manifest.

---

## 8. Failure Modes — What Breaks and How the Harness Behaves

| Failure | Symptom | Harness behavior |
|---------|---------|------------------|
| Engineer edits manifest directly | Manifest hash diverges from Kernel's view | Kernel refuses to write; next worker A2A call returns `ManifestTampered`; Orchestrator halts project pending reconciliation |
| Worker bypasses Kernel (calls eIQ directly) | Audit log gap | `eiq_*` MCP tools carry a per-call policy binding; if the call didn't come through the Kernel, the tool returns `UnauthorizedCaller`. **Enforcement point: MCP server-side, not agent-side.** This is why MCP tools are typed, not bash. |
| Stale manifest in worker | Worker acts on outdated gate state | A2A envelope carries `manifest_ref`; worker calls `manifest.read` before acting; mismatch returns `ManifestStale`, worker re-fetches and re-evaluates |
| Policy disagreement between OPA versions | Inconsistent decisions | Policies are pinned per project in `governance/policy.lock`; Kernel loads only the pinned bundle |
| Adversarial prompt injection against an LLM worker | Worker emits a malicious plan | LLM worker outputs are routed through the same Kernel: every proposed action is evaluated by the same `policy.evaluate` call. The LLM is untrusted; the Kernel is trusted. (This is also why ex-02's Stage 8 governance matters: the harness *is* an LLM agentic pipeline, so it must govern itself by the same rules.) |
| A safety function deployed without FSE sign-off | Manifest missing `FS-07` | `DeploymentGatekeeper` blocks; `policy.evaluate_gate` returns `REQUIRE_APPROVAL` until the FSE signs the manifest row. The artifact is unsigned and unusable on devices. |
| Toolchain CVE published mid-project | eIQ version flagged | `EnvironmentSteward` is subscribed to CVE feeds; on critical CVE it emits `env.vulnerable` A2A to Orchestrator; Orchestrator pauses project, requests upgrade plan, blocks further stage advances until patched (D3 SR-03). |

---

## 9. Build Sequence — Five Phases, Each Independently Useful

This mirrors ex-02's phased roadmap. The harness is itself the realization of that roadmap — each phase lands a worker + the Kernel pieces it needs.

| Phase | Scope | Kernel pieces | Workers landed | Value unlocked |
|------|------|---------------|----------------|----------------|
| **0 — Kernel skeleton** | OPA bundle, manifest ledger, audit log, A2A bus, MCP server stub | All five Kernel components at MVP depth | None yet (manual scripts call the Kernel) | One place to add policies; one place to read the manifest |
| **1 — The two hard gates** | Quantization + Deployment workers | Hook library: `quantize_regression`, `evaluate_gate` | QuantizationGatekeeper, DeploymentGatekeeper | The two highest-risk transitions are now agent-driven and kernel-governed |
| **2 — Evidence automation** | Dataset, Model, Training workers | Hook library: `datasheet`, `sbom`, `training_run` | DataCurator, ModelRegistrar, Trainer | The evidence ex-02 Stage 1–3 needs is now a byproduct of agent work |
| **3 — Validation depth** | Validation + Profiling workers | Hook library: `adversarial`, `subgroup_matrix`, `determinism` | Validator, Profiler | Fairness, robustness, safety, environmental controls operationalized as agent capabilities |
| **4 — Foundation & GenAI frontier** | Environment + Pipeline workers | Hook library: `rbac`, `cve_feed`, `mcp_registry`, `a2a_graph` | EnvironmentSteward, PipelineAssembler | The factory itself is governed; GenAI/Agentic customer pipelines are reference-implemented by the same kernel |

> **If you can only do one phase, do Phase 1.** The two hard gates are the difference between "the harness has governance" and "the harness is a wrapper without teeth."

---

## 10. RACI in the Harness

The framework's role definitions (AIPO, MRO, DSL, DPO, FSE, GC) are preserved but mapped onto agent capabilities:

| Role | In the harness | Approves |
|------|---------------|---------|
| **AIPO** | Orchestrator (intent intake, ethics gate, project sign-off) | EU-01, EU-02, HO-01, L1 Decision Record |
| **MRO** | Validator, ModelRegistrar, Trainer (model risk controls) | ML-01/03/04/05/06, FB-04, EU-05, manifest updates for L3-L5 |
| **DSL** | EnvironmentSteward, DeploymentGatekeeper (security + deployment) | SR-01/03/04/05/09, ML-10/11/12, HO-05 |
| **DPO** | DataCurator (privacy + DPIA) | DP-01/03, datasheet sign-off |
| **FSE** | Validator, Profiler (safety functions only) | FS-04/05/06/07/08, determinism sign-off |
| **GC** | Kernel maintainer (policy updates, framework evolution) | New policy versions in `policy.lock` |

A single role can be embodied by more than one agent (MRO maps to three workers), and an agent's outputs can require sign-off from a different role than the agent itself (the FSE is not a worker; the FSE *signs* the Validator's output). This separation is what makes the harness auditable.

---

## 11. What This Gets You

Concretely, the harness turns the example's shift-left thesis into a deployable system:

- **Every project starts governed.** EnvironmentSteward scaffolds `governance/` on workspace creation; the manifest is empty but typed, and every subsequent worker writes only the controls it owns.
- **No engineer can ship a model that bypasses a 🔴 control.** `DeploymentGatekeeper` is the only path to a signed artifact, and it is policy-gated. A bypass requires either (a) editing the manifest directly, which the Kernel detects, or (b) calling eIQ outside the harness, which the MCP server's caller-binding rejects.
- **Governance is reproducible.** Manifest + audit log + toolchain lock = the entire governance history of a project, replayable and auditable.
- **The same kernel governs the factory and the products.** A customer building a GenAI pipeline with the eIQ Agentic AI Framework uses the same MCP-tool-registry + A2A-graph + policy machinery the harness uses internally. Governance by construction, demonstrated.

---

## 12. Companion Files (to be produced)

This design is the entry point. The next deliverables are the implementation:

| Path | Purpose |
|------|---------|
| `output/harness/governance_kernel/policies/` | Rego policy bundles, one per stage |
| `output/harness/governance_kernel/manifest.py` | Manifest ledger (read/write/version) |
| `output/harness/governance_kernel/audit.py` | Append-only hash-chained audit log |
| `output/harness/protocols/a2a_envelope.schema.json` | A2A message schema (§6.1) |
| `output/harness/protocols/mcp_tool_registry.yaml` | MCP tool registry template (§6.2) |
| `output/harness/agents/orchestrator.py` | Orchestrator agent implementation |
| `output/harness/agents/<worker>.py` | One file per stage worker |
| `output/harness/tests/test_policy_bundle.py` | OPA policy unit tests |
| `output/harness/tests/test_stage_gates.py` | End-to-end gate tests |

---

*Design v1.0 — 2026-06-01 | Companion to `examples/ex-02-eiq-toolkit-governed-workflow.md`*
