# Harness Workflow Design — Agents, Calls, Skills, Data, Guardrails

> **Version:** 1.0 | **Date:** 2026-06-11 | **Canonical roster:** [`agents-manifest.yaml`](agents-manifest.yaml)
> **Companions:** [`user-guide.md`](user-guide.md) · [`agents-manifest-diagram.html`](agents-manifest-diagram.html) · design source `../2026-06-01-agentic-harness-design.md`

This document is the operational view of the harness: **what is in use at every step** —
which agent, which A2A verbs, which MCP tools, which data artifacts, which guardrails.

---

## 1. The four surfaces

Every interaction in the harness travels over exactly one of four surfaces. Nothing else exists.

| Surface | Protocol | Who uses it | Examples |
|---|---|---|---|
| **User ⇄ Orchestrator** | A2A (console: web/CLI/IDE) | Engineer, AIPO | `project.intent`, `gate.approve`, `gate.reject` |
| **Orchestrator ⇄ Workers** | A2A envelopes (NATS/Kafka bus) | All 10 agents | `quantize.run` → `quantize.passed` |
| **Agents → Tools** | MCP (HTTP/gRPC), server-side caller binding | All agents | `neutron_converter.invoke`, `pii_scanner.scan` |
| **Agents → Kernel** | MCP (`policy.evaluate`, `manifest.read`) | All agents | policy decision before every side effect |

The A2A envelope (schema §6.1 of the design) always carries: `from`, `to`, `intent`,
`correlation_id`, **`manifest_ref`** (optimistic concurrency), `payload`, `evidence[]`.

---

## 2. End-to-end workflow — the standard run

A project ("build a conveyor defect classifier on i.MX 93") flows through eleven steps.
**Bold** = hard gate. Every step also runs the five global guardrails (§4).

| # | Step | Agent | A2A in → out | MCP tools used | Data written | Controls updated |
|---|------|-------|--------------|----------------|--------------|------------------|
| 0 | Intent intake | Orchestrator | `project.intent` → `plan.created` | `manifest.read`, `policy.evaluate`, `notify.user` | project plan | — (checks AIPO L1 Decision Record) |
| 1 | Provision | EnvironmentSteward | `env.provision` → `env.provisioned` | `eiq_hub.workspace.create`, `eiq_toolkit.pin_version`, `rbac.assign_role`, `git.repo.init`, `cve_feed.subscribe` | `eiq-toolchain.lock`, `governance-roles.yaml`, `governance/` scaffold | manifest initialised, all L1 `pending` |
| 2 | Data intake | DataCurator | `dataset.import` → `dataset.cleared` \| `dataset.blocked` | `eiq_portal.dataset.import`, `pii_scanner.scan`, `governance.write_datasheet`, `dpia.check_signature` | `datasheet-<id>.yaml`, PII report, representativeness report | DG-02/03/04, DP-01/03, FB-02 |
| 3 | Model intake | ModelRegistrar | `model.select` → `model.registered` \| `model.blocked` | `model_zoo.fetch` / `byom.import`, `sbom.generate`, `license.check_allowlist`, `eu_ai_act.classify_use` | `model-registry.yaml`, `sbom-<id>.json` | ML-01, LM-01, SR-04/05, EU-02 |
| 4 | Training | Trainer | `train.run` → `train.completed` | `eiq_portal.train`, `manifest.read` (DPIA gate), `model_card.draft`, `artifact.stamp_fingerprint` | `training-runs/run-NNN.yaml`, model card draft | ML-03/04, FB-04, DG-07, EU-05, FS-04, XT-02 |
| 5 | **Quantization (HARD GATE 1)** | QuantizationGatekeeper | `quantize.run` → `quantize.passed` \| `quantize.blocked` | `neutron_converter.invoke`, `eval.run_general`, `eval.run_safety`, `eval.run_subgroup`, `report.write` | `quant-regression-<model>.md`, signed artifact on PASS only | ML-05/06/07, FB-06, FS-05, ES-03 |
| 6 | Validation | Validator | `validate.run` → `validate.passed` \| `validate.blocked` | `eiq_portal.validate`, `fgsm.run`, `pgd.run`, `garak.run`, `shap.compute`, `fse.request_signoff` | `validation-<model>.md`, adversarial registry | SR-06/07/08, FB-04/07, LM-06/08, XT-04/05, FS-07/08/09, HO-03/04 |
| 7 | Profiling | Profiler | `profile.run` → `profile.passed` \| `profile.blocked` | `eiq_target.profile`, `power.assert_budget`, `determinism.check` | `profiling-<model>.json`, `determinism-<model>.json` | ES-03, FS-06, ML-09 |
| 8 | **Deployment (HARD GATE 2)** | DeploymentGatekeeper | `deploy.export` → `deploy.signed` \| `deploy.blocked` | `eiq_export.run`, `manifest.read`, `policy.evaluate_gate`, `artifact.sign` (exclusive), `fleet.register_version` | signed artifact, `deployment-auth-<version>.md`, fleet registry | ML-10/11/12, SR-09, HO-05/06 |
| 9 | (GenAI only) Pipeline | PipelineAssembler | `pipeline.assemble` → `pipeline.assembled` | `genai_flow.compose`, `agentic_framework.deploy`, `mcp_registry.write`, `a2a_graph.write`, `autonomy.classify` | `mcp-tool-registry.yaml`, `a2a-graph.yaml` | LM-14, HO-01/02, EU-10, FS-12, DG-09/10/11 |
| 10 | Completion | Orchestrator | → `project.complete` | `manifest.read`, `notify.user` | final manifest snapshot | — |

