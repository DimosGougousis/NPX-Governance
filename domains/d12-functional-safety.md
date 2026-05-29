# D12 — Functional Safety

> **Domain purpose:** Govern the functional safety requirements for AI systems deployed on NXP edge devices that operate in safety-relevant contexts — where AI decisions or failures can result in injury, death, environmental damage, or critical system failure. Provides safety integrity level (SIL/ASIL) determination guidance, fail-safe design requirements, prohibited autonomous decision categories, and hazard analysis procedures applicable across all verticals.
>
> **Primary standards:** IEC 61508 (functional safety of E/E/PE safety-related systems), ISO 26262 (automotive — principles apply broadly), IEC 62061 (machinery safety), ISO/IEC TR 5469 (AI and functional safety, 2024)
>
> **RACI owner:** Functional Safety Engineer (FSE) — primary; Model Risk Officer (MRO) — secondary for model validation

---

## NXP-Specific Context

NXP targets safety-critical markets with its i.MX 95 and S32 processor families, which include automotive-grade (ASIL-B/D capable) hardware. However, **hardware safety certification does not imply software or AI safety**. The introduction of AI into a safety-related function changes the safety architecture in fundamental ways:

1. **AI is not deterministic.** Traditional functional safety relies on deterministic software behavior — the same input always produces the same output. Neural networks operating on quantized NPUs can produce slightly different outputs across hardware batches due to numerical rounding. Safety analysis must account for this non-determinism.

2. **AI failure modes are different.** A traditional software bug either crashes or produces a wrong output for a known class of inputs. AI failure modes include: overconfident incorrect predictions, silent degradation on out-of-distribution inputs, adversarial failures, and quantization-induced accuracy drops. These are harder to enumerate in a traditional FMEA.

3. **ISO/IEC TR 5469 (2024).** The first international technical report specifically addressing AI and functional safety recommends treating AI as a design element within a safety function — not as a standalone safety-capable entity. The safety case must show how the surrounding system architecture (fallbacks, monitors, human oversight) compensates for AI's probabilistic nature.

4. **Autonomy levels determine safety risk.** An AI system that only advises a human operator has a fundamentally different safety profile than one that directly actuates physical systems. The autonomy level classification (D8) feeds directly into the SIL determination.

---

## Control Checklist

### Sub-domain 12.1: Hazard Analysis & Risk Assessment (L1)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| FS-01 | **Hazard analysis (HARA/HAZOP):** Conduct a systematic hazard analysis (Hazard Analysis and Risk Assessment per ISO 26262 / IEC 61508 §7.4) for every AI-enabled function. Identify: (a) hazardous events that can result from AI system failure or misbehavior; (b) severity of potential harm (S0–S3); (c) probability of exposure (E0–E4); (d) controllability by the affected party (C0–C3). Document the analysis formally. | 🔴 | L1 | HARA/HAZOP report per AI function; severity/exposure/controllability ratings |
| FS-02 | **Safety Integrity Level (SIL/ASIL) determination:** Based on the hazard analysis, determine the required SIL (IEC 61508) or ASIL (ISO 26262) for each safety-related AI function. Where AI is not in a safety-related function (no identified hazards), document this explicitly as a "QM" (quality management only) determination. The SIL/ASIL determination must be reviewed and approved by the FSE. | 🔴 | L1 | SIL/ASIL determination document; FSE sign-off |

### Sub-domain 12.2: Safety Architecture & Data (L2/L3)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| FS-03 | **Safety-critical training data requirements:** For AI functions with SIL ≥ 2 or ASIL ≥ B, the training dataset must include representative coverage of all identified safety-critical scenarios, including rare but hazardous conditions. Document coverage gaps and the mitigation strategy for uncovered scenarios (e.g., synthetic data augmentation, operational design domain restriction). | 🔴 | L2 | Safety-critical scenario coverage analysis; gap mitigation plan |
| FS-04 | **Fail-safe behavior specification:** For every AI function, specify the fail-safe behavior: what the system does if the AI model fails, produces an out-of-distribution output, exceeds confidence thresholds, or is unavailable. Fail-safe behavior must be: (a) deterministic and non-AI (rule-based fallback or halt); (b) validated independently of the AI model; (c) able to activate within the fault-tolerant time interval defined in the hazard analysis. | 🔴 | L3 | Fail-safe behavior specification per function; independence evidence; activation time validation |

### Sub-domain 12.3: Compression & Quantization Safety (L4)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| FS-05 | **Safety property preservation through quantization (functional safety view):** For AI functions at SIL ≥ 1 / ASIL ≥ A, perform a dedicated safety-focused quantization validation. Test all identified safety-critical input scenarios against the quantized model. Any case where the quantized model produces a different safety-relevant output from the FP32 model must be investigated and resolved before deployment. (Complements ML-06 in D9.) | 🔴 | L4 | Safety quantization validation report; comparison of FP32 vs. INT8/INT4 on safety test vectors |
| FS-06 | **Non-determinism characterization:** Measure and document the degree of non-determinism in the deployed quantized model on the target NXP hardware. Run the same input set multiple times and characterize output variance. If variance affects safety-relevant outputs, implement determinism controls (fixed seeds, deterministic NPU execution mode) or treat variance as a failure mode in the safety analysis. | 🟡 | L4 | Non-determinism measurement report; characterization of variance on safety-critical outputs |

