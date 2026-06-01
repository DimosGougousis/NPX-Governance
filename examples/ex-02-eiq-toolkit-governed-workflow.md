# Example 2 — Governing the eIQ AI Development Environment Itself

> **NXP Product:** eIQ AI Development Environment (eIQ Toolkit, eIQ Portal, eIQ AI Hub, Neutron Converter, Time Series Studio, GenAI Flow, Agentic AI Framework)
> **Reference:** [eIQ AI development environment](https://www.nxp.com/design/design-center/software/eiq-ai-development-environment:EIQ) · [eIQ Toolkit](https://www.nxp.com/design/design-center/software/eiq-ai-development-environment/eiq-toolkit-for-end-to-end-model-development-and-deployment:EIQ-TOOLKIT)
> **Focus:** This example is different from the deployment scenarios (ex-01, ex-03). It does not govern *one product* — it governs the **toolchain that produces all products**. The goal is to embed governance *into the eIQ workflow itself*, so that every model built by any team is governed by construction, not by audit after the fact.

---

## The Core Idea: Shift-Left Governance

In ex-01 and ex-03, governance is applied to a finished AI system across its lifecycle. That works, but it has a structural weakness: governance arrives **after** the engineering decisions are already made. By the time the Model Risk Officer reviews a quantized model, the team has already chosen the dataset, trained the model, and quantized it. Governance becomes a gate that says "no" late — expensive, adversarial, and easy to bypass under deadline pressure.

**Embedding governance into the eIQ workflow inverts this.** When the governance artifact is a natural output of the eIQ stage the engineer is already performing — when curating a dataset in the eIQ Portal *also* produces the data-source registry entry, when running the Neutron Converter *also* runs the quantization regression gate — governance stops being a separate burden. It becomes a property of the toolchain.

This is the single highest-leverage governance investment NXP-based teams can make: instrument the development environment once, and every project downstream inherits governance automatically.

```
Traditional governance:   [Build] → [Build] → [Build] → 🚧 [Governance audit] → ship
Shift-left governance:     [Build+govern] → [Build+govern] → [Build+govern] → ship
                            every eIQ stage emits its own governance evidence
```

---

## The eIQ Workflow Mapped to the Governance Lifecycle

The eIQ Toolkit supports a defined end-to-end workflow: **dataset curation → model selection → training → optimization (quantization/pruning) → validation → target profiling → deployment**. The newer suite adds **GenAI Flow** (LLM+RAG pipelines), **Time Series Studio** (auto-ML for MCUs), the **Agentic AI Framework** (A2A + MCP orchestration), and the cloud-based **eIQ AI Hub**.

Each eIQ stage maps to one or more governance lifecycle stages (L1–L8) and triggers specific controls from the 12-domain framework:

| eIQ Workflow Stage | Governance Lifecycle Stage | Primary Control Domains |
|--------------------|---------------------------|------------------------|
| 0. Environment & Access (eIQ AI Hub / Toolkit) | L1 + cross-cutting | D3 Security, D10 Regulatory |
| 1. Dataset Curation (eIQ Portal Curator) | L2 Data | D1 Data, D4 Privacy, D5 Fairness |
| 2. Model Selection (Model Zoo / BYOM) | L1 + L3 | D2 LLM/SLM, D3 Security, D6 Ethics |
| 3. Training (eIQ Portal Trainer) | L3 Training | D1 Data, D5 Fairness, D9 MLOps |
| 4. Optimization — Quantize & Prune (Neutron Converter) | L4 Compression | D9 MLOps, D12 Functional Safety |
| 5. Validation (eIQ Portal Validate) | L5 Validation | D5 Fairness, D7 Explainability, D12 Safety |
| 6. Target Profiling (on-device) | L5 + L6 | D9 MLOps, D11 Environmental |
| 7. Deployment / Export (inference engine) | L6 Deployment | D9 MLOps, D3 Security, D8 Human Oversight |
| 8. Pipeline Assembly (GenAI Flow / Agentic A2A+MCP) | L1 + L3 + L6 | D2 LLM/SLM, D8 Human Oversight, D6 Ethics |

The rest of this document walks each stage. For every stage there are four parts: **Objective & Value Creation**, **Why Governance Matters Here**, **How to Blend Governance into the Workflow**, and **Implementation Steps**.

---

## Stage 0 — Environment & Access (eIQ AI Hub / Modular Toolkit)

### Objective & Value Creation
The eIQ AI Hub provides cloud access to the full tool suite (Toolkit, Time Series Studio, GenAI Flow, Agentic Framework), or teams can download the modular toolkit for on-premise use. The value is **a single, consistent, reproducible AI development environment** across all teams — no more "it worked on my machine" model builds, and a shared starting point for every project.

### Why Governance Matters Here
This is where governance either becomes effortless or impossible. If every team installs eIQ differently, with different versions, on uncontrolled machines, then no governance control downstream can be trusted — you cannot reproduce a build, verify a toolchain version, or guarantee an evidence trail. The environment is the foundation that makes every later control trustworthy. It is also the first security surface: the eIQ toolchain, its dependencies, and the AI Hub credentials are all attack targets (D3 SR-03 AI framework CVE tracking).

### How to Blend Governance into the Workflow
1. **Standardize the environment as code.** Pin the exact eIQ Toolkit version, Neutron Converter version, and inference-engine versions in a version-controlled manifest. Every project starts from this baseline.
2. **Make the environment emit its own identity.** Every build artifact should carry the toolchain fingerprint (eIQ version, converter version, host OS) automatically.
3. **Govern AI Hub access** with role-based access aligned to the governance roles (AIPO, MRO, DSL, etc.) so that who-can-do-what is enforced at the platform, not by policy memo.

### Implementation Steps
1. Create a `governed-eiq-baseline/` repo containing:
   - `eiq-toolchain.lock` — pinned versions of eIQ Toolkit, Neutron Converter, TFLite/ONNX/Glow/DeepViewRT runtimes
   - `Dockerfile` or `eiq-env.yaml` — a reproducible eIQ environment definition
   - `governance-roles.yaml` — mapping of AI Hub users to governance roles (RACI)
2. Subscribe the eIQ toolchain components to CVE feeds (D3 SR-03). Define a patch SLA: critical CVEs within 30 days.
3. Configure the eIQ AI Hub workspace so that project creation automatically scaffolds a `governance/` directory and an empty `governance-manifest.yaml` (see Automation Playbook).
4. Add a toolchain-fingerprint step that stamps every model artifact with `eiq_version`, `converter_version`, and `build_host_hash`.

---

## Stage 1 — Dataset Curation (eIQ Portal Dataset Curator)

### Objective & Value Creation
The eIQ Portal Dataset Curator lets developers import, annotate, augment, and organize training data through a GUI. The value is **fast, structured dataset preparation** — turning raw images/signals into labeled, training-ready datasets without writing data-pipeline code. This is the single most determinative stage for model quality: the curated dataset *is* what the model learns.

### Why Governance Matters Here
Everything the model knows comes from this dataset. Three governance risks live here, and all are invisible if not deliberately surfaced:
- **Provenance & licensing** (D1 DG-01/02): Where did each image come from? Is it licensed for commercial AI training?
- **Privacy** (D4 DP-01/03): Do the images contain faces, license plates, or other personal/biometric data? If so, a DPIA may be mandatory before curation even begins.
- **Representativeness & bias** (D5 FB-02/03): Does the dataset cover all the conditions, demographics, and hardware variants the model will face in production? Curating only daytime images for a 24/7 device bakes in a failure mode.

The eIQ Curator makes labeling fast — but speed without provenance discipline produces a fast path to an ungovernable model.

### How to Blend Governance into the Workflow
1. **Attach a datasheet to every imported dataset** at import time. The curator already records dataset structure; extend the workflow so a "Dataset Datasheet" (source, license, collection method, date range, known gaps) is a required field before the dataset can be used for training.
2. **Run a PII/biometric scan on import.** Before images enter the curated set, scan for faces and personal identifiers. Flag datasets that require a DPIA and block training until the DPIA is signed (D4 DP-01).
3. **Generate a representativeness report** from the curator's class/label distribution. The curator already knows the class balance — surface it as a fairness artifact (D5 FB-02).

### Implementation Steps
1. Wrap the dataset import with a pre-ingestion hook (`scripts/dataset_intake_gate.py`) that:
   - Requires a completed `datasheet.yaml` (template below) before import proceeds
   - Runs a face/PII detector over a sample and sets `requires_dpia: true/false`
   - Computes class distribution and writes `representativeness-report.json`
2. Store the datasheet and reports in the project's `governance/data/` directory, referenced from the governance manifest (DG-02, DG-04).
3. Configure the training stage (Stage 3) to refuse to start if `requires_dpia: true` and no signed DPIA is present — making the privacy control non-bypassable.

### Template — Dataset Datasheet (filled-in example)
```yaml
# governance/data/datasheet-conveyor-defects-v1.yaml
dataset_id: "conveyor-defects-v1"
title: "Conveyor Belt Surface Defect Images"
imported_via: "eIQ Portal Dataset Curator v25.1"
source: "Internal — Plant A line cameras, with vendor sample set"
license: "Internal use; vendor portion under CC-BY-4.0"
collection_method: "Line camera captures + augmentation (rotation, brightness)"
date_range: "2025-09 to 2026-02"
size: 18450
classes: ["no_defect", "tear", "fray", "foreign_object"]
class_distribution: { no_defect: 14200, tear: 2100, fray: 1600, foreign_object: 550 }
known_gaps: "foreign_object underrepresented (3%); no night-shift low-light samples"
contains_pii: false
requires_dpia: false
representativeness_mitigation: "Augment foreign_object class; collect low-light set before deployment"
document_owner: "Data Engineering Lead"
approved: true
```

---

## Stage 2 — Model Selection (eIQ Model Zoo / BYOM)

### Objective & Value Creation
Teams either pick a pre-optimized model from the **eIQ Model Zoo** or **Bring Your Own Model (BYOM)** — importing a TensorFlow, PyTorch, or ONNX model trained elsewhere. The value is **time-to-solution**: a Model Zoo model can be deployed in hours; BYOM lets teams reuse existing IP. This is also a moment of consequential choice that shapes the entire downstream risk profile.

### Why Governance Matters Here
The model is a supply-chain artifact (D3 SR-04/05) and a licensing artifact (D2 LM-01). A BYOM model imported from an external source could carry:
- **License risk** — a model under a research-only license deployed commercially
- **Provenance risk** — weights that are not what they claim to be (no checksum verification)
- **Security risk** — a model with no SBOM, no CVE history for its training framework
- **Ethical/scope risk** — a general-purpose model being used for a purpose it was never evaluated for (D6 EU-02)

The Model Zoo reduces these risks (NXP-curated), but BYOM reintroduces all of them. Governance must distinguish the two paths.

### How to Blend Governance into the Workflow
1. **Make model registration mandatory at selection.** Whether Model Zoo or BYOM, the model gets a registry entry (D9 ML-01) the moment it enters the project.
2. **Gate BYOM imports with a provenance + license check.** A BYOM model cannot proceed to training/fine-tuning until its source, checksum, and license are recorded and the license is confirmed compatible with the intended use.
3. **Generate the initial Model SBOM** (D3 SR-04) at selection: base model, source, checksum, framework versions.

### Implementation Steps
1. Add a `scripts/model_intake_gate.py` hook that runs on model import:
   - For Model Zoo: record the model ID and NXP-published version (provenance is implicit)
   - For BYOM: require `source_url`, verify `sha256` against a recorded value, require `license` field, and check it against an allowed-license list
   - Writes the model registry entry and the initial SBOM
2. Block progression to Stage 3 (Training) if license is `unknown` or `incompatible`.
3. Cross-reference the model's intended use against the EU AI Act classification (D6 EU-02) and prohibited-use registry (D6 EU-10) recorded at project conception.

### Template — Model Registry + SBOM Entry (filled-in)
```yaml
# governance/models/model-registry.yaml
- model_id: "defect-mobilenetv2-byom"
  selection_path: "BYOM"
  base_model: "MobileNetV2 (custom-trained internally)"
  source_url: "internal-mlflow://models/defect-mnv2/v3"
  sha256: "9f2c4a1e7b..."
  sha256_verified: true
  license: "Proprietary (internal)"
  license_compatible: true
  intended_use: "Conveyor surface defect classification — advisory"
  eu_ai_act_class: "minimal_risk"
  framework: "tensorflow==2.15"
  sbom_file: "governance/models/sbom-defect-mnv2.json"
  registered_by: "MRO"
  registered_date: "2026-06-15"
```

---

## Stage 3 — Training (eIQ Portal Trainer)

### Objective & Value Creation
The eIQ Portal trains (or fine-tunes) the selected model on the curated dataset, directly in the GUI. The value is **accessible model development** — engineers without deep ML-framework expertise can produce working models. The output is a trained FP32 model ready for optimization.

### Why Governance Matters Here
A trained model that cannot be reproduced cannot be governed. If, six months later, a deployed model behaves unexpectedly and you cannot reconstruct exactly how it was trained (dataset version, hyperparameters, seed, toolchain), you cannot investigate, cannot validate the fix, and cannot satisfy an auditor (D9 ML-03). Training is also where data-poisoning defenses (D1 DG-07) and the first fairness signal (D5 FB-04) live.

### How to Blend Governance into the Workflow
1. **Auto-capture the training run record.** The eIQ Portal already knows the dataset, model, and hyperparameters — emit them as a signed training run log automatically (D9 ML-03).
2. **Bind the dataset version.** The training record must reference the exact curated dataset version (and its datasheet) from Stage 1, creating end-to-end lineage (D1 DG-04).
3. **Draft the model card** at training completion (D9 ML-04, D7 XT-06) — pre-populated from the datasheet, model registry, and training metrics.

### Implementation Steps
1. Add a post-training hook (`scripts/capture_training_run.py`) that reads the eIQ Portal project state and writes `governance/training-runs/run-NNN.yaml` containing dataset hash, model ID, hyperparameters, seed, toolchain fingerprint, and output artifact hash.
2. Auto-generate a model-card draft from the registry + datasheet + training metrics, saved to `governance/model-cards/`.
3. Enforce the Stage 1 privacy gate here: training refuses to start if the dataset's `requires_dpia` is true and no signed DPIA exists.

---

## Stage 4 — Optimization: Quantization & Pruning (Neutron Converter)

### Objective & Value Creation
The eIQ Toolkit quantizes and prunes the trained model, and the **Neutron Converter Tool** converts the quantized TensorFlow Lite model so it can run on the eIQ Neutron NPU. This is the stage that makes edge AI *possible* — an FP32 model that needs a GPU becomes an INT8 model that runs in milliseconds on an i.MX or MCX device. Enormous value: this is the difference between "AI in the cloud" and "AI on the device."

### Why Governance Matters Here
**This is the highest-risk transformation in the entire eIQ workflow, and the one most often ungoverned.** Quantization changes the model's numerical behavior. Accuracy can drop, and — critically — it can drop *unevenly*: more for rare classes, more for safety-critical inputs, more for underrepresented groups. A model that passed every check at FP32 can fail silently after INT8 conversion. The governance framework treats this as a mandatory gate (D9 ML-05/06, D12 FS-05 for safety functions, D5 FB-06 for fairness). See ex-01 Stage L4 for a worked example where exactly this caught an emergency-stop misclassification.

### How to Blend Governance into the Workflow
1. **Make quantization regression a mandatory step in the converter workflow**, not an optional QA task. The model does not advance to validation until the regression report exists and passes.
2. **Run three regressions automatically** after every Neutron Converter run: general accuracy (D9 ML-05), safety-critical vectors (D9 ML-06 / D12 FS-05), and fairness/subgroup (D5 FB-06).
3. **Capture the quantization configuration** as part of the model definition (D9 ML-07) — calibration set, scheme, excluded layers.

### Implementation Steps
1. Wrap the Neutron Converter invocation in `scripts/quantization_regression_test.py` (see ex-01 Automation Playbook) that:
   - Runs the FP32 baseline and the INT8/INT4 quantized model on the held-out test set
   - Asserts accuracy delta within threshold; asserts zero discrepancies on the safety-critical vector set
   - Runs subgroup accuracy comparison for fairness drift
   - Writes `governance/reports/quant-regression-<model>.md`
2. Configure the converter step to fail (non-zero exit) if any assertion fails — blocking the artifact from progressing.
3. Record the quantization config in the model card and registry.

### Template — Converter Governance Gate Output (filled-in)
```
$ governed-convert --model defect-mnv2.tflite --target neutron --calib calib-500.npz

[eIQ Neutron Converter] Quantizing FP32 → INT8 ............ done
[Governance Gate ML-05] General accuracy regression:
    FP32: 96.2%  INT8: 95.1%  delta -1.1%  (threshold -2.0%) ...... PASS
[Governance Gate ML-06] Safety-critical vectors (foreign_object):
    FP32: 22/22  INT8: 22/22  discrepancies 0 .................... PASS
[Governance Gate FB-06] Subgroup fairness (lighting conditions):
    daylight -0.9%, low-light -1.4%, max disparity 0.5% .......... PASS
[Governance Gate ML-07] Quantization config recorded ............. OK
Artifact APPROVED for validation → governance/reports/quant-regression-defect-mnv2.md
```

---

## Stage 5 — Validation (eIQ Portal Validate)

### Objective & Value Creation
The eIQ Portal validates the (quantized) model — confusion matrices, per-class accuracy, and comparison between the trained and the deployed-target model. The value is **confidence before deployment**: you see how the model will actually behave on the target, with the real quantized weights, before it reaches a device.

### Why Governance Matters Here
Validation is where the framework's risk controls converge: fairness testing (D5 FB-04), explainability validation (D7 XT-05), adversarial robustness (D3 SR-06), and — for any safety function — the functional safety review (D12 FS-07). The eIQ confusion matrix is a governance artifact in disguise: per-class and per-subgroup error rates are exactly what a fairness auditor needs.

### How to Blend Governance into the Workflow
1. **Promote the eIQ confusion matrix into the fairness evidence.** Export it disaggregated by subgroup, not just overall, and store it as the D5 FB-04 artifact.
2. **Add adversarial and prompt-injection probes to the validation suite** (D3 SR-06, and for GenAI/LLM models D2 LM-06).
3. **Trigger the functional safety review** for any model classified SIL ≥ 1 (D12 FS-07) — validation cannot be marked complete without FSE sign-off for safety functions.

### Implementation Steps
1. Configure the validation stage to export per-subgroup confusion matrices to `governance/reports/validation-<model>.md`.
2. Add `scripts/adversarial_probe.py` (image: FGSM/PGD; LLM: garak prompt-injection suite) to the validation pipeline; record results in the prompt-injection / adversarial registry.
3. For safety functions, add a manual FSE sign-off gate in the governance manifest that must flip to `complete` before deployment is allowed.

---

## Stage 6 — Target Profiling (on-device)

### Objective & Value Creation
The eIQ Toolkit profiles the model on actual target hardware — measuring latency, memory, and (with NPU targets) NPU utilization. The value is **proof the model meets real-time and resource constraints** on the i.MX / MCX / Neutron / Ara NPU it will ship on, not just in simulation.

### Why Governance Matters Here
Two governance concerns surface here. First, **non-determinism and hardware variance** (D12 FS-06): does the model produce consistent outputs across hardware units and runs? Second, **energy and power budgets** (D11 ES-01/03): does the model fit the device's power envelope, which matters at fleet scale and for battery/thermal-constrained deployments? Profiling produces the measurements both controls need.

### How to Blend Governance into the Workflow
1. **Capture profiling output as governance evidence.** Latency, memory, and NPU power figures feed directly into the environmental power-budget control (D11 ES-03) and the MLOps HiL validation control (D9 ML-09).
2. **Run a determinism check** as part of profiling for safety functions (D12 FS-06): execute the same safety-critical inputs N times on hardware and record output variance.

### Implementation Steps
1. Extend the profiling run to emit `governance/reports/profiling-<model>.json` with latency percentiles, peak memory, and (NPU) average inference power.
2. Assert the profiled inference power against the power budget defined at L1 (D11 ES-01); flag overruns for hardware-team review.
3. For safety functions, run `scripts/determinism_check.py` (10× same input) and record max output variance.

---

## Stage 7 — Deployment / Export (Inference Engine)

### Objective & Value Creation
The eIQ Toolkit exports the validated, profiled model to the chosen inference engine (TensorFlow Lite, ONNX Runtime, Glow, or DeepViewRT) and packages it for the target. The value is **a deployable artifact** — the bridge from development to a model running on real devices in the field.

### Why Governance Matters Here
This is the point of no return — the model reaches devices. The OTA authorization chain (D9 ML-10), staged rollout (D9 ML-11), artifact signing (D3 SR-09), and per-device version tracking (D9 ML-12) all gate here. For autonomous systems, the kill-switch test (D8 HO-05) must pass before deployment. An ungoverned export is how an untested model reaches 500 devices at once.

### How to Blend Governance into the Workflow
1. **Make the export refuse to produce a deployable artifact unless the governance manifest is green** for all critical L1–L5 controls. This is the single most important integration point — the deployment gate.
2. **Sign the exported artifact automatically** (D3 SR-09) using the device-fleet signing key.
3. **Require a completed deployment authorization** (D9 ML-10) — the export emits an unsigned artifact for testing, but only a fully-authorized, manifest-green project produces a fleet-signed, deployable package.

### Implementation Steps
1. Integrate `scripts/check_governance_gate.py` (see ex-01) into the export step: it reads `governance-manifest.yaml` and blocks export if any critical L1–L5 control is not `complete`.
2. On a passing gate, sign the artifact and record the deployment authorization (4-party for safety-critical, per ex-03).
3. Register each deployed artifact's version per device in the fleet management system (D9 ML-12).

---

## Stage 8 — Pipeline Assembly (GenAI Flow / Agentic Framework, A2A + MCP)

### Objective & Value Creation
For generative and agentic use cases, the eIQ GenAI Flow assembles an LLM+RAG pipeline, and the eIQ Agentic AI Framework orchestrates multi-agent workflows using open standards — **A2A (Agent-to-Agent)** and **MCP (Model Context Protocol)**. The value is **assembling a multi-step autonomous AI pipeline on-device** from modular components, without building orchestration from scratch.

### Why Governance Matters Here
This is the frontier of edge AI risk. A multi-agent pipeline introduces governance concerns that single models do not have:
- **RAG corpus governance** (D1 DG-09/10/11) — see ex-01 for the worked example
- **Agent autonomy and oversight** (D8 HO-01/02) — see ex-03 for the multi-agent worked example
- **MCP tool-use governance** (D2 LM-14, excessive agency) — each tool/data source an agent can call via MCP is a permission that must be governed with least-privilege
- **A2A trust** — when agents call other agents, each must prove identity and authorization; an ungoverned A2A graph is an uncontrolled chain of autonomous actions

### How to Blend Governance into the Workflow
1. **Govern the MCP tool registry.** Every MCP tool/data source an agent can invoke is an entry in a least-privilege permission registry (D2 LM-14). An agent can only call tools explicitly granted for its defined use case.
2. **Govern the A2A graph.** Document and approve which agents may call which agents, with what authority. Treat the A2A topology as a governance artifact requiring sign-off (D8 HO-02).
3. **Apply RAG corpus governance** (ex-01 pattern) and **autonomy classification** (ex-03 pattern) to every GenAI/agentic pipeline.

### Implementation Steps
1. Create `governance/agentic/mcp-tool-registry.yaml` listing every MCP tool, the agents authorized to call it, and the granted scope (read-only, action, etc.).
2. Create `governance/agentic/a2a-graph.yaml` documenting the approved agent-to-agent call graph with per-edge authority and the human-oversight escalation rules (D8).
3. Run the autonomy classification (D8 HO-01) and prohibited-decisions registry (D12 FS-12) for the assembled pipeline before deployment, exactly as in ex-03.

### Template — MCP Tool Registry (filled-in)
```yaml
# governance/agentic/mcp-tool-registry.yaml
pipeline: "factory-maintenance-agent-v1"
mcp_tools:
  - tool: "rag.maintenance_corpus.query"
    agents_authorized: ["DiagnosticAgent"]
    scope: "read_only"
    governed_by: "D1 DG-09 corpus curation"
  - tool: "cmms.workorder.create"
    agents_authorized: ["DiagnosticAgent"]
    scope: "action_with_human_confirm"
    governed_by: "D8 HO-02 mandatory intervention point"
  - tool: "plc.actuator.write"
    agents_authorized: []          # NO agent may directly actuate machinery
    scope: "prohibited"
    governed_by: "D12 FS-12 prohibited autonomous decision"
a2a_escalation: "Any tool with scope=action_with_human_confirm pauses for operator approval"
```

---

## Putting It Together: The Governed eIQ Workflow Architecture

The integration pattern is a thin **governance wrapper** around each eIQ stage. The engineer uses the eIQ Portal / Toolkit exactly as before; the wrapper hooks fire automatically, producing evidence and enforcing gates.

```
  Engineer action (eIQ Portal/Toolkit)        Governance wrapper (automatic)
  ──────────────────────────────────────     ──────────────────────────────────
  Import dataset            ───────────▶      dataset_intake_gate.py → datasheet, PII scan
  Select model (Zoo/BYOM)   ───────────▶      model_intake_gate.py → registry, SBOM, license
  Train                     ───────────▶      capture_training_run.py → run log, model card
  Quantize (Neutron Conv.)  ───────────▶      quantization_regression_test.py → 3 regressions [GATE]
  Validate                  ───────────▶      adversarial_probe.py + subgroup matrices [GATE]
  Profile on target         ───────────▶      profiling → power budget + determinism check
  Export to inference engine───────────▶      check_governance_gate.py → sign + authorize [HARD GATE]
  Assemble GenAI/Agentic    ───────────▶      mcp-tool-registry + a2a-graph governance [GATE]
                                                        │
                                                        ▼
                                          governance-manifest.yaml (live ledger)
                                                        │
                                                        ▼
                                          CI: governance-gate.yml blocks deploy tag
```

Every hook writes to a single `governance-manifest.yaml` — the machine-readable governance ledger for the project (template in ex-01 Automation Playbook). The CI gate reads that manifest and refuses to cut a deployment tag unless the required controls are `complete`.

---

## Implementation Roadmap (Phased)

Embedding governance into the eIQ workflow is itself a project. Sequence it so each phase delivers standalone value:

| Phase | Scope | Deliverable | Value Unlocked |
|-------|-------|-------------|----------------|
| **Phase 1 — Foundation** | Stage 0 + manifest | Pinned eIQ baseline env; auto-scaffolded `governance/` dir + empty manifest; toolchain fingerprinting | Reproducible builds; every project starts governed |
| **Phase 2 — The critical gate** | Stage 4 + Stage 7 | Quantization regression gate + deployment gate (`check_governance_gate.py`) | The two highest-risk transitions (quantize, deploy) are now blocked unless governed |
| **Phase 3 — Evidence automation** | Stages 1, 2, 3 | dataset/model/training intake hooks producing datasheets, SBOMs, run logs automatically | Governance evidence produced as a byproduct of normal work |
| **Phase 4 — Validation depth** | Stages 5, 6 | adversarial probes, subgroup matrices, determinism + power checks | Fairness, robustness, safety, and environmental controls operationalized |
| **Phase 5 — Agentic frontier** | Stage 8 | MCP tool registry + A2A graph governance | GenAI Flow and Agentic pipelines governed by construction |

> **Start with Phase 2 if you can only do one thing.** The quantization gate and the deployment gate are where ungoverned eIQ workflows do the most damage, and they are the cheapest to instrument because they wrap a single tool invocation each.

---

## Tooling Summary

| Governance Hook | Wraps eIQ Component | Produces | Framework Controls |
|-----------------|--------------------|---------|--------------------|
| `dataset_intake_gate.py` | Portal Dataset Curator | datasheet, PII scan, representativeness report | DG-02, DP-01, FB-02 |
| `model_intake_gate.py` | Model Zoo / BYOM import | model registry entry, SBOM, license check | ML-01, SR-04, LM-01 |
| `capture_training_run.py` | Portal Trainer | training run log, model card draft | ML-03, ML-04 |
| `quantization_regression_test.py` | Neutron Converter | 3 regression reports (accuracy/safety/fairness) | ML-05, ML-06, FB-06, FS-05 |
| `adversarial_probe.py` | Portal Validate | adversarial/injection registry, subgroup matrices | SR-06, FB-04, LM-06 |
| `profiling` + `determinism_check.py` | Target Profiler | power-budget evidence, determinism variance | ES-03, FS-06, ML-09 |
| `check_governance_gate.py` | Export / Deploy | deployment gate decision, artifact signing | ML-10, ML-11, SR-09 |
| `mcp-tool-registry.yaml` + `a2a-graph.yaml` | GenAI Flow / Agentic | least-privilege tool & agent-call governance | LM-14, HO-02, FS-12 |

---

## Why This Matters Most

ex-01 and ex-03 show governance applied to *products*. This example shows governance applied to the *factory that builds the products*. The leverage is multiplicative: instrument the eIQ workflow once, and every future model — FactoryAssist, BuildingSafe, and the dozens of projects after them — is born governed. Governance stops being a department that says "no" at the end and becomes a property of the toolchain that quietly produces the evidence and enforces the gates while engineers do their normal work in the eIQ Portal.

That is the difference between a governance framework that exists on paper and one that actually governs.

---

## Sources

- [NXP eIQ AI Development Environment](https://www.nxp.com/design/design-center/software/eiq-ai-development-environment:EIQ)
- [eIQ Toolkit for End-to-End Model Development and Deployment](https://www.nxp.com/design/design-center/software/eiq-ai-development-environment/eiq-toolkit-for-end-to-end-model-development-and-deployment:EIQ-TOOLKIT)
- [NXP's eIQ ML Software Development Environment (blog)](https://www.nxp.com/company/about-nxp/smarter-world-blog/BL-NXP-EIQ-ML-SOFTWARE-DEVELOPMENT)
- [NXP Advances Edge AI Leadership with New eIQ Agentic AI Framework](https://www.nxp.com/company/about-nxp/newsroom/NW-NXP-ADVANCES-EDGE-AI-LEADERSHIP-EIQ)
- [eIQ GenAI Flow: Conversational AI Software Pipeline on Edge Devices](https://www.nxp.com/design/design-center/software/embedded-software/eiq-genai-flow-conversational-ai-software-pipeline-on-edge-devices:GEN-AI-FLOW)

---

*Example 2 | Governing the eIQ AI Development Environment | Version 1.0 — 2026-06-01*