Between every step: the Orchestrator calls `stage.advance`, which the **Kernel state
machine** grants only when the previous stage's controls are `complete`.

---

## 3. The per-call micro-workflow (every worker, every action)

Every worker action follows the same seven beats — this IS the harness:

```
1. RECEIVE   A2A envelope from Orchestrator
2. VERIFY    manifest_ref == HEAD            → else ManifestStale (re-read, retry)
             caller in agents_authorized      → else UnauthorizedCaller (tamper alert)
             policy.lock matches pinned bundle→ else PolicyVersionMismatch
3. EVALUATE  policy.evaluate(action, context) BEFORE any side effect
             → BLOCK?  abort, emit BlockedByPolicy + remediation hint
4. ACT       call the eIQ MCP tool (the actual work)
5. EVIDENCE  write reports/datasheets/logs to governance/ (typed artifacts)
6. PROPOSE   emit manifest delta — the KERNEL validates and commits (sole writer)
7. RESPOND   A2A response with evidence paths + fresh manifest_ref
```

Audit: beats 2, 3, 6 and 7 each append to the hash-chained audit log
(`governance/audit/YYYY-MM-DD.jsonl`) — policy decision, manifest write, A2A message.

---

## 4. Guardrails in use — complete inventory

### 4.1 Global (every agent, every call)

| ID | Direction | Check | On failure |
|----|-----------|-------|-----------|
| G-IN-1 | inbound | `manifest_ref` matches manifest HEAD | `ManifestStale` → re-read + re-evaluate |
| G-IN-2 | inbound | caller authorised for each MCP tool | `UnauthorizedCaller` → tamper alert |
| G-IN-3 | inbound | policy bundle version matches `policy.lock` | `PolicyVersionMismatch` → refuse |
| G-OUT-1 | outbound | `policy.evaluate` precedes every side effect | `BLOCK` → `BlockedByPolicy` + hint |
| G-OUT-2 | outbound | manifest written only by Kernel | delta rejected / `ManifestTampered` |

### 4.2 The two hard gates (cannot be skipped — state-machine enforced)

| Gate | Agent | Blocks when | What survives a FAIL |
|------|-------|-------------|---------------------|
| **L4→L5** | QuantizationGatekeeper | `general_drop > 2%` · `safety_discrepancies > 0` · `subgroup_disp > 2%` | nothing — artifact cannot advance |
| **L6→L7** | DeploymentGatekeeper | any L1–L5 🔴 control ≠ `complete` · missing auth form · missing 4-party sign-off (safety) · kill-switch untested | an **unsigned test artifact** at most |

### 4.3 Non-bypassable cross-agent rules

- **DPIA gate** (DataCurator + Trainer): training cannot start while `DP-01` ≠ complete.
- **One signed-artifact path**: `artifact.sign` is scoped to DeploymentGatekeeper only.
- **Reproducibility**: Trainer/QuantizationGatekeeper/Profiler must stamp toolchain fingerprint + dataset hash (+ seed).
- **Safety-function trigger** (`sil ≥ 1`): FSE sign-off at L5, determinism N=10 at L6, 4-party authorization at L7.
- **LLM is untrusted**: every LLM worker's *proposed action* goes through the same `policy.evaluate` — the Kernel is the trust anchor, not the model.

### 4.4 Role guardrails (who may invoke what)

| A2A verb | Required caller |
|----------|-----------------|
| `env.provision` | DSL or AIPO |
| `dataset.import` | MRO or DPO |
| `train.run`, `validate.run` | MRO (FSE for safety review) |
| `quantize.run`, `deploy.export` | **Orchestrator only** (never a user directly) |
| `pipeline.assemble` | AIPO |
| `gate.approve` / `gate.reject` | role owning the gated control (RACI §10) |

---

## 5. Failure paths (what the user sees)

| Event | Surface behaviour |
|-------|-------------------|
| `dataset.blocked` / `model.blocked` / `*.blocked` | Orchestrator relays the block reason + remediation hint; project holds at that stage |
| `BlockedByPolicy` (any side effect) | identical — plus the violated Rego rule is named in the audit log |
| `gate.request` rejected by approver | project halted; manifest snapshot preserved |
| `ManifestTampered` (manual manifest edit) | project halted pending reconciliation; Kernel refuses all writes |
| `env.vulnerable` (CVE mid-project) | Orchestrator pauses project, requests upgrade plan, blocks stage advances |
| Worker crash | no state lost — A2A envelope durable on bus, worker is stateless, replay on recovery |

---

## 6. Known boundaries (by design, v1.0)

- **Build-time harness.** Runtime adjudication (SafetyAdjudicator) and post-market
  monitoring/retirement (FleetMonitor) are `proposed` in the manifest — L7/L8 evidence
  (ML-13/14/15, RC-09, DG-12, DP-10, ML-16/17) has no active agent owner yet.
- **Human sign-off authentication** (how a 4-party signature is cryptographically
  verified) is not yet specified — current design relies on RBAC + manifest rows.
- PipelineAssembler lands in build Phase 4; until then the harness itself is not
  yet governed by its own Stage-8 standard (bootstrap gap, accepted).