### Sub-domain 12.4: Validation & Safety Review (L5)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| FS-07 | **Functional safety review sign-off:** Before any AI system at SIL ≥ 1 / ASIL ≥ A proceeds to deployment, the FSE must conduct a formal functional safety review. The review assesses: hazard analysis completeness, fail-safe behavior implementation, quantization safety validation, fail-safe activation test results, and residual risk acceptability. The review must produce a written assessment with a pass/fail determination. | 🔴 | L5 | Formal functional safety review report; FSE sign-off; residual risk statement |
| FS-08 | **Fail-safe activation testing:** Test fail-safe activation in all identified failure modes: model failure (corrupt weights), hardware NPU fault, out-of-distribution input, and watchdog timeout. Measure actual activation latency against the fault-tolerant time interval. All tests must pass before deployment approval. | 🔴 | L5 | Fail-safe activation test results per failure mode; latency measurements vs. FTTI |
| FS-09 | **Independent safety validation:** For SIL ≥ 2 / ASIL ≥ C functions, the safety validation must be conducted by an assessor independent from the development team. This is a mandatory independence requirement under IEC 61508 §8.3 and ISO 26262 Part 8. | 🟡 | L5 | Independent assessor qualification records; independent validation report |

### Sub-domain 12.5: Deployment Safety (L6)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| FS-10 | **Operational design domain (ODD) enforcement:** For AI functions with defined operational design domains (specific environmental conditions, sensor ranges, or operational contexts within which the AI is validated), implement runtime monitoring that detects when the device is operating outside the ODD and activates the fail-safe or alerts the operator. | 🔴 | L6 | ODD definition document; ODD monitoring implementation; out-of-ODD response test results |

### Sub-domain 12.6: Operations & Monitoring (L7)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| FS-11 | **Safety performance monitoring:** Define and continuously monitor safety-relevant performance metrics in production: false negative rate for safety-critical detections, fail-safe activation frequency, ODD exceedance rate. Define thresholds that trigger investigation and those that mandate model rollback. | 🔴 | L7 | Safety monitoring dashboard; threshold definitions; escalation procedure |
| FS-12 | **Prohibited autonomous decisions registry:** Maintain an explicit, approved registry of decision categories that the AI system is prohibited from making autonomously — decisions that must always involve human oversight or have a deterministic rule-based check. Examples: disabling a physical safety interlock, issuing an evacuation command, administering medication dosage. Any change to this registry requires FSE and AIPO approval. | 🔴 | L7 | Prohibited decisions registry; review and approval history |

### Sub-domain 12.7: Retirement (L8)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| FS-13 | **Safety case archiving:** Upon retirement of any AI system at SIL ≥ 1 / ASIL ≥ A, archive the complete safety case (HARA, SIL determination, validation reports, functional safety review) for a period consistent with the product's legal and regulatory retention obligations. Minimum 10 years for most regulated sectors. | 🟡 | L8 | Archive confirmation; retention period documentation |

---

## Prohibited Autonomous Decision Categories (Template)

The following categories **must never** be made autonomously by an edge AI system without a deterministic safety check or human oversight gate. This list must be reviewed and approved for each deployment context:

| Category | Rationale |
|----------|-----------|
| Physical safety interlock disablement | Removing a safety barrier could directly cause injury |
| Emergency system activation (evacuation, shutdown) | False positive has severe consequences; false negative risks lives |
| Medical intervention dosing | Patient safety; irreversible harm from error |
| Autonomous vehicle speed > operational limit | Exceeds validated ODD |
| Unilateral network access modification | Could compromise fleet-wide connectivity |
| Personal data disclosure to third parties | GDPR/privacy violation; irreversible |

---

## Standards Cross-Reference

| Control(s) | IEC 61508 | ISO 26262 | ISO/IEC TR 5469 | IEC 62061 | NIST AI RMF |
|------------|-----------|-----------|----------------|-----------|-------------|
| FS-01, FS-02 | §7.4 (hazard analysis) | Part 3 §7 (HARA) | §6.3 (risk classification) | §7.4 | MAP 5.1 |
| FS-03, FS-04 | §7.9 (safety requirements) | Part 4 (product development) | §6.4 (safety design) | §8 | MANAGE 2.1 |
| FS-05, FS-06 | §7.11 (integration testing) | Part 6 (software level) | §6.4 | §8 | MEASURE 2.5 |
| FS-07, FS-08, FS-09 | §8.3 (independence) | Part 8 §8 | §6.5 | §8.4 | MEASURE 2.7 |
| FS-10 | §7.9 | Part 4 §7 | §6.4 | — | MANAGE 3.1 |
| FS-11, FS-12 | §7.7 (failure modes) | Part 7 §9 | §6.6 | §9 | MANAGE 4.1 |
| FS-13 | §5.4 (documentation) | Part 2 §6.4 | — | §5.4 | GOVERN 4.2 |

---

*Domain D12 | Version 1.0 — 2026-05-29*
