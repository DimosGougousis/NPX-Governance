# D9 — MLOps & Model Lifecycle Governance

> **Domain purpose:** Govern the end-to-end lifecycle of AI/ML models deployed on NXP edge devices — from initial model registration through training pipeline controls, compression and quantization, fleet deployment, production monitoring, and final retirement. Ensures that every model version on every device is traceable, authorized, and reversible.
>
> **Primary standards:** ISO/IEC 42001 §6 (Planning), §8 (Operation), §9 (Performance Evaluation); NXP eIQ GenAI Flow deployment guidelines
>
> **RACI owner:** Model Risk Officer (MRO) — primary; Device Security Lead (DSL) — secondary for OTA and device controls

---

## NXP-Specific Context

NXP's eIQ GenAI Flow pipeline introduces governance challenges absent from cloud-based AI:

1. **Quantization as a governance event.** Converting a full-precision model to INT8 or INT4 for the Neutron NPU changes numerical behavior. Accuracy regressions, classification boundary shifts, and safety-property degradation can occur silently. This is not a deployment detail — it is a **model transformation that requires validation**.

2. **Distributed fleet management.** Thousands of edge devices may each run slightly different model versions due to staged rollouts, failed OTA updates, or deliberate segmentation. The governance system must track each device's exact model version at any point in time.

3. **Constrained observability.** Unlike cloud models, edge models may operate offline or with limited telemetry bandwidth. Drift detection, incident response, and audit trail retrieval must account for intermittent connectivity.

4. **OTA update attack surface.** Every model update is a potential attack vector. The authorization chain for pushing a new model must be as rigorous as deploying firmware.

---

## Control Checklist

### Sub-domain 9.1: Model Registration & Versioning (L1)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| ML-01 | **Model version registry:** Maintain a central registry of all AI/ML models intended for NXP edge deployment. Each entry must record: model name, version, base architecture, quantization format (FP32/INT8/INT4), target hardware (i.MX 93 / 8M Plus / 95), license, deployment status, and responsible MRO. | 🔴 | L1 | Registry document or system export showing all active models |
| ML-02 | **Training data provenance link:** Each model version in the registry must link to the exact dataset version(s) used for training and fine-tuning. | 🔴 | L2 | Registry entries with dataset version IDs; reproducible training run logs |

### Sub-domain 9.2: Training Pipeline Governance (L3)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| ML-03 | **Reproducible training:** All model training runs must be logged with: framework version, hyperparameters, dataset hash, random seed, and hardware configuration. Runs must be reproducible from the log alone. | 🔴 | L3 | Training run logs with complete configuration capture |
| ML-04 | **Model card draft at training completion:** A model card must be drafted upon completing each training run, covering: intended use, out-of-scope uses, training data summary, performance metrics across subgroups, and known limitations. | 🟡 | L3 | Draft model card document |

### Sub-domain 9.3: Compression & Quantization Governance (L4) ← NXP-Critical

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| ML-05 | **INT8/INT4 quantization accuracy regression testing:** Before promoting any quantized model (INT8 or INT4) to validation, run a full accuracy regression suite comparing the quantized model against the FP32 baseline on a held-out representative test set. Define acceptance thresholds per model class (e.g., max 1% accuracy drop for classification, max 2% F1 drop for detection). Fail the promotion if thresholds are exceeded. | 🔴 | L4 | Regression test report with FP32 vs. quantized accuracy deltas; pass/fail against defined thresholds |
| ML-06 | **Safety property preservation through quantization:** For any model making safety-relevant outputs (fail/no-fail decisions, hazard alerts, safety interlocks), formally verify that quantization has not altered decision boundaries for known safety-critical inputs. Test with the same safety-critical input set used in functional safety review (D12). | 🔴 | L4 | Safety property test report showing equivalence of FP32 and quantized outputs on safety-critical test vectors |
| ML-07 | **Quantization configuration documentation:** Document the quantization configuration used (calibration dataset, quantization scheme, layer exclusions, NPU-specific optimizations) as part of the model card. This configuration is a material part of the model definition. | 🟡 | L4 | Quantization config file committed to version control; model card updated |
| ML-08 | **ONNX/TFLite export verification:** After conversion to ONNX, TFLite, or eIQ-native format, run inference output equivalence tests on a representative sample to confirm the export pipeline has not introduced silent numerical errors. | 🔴 | L5 | Export equivalence test report; sample input/output comparison |

