# Harness User Guide — Building a Governed Edge AI Model

> **Version:** 1.0 | **Date:** 2026-06-11 | **Audience:** engineers, AIPO, MRO, DSL, DPO, FSE
> **Companions:** [`workflow-design.md`](workflow-design.md) · [`agents-manifest.yaml`](agents-manifest.yaml) · [`agents-manifest-diagram.html`](agents-manifest-diagram.html)

## What this is

The harness is an agent factory wrapped around the eIQ toolchain. You describe what you
want to build; ten governance-aware agents do the work; a policy kernel makes sure no
artifact can ship with an incomplete critical control. **You talk to one agent only —
the Orchestrator.** You never invoke workers directly.

Three things to internalise before your first project:

1. **Governance is not a review at the end.** Every action your project takes emits its
   own evidence. If your project reaches deployment, the audit pack already exists.
2. **Blocks are features.** When an agent refuses, it names the control and the fix.
   Remediate and re-run; nothing is lost.
3. **The manifest is the project.** `governance/governance-manifest.yaml` shows every
   control's status in real time. If you want to know "where are we?", read the manifest.

---

## Your role determines your verbs

| You are | You can | You sign |
|---------|---------|----------|
| Engineer | submit `project.intent`, watch the manifest, remediate blocks | — |
| **AIPO** | approve L1 Decision Record, `gate.approve`/`gate.reject`, authorise pipelines | EU-01/02, HO-01, deploy (safety) |
| **MRO** | import datasets, run training/validation | ML-* and FB-* manifest rows, deploy (safety) |
| **DSL** | provision environments, authorise deployments | SR-*, deploy (safety) |
| **DPO** | sign DPIAs, approve datasheets | DP-01/03 |
| **FSE** | sign safety validations and determinism reports | FS-*, deploy (safety) |

---

## Quickstart — a project in eleven moves

```text
 you > project.intent "conveyor defect classifier on i.MX 93, advisory autonomy"
```

The Orchestrator parses your intent, checks the **AIPO-signed L1 Decision Record**
(no project starts without one), and shows you the 8-stage plan. Then:

| Move | What you do | What happens behind the scenes |
|------|-------------|--------------------------------|
| 1 Provision | nothing (auto) | EnvironmentSteward pins the toolchain, scaffolds `governance/`, subscribes CVE feed |
| 2 Data | provide source, license, classes; DPO signs DPIA if PII | DataCurator writes the datasheet, scans PII, gates on DPIA |
| 3 Model | pick from Model Zoo or provide BYOM (`source_url`+`sha256`+`license`) | ModelRegistrar generates SBOM, checks license + EU AI Act class |
| 4 Train | provide hyperparameters + seed | Trainer refuses if DPIA incomplete; captures a fully reproducible run log; drafts the model card |
| 5 Quantize | nothing (auto) — **HARD GATE** | three regressions: accuracy ≤2% drop, zero safety discrepancies, ≤2% subgroup disparity |
| 6 Validate | FSE signs if safety function | subgroup confusion matrices, FGSM/PGD/garak probes, XAI validation |
| 7 Profile | nothing (auto) | latency/memory/power vs your L1 budget; determinism ×10 for safety functions |
| 8 Deploy | approve `gate.request`; 4 sign-offs if safety-critical — **HARD GATE** | DeploymentGatekeeper verifies every L1–L5 🔴 control, signs, registers per device |
| 9 (GenAI) | AIPO approves pipeline | PipelineAssembler validates MCP registry + A2A graph + autonomy class |

```text
 orchestrator > project.complete  ✓ manifest green · audit pack at governance/audit/
```

---

## When you get blocked (and you will)

Blocks always arrive with the control ID and a remediation hint. The common ones:

| Block message says | It means | Fix |
|--------------------|----------|-----|
| `DG-02 (datasheet) not complete` | dataset imported without full datasheet fields | fill the missing fields in `governance/data/datasheet-<id>.yaml`, re-import |
| `DP-01 (DPIA) not complete` | dataset has PII, no signed DPIA | get DPO sign-off; the Trainer will not start without it — there is no override |
| `license not in allow-list` | BYOM/dataset license incompatible | substitute the asset or get GC to extend the allow-list (policy change, reviewed) |
| `sha256 mismatch` | BYOM payload differs from declared hash | re-download from source; if it persists, treat as supply-chain incident |
| `quantize.blocked: safety_discrepancies > 0` | INT8/INT4 conversion changed a safety-relevant output | exclude affected layers from quantization, or re-calibrate; re-run the gate |
| `subgroup_disp > 2%` | quantization degraded one subgroup more than others | rebalance calibration set; document and re-run |
| `sil >= 1 without FSE sign-off` | safety function not signed | request FSE review via `fse.request_signoff` |
| `deploy.blocked: <stage>/<control> not complete` | a critical control upstream is open | the message lists exactly which; remediate at that stage |
| `ManifestStale` | another worker advanced the manifest mid-call | automatic retry — no action needed |
| `ManifestTampered` | someone edited the manifest by hand | don't. Restore from Git; the Kernel is the only writer |

**Escalation:** an unresolvable block goes to the role that owns the control (see RACI
in the manifest). The Orchestrator's `gate.request` carries the evidence either way —
approval is a decision, never a default.

---

## Reading the manifest

```yaml
stages:
  L4_quantization:
    controls:
      ML-05: { status: complete,   evidence: reports/quant-regression-defect.md }
      ML-06: { status: complete,   evidence: reports/quant-regression-defect.md }
      FB-06: { status: in_progress }        # <- you are here
```

- `pending` → not started · `in_progress` → agent working or waiting on you ·
  `complete` → evidence written and Kernel-committed · `blocked` → see audit log.
- Every `complete` has an evidence path. No evidence, no complete.
- The CLI `harness manifest --watch` tails status changes live.

## Audit & evidence

Everything is replayable: `governance/audit/*.jsonl` is append-only and hash-chained.
One project = one evidence pack = manifest + audit log + toolchain lock + reports.
For an EU AI Act / ISO 42001 audit, export the pack — nothing is reconstructed after
the fact.

## FAQ

**Can I skip quantization regressions for a quick demo?** No. The three regressions are
non-configurable. Use an unsigned test artifact on a dev board instead — those exist
precisely for this.

**Can I deploy with one 🟡 control open?** Yes — only 🔴 (critical) controls gate
deployment. 🟡/🟢 appear in the manifest and your risk report.

**Who can deploy a safety-critical model?** Nobody alone. AIPO + MRO + DSL + FSE must
all sign; the DeploymentGatekeeper enforces the four rows mechanically.

**What happens after deployment?** v1.0 is a build-time harness: post-market monitoring
(drift, incidents, Art. 72/73) is the `fleet-monitor` proposal in the manifest — until
it lands, L7 obligations run through your existing MLOps process.

**Can I add my own agent?** Copy a row-shape from `agents-manifest.yaml`: mission, A2A
verbs, MCP tools (least privilege), guardrails in/out, **concrete blocks**. The design
rule: *if you can't fill the blocks column, your agent's policy isn't specific enough.*
New agents enter via GC review (policy.lock bump).