### Sub-domain 9.4: Validation & Testing (L5)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| ML-09 | **Hardware-in-the-loop (HiL) validation:** All models must be validated on actual target NXP hardware (not only simulation) before deployment. HiL tests must cover edge cases, out-of-distribution inputs, and worst-case latency scenarios on the Neutron NPU. | 🔴 | L5 | HiL test execution report on target hardware; latency and accuracy results |

### Sub-domain 9.5: Deployment & Fleet Management (L6)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| ML-10 | **OTA update authorization chain:** Every model update pushed over-the-air must pass through a documented multi-party authorization chain. At minimum: MRO approval (technical), AIPO approval (use case), DSL sign-off (security). Emergency updates may use an expedited path but must be retrospectively approved within 24 hours. | 🔴 | L6 | Authorization chain policy document; signed approval records for each deployment |
| ML-11 | **Staged rollout policy:** No model update may be deployed directly to the full fleet. Minimum stages: (1) canary — <1% of fleet, 48-hour monitoring; (2) early adopter — 5-10% of fleet, 72-hour monitoring; (3) general availability — remaining fleet. Each stage requires MRO sign-off before advancing. | 🔴 | L6 | Staged rollout policy; deployment logs showing stage progression; monitoring reports per stage |
| ML-12 | **Device-level model version tracking:** Every device in the fleet must report its current model name, version, and quantization format to the fleet management system. This state must be queryable at any time to determine the exact model running on any individual device. | 🔴 | L6 | Fleet management system export showing per-device model versions; query capability demonstrated |

### Sub-domain 9.6: Operation & Monitoring (L7)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| ML-13 | **Model drift detection:** Define thresholds for input distribution drift (data drift) and output distribution drift (concept drift) per model. Implement automated detection with defined alert procedures. Include provisions for devices with limited telemetry (batch reporting, delta compression). | 🔴 | L7 | Drift detection configuration with thresholds; alert procedure documentation; example drift alert |
| ML-14 | **Rollback capability:** For every model deployed to any device, maintain the immediately preceding model version in a retrievable state. Rollback must be executable within a defined time window (default: 4 hours from alert to rollback completion across 95% of affected devices). | 🔴 | L7 | Rollback procedure documentation; rollback time test results; version retention policy |
| ML-15 | **Model performance monitoring:** Continuously monitor model performance metrics (accuracy, latency, confidence distribution) in production. Define thresholds that trigger investigation and those that trigger mandatory rollback. | 🟡 | L7 | Performance monitoring dashboard configuration; threshold definitions; escalation procedure |

### Sub-domain 9.7: Model Retirement (L8)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| ML-16 | **End-of-life procedure:** When retiring a model version, execute a documented decommission process: (1) remove model artifacts from all devices; (2) archive training artifacts, model card, test reports to long-term storage; (3) update registry with retirement date and successor model; (4) notify all stakeholders. | 🔴 | L8 | Completed EOL checklist; archive confirmation; registry update |
| ML-17 | **Device data purge on decommission:** When a device is permanently decommissioned, securely erase all locally stored model artifacts, inference logs, and RAG knowledge base data. For regulated data (PHI, PII), provide a certificate of destruction. | 🟡 | L8 | Data purge procedure; destruction certificates where applicable |

---

## Standards Cross-Reference

| Control(s) | ISO/IEC 42001 | NIST AI RMF | EU AI Act | Other |
|------------|--------------|-------------|-----------|-------|
| ML-01, ML-02 | §8.3 (AI system life cycle) | MANAGE 1.1 | Art. 17 (quality management) | — |
| ML-03, ML-04 | §8.4 (AI system documentation) | MANAGE 2.2 | Art. 11, Annex IV | — |
| ML-05, ML-06 | §8.5 (operational controls) | MEASURE 2.5 | Art. 9 (risk mgmt), Art. 15 (accuracy) | IEC 61508 §7 |
| ML-07, ML-08 | §8.4 (documentation) | MEASURE 2.6 | Art. 11 | NXP eIQ Flow docs |
| ML-09 | §8.5 | MEASURE 2.7 | Art. 9 | — |
| ML-10, ML-11, ML-12 | §8.5 (deployment controls) | MANAGE 3.1 | Art. 72 (post-market monitoring) | ETSI EN 303 645 §5.3 |
| ML-13, ML-14, ML-15 | §9.1 (performance evaluation) | MANAGE 4.1 | Art. 72 | — |
| ML-16, ML-17 | §8.3 (lifecycle) | GOVERN 4.1 | Art. 18 (documentation retention) | GDPR Art. 17 |

---

*Domain D9 | Version 1.0 — 2026-05-29*
