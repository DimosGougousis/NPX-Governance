# D5 — Fairness & Bias

> **Domain purpose:** Govern the identification, measurement, and mitigation of bias in NXP edge AI systems — including bias in training data, sensor hardware variance across device units, and discriminatory outcomes in production. Ensures that AI-assisted decisions do not unfairly disadvantage any group based on protected characteristics.
>
> **Primary standards:** IEEE 7003 (Algorithmic bias considerations), ISO/IEC TR 24027 (Bias in AI systems), EU AI Act Recitals 44-47 and Art. 10 (training data bias)
>
> **RACI owner:** Model Risk Officer (MRO) — primary

---

## NXP-Specific Context

Edge AI introduces bias sources not present in cloud AI:

1. **Sensor hardware bias.** Different NXP hardware units (camera sensors, microphones, IMUs) have manufacturing variances that create input distribution differences across devices. A model trained predominantly on one sensor lot may underperform on another — a form of hardware-induced distributional bias that is unique to edge deployments.

2. **Environmental bias.** Edge devices operate in heterogeneous physical environments (different lighting, temperatures, acoustic conditions). Models trained in controlled lab conditions may systematically underperform for users in environments not represented in training — effectively an environmental fairness issue.

3. **Limited feedback loop.** Cloud AI systems can observe outcomes and measure disparate impact across populations in near-real-time. Offline or low-connectivity edge devices make outcome monitoring harder — requiring governance that front-loads bias testing before deployment rather than relying on production monitoring alone.

---

## Control Checklist

### Sub-domain 5.1: Bias Risk Assessment (L1)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| FB-01 | **Protected characteristics mapping:** Identify all legally protected characteristics relevant to the AI system's use case (race, ethnicity, gender, age, disability, religion, nationality — and any jurisdictionally specific characteristics). Document how each could be affected by the AI system's outputs or decisions. If the system does not process personal data or affect individuals directly, document this explicitly as "no disparate impact risk." | 🔴 | L1 | Protected characteristics mapping document |

### Sub-domain 5.2: Training Data Fairness (L2)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| FB-02 | **Training data representativeness analysis:** Analyze the training dataset for demographic and environmental representation. Document: (a) which groups / environments / hardware variants are represented; (b) which are underrepresented or absent; (c) the mitigation strategy for gaps (augmentation, targeted collection, operational domain restriction). | 🔴 | L2 | Representativeness analysis report; gap mitigation plan |
| FB-03 | **Sensor hardware diversity in training data:** For models trained on sensor inputs, document the hardware diversity of the training dataset: which NXP hardware SKUs, manufacturing lots, sensor configurations, and calibration states are represented. If training data is dominated by a single hardware variant, assess the risk of performance degradation on other variants. | 🟡 | L2 | Hardware diversity analysis; performance variance risk assessment |

### Sub-domain 5.3: Pre-Deployment Bias Testing (L3)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| FB-04 | **Pre-deployment bias testing:** Before any model that affects individuals proceeds to validation, execute a bias testing suite. Test model performance (accuracy, false positive rate, false negative rate) disaggregated by relevant protected characteristics and demographic groups. Define acceptance thresholds for maximum allowed performance disparities. Document methodology and results. | 🔴 | L3 | Bias testing methodology; disaggregated performance results; pass/fail against thresholds |
| FB-05 | **Proxy variable analysis:** Identify features in the model's input that may serve as proxies for protected characteristics (e.g., location as proxy for ethnicity, voice pitch as proxy for gender). Document identified proxies and the mitigation approach (feature removal, fairness constraints, post-processing). | 🟡 | L3 | Proxy variable analysis report; mitigations per identified proxy |

### Sub-domain 5.4: Quantization Fairness (L4)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| FB-06 | **Quantization fairness regression:** After INT8/INT4 quantization, re-run the disaggregated performance test from FB-04 on the quantized model. Quantization can introduce differential accuracy degradation across subgroups — a phenomenon known as quantization-induced bias amplification. Any new performance disparities introduced by quantization must be investigated and resolved. | 🔴 | L4 | Quantization fairness regression report; comparison of FP32 vs. quantized disaggregated metrics |

### Sub-domain 5.5: Validation (L5)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| FB-07 | **Environmental fairness testing:** Test model performance across the range of physical environments expected in deployment (lighting conditions, background noise levels, temperature ranges, vibration profiles). Identify conditions where performance degrades disproportionately for specific user groups. | 🟡 | L5 | Environmental testing matrix and results; failure condition documentation |
| FB-08 | **Hardware variant fairness testing:** If deployment covers multiple NXP hardware SKUs or sensor generations, test model performance across hardware variants. Performance disparities across hardware variants must be within defined bounds or restricted deployment must be documented. | 🟡 | L5 | Cross-hardware performance comparison report |

### Sub-domain 5.6: Deployment (L6)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| FB-09 | **Bias documentation in model card:** The model card (required by D9/ML-04) must include a dedicated fairness section documenting: protected characteristics assessed, testing methodology, disaggregated performance metrics, known bias risks, and mitigations applied. This section must be reviewed by the MRO before deployment. | 🔴 | L6 | Model card with completed fairness section; MRO review sign-off |

### Sub-domain 5.7: Operations & Monitoring (L7)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| FB-10 | **Production outcome monitoring (where feasible):** Where the device collects outcome data (actual results of AI-assisted decisions), monitor outcome rates disaggregated by demographic groups or environmental categories. Define alert thresholds for emerging disparities. For devices with limited connectivity, implement batch reporting of outcome metrics. | 🟡 | L7 | Outcome monitoring configuration; alert threshold definitions |
| FB-11 | **Bias incident response:** Define a procedure for responding to detected bias incidents (discriminatory outcomes identified in production monitoring, customer complaints indicating bias, regulatory inquiry). Include: investigation methodology, corrective action options (model rollback, retraining, operational restriction), and stakeholder notification. | 🟡 | L7 | Bias incident procedure; corrective action options documented |

---

## Standards Cross-Reference

| Control(s) | IEEE 7003 | ISO/IEC TR 24027 | EU AI Act | NIST AI RMF |
|------------|-----------|-----------------|-----------|-------------|
| FB-01 | §6.2 (stakeholder analysis) | §5.2 | Recital 44, Art. 10 | MAP 1.5 |
| FB-02, FB-03 | §7.1 (data requirements) | §6.2 (data bias) | Art. 10(2)(f) | MEASURE 2.5 |
| FB-04, FB-05 | §7.3 (testing) | §6.4 | Art. 10(3) | MEASURE 2.5 |
| FB-06 | §7.3 | §6.3 | Art. 10 | MEASURE 2.6 |
| FB-07, FB-08 | §7.4 | §6.5 | Art. 9 | MEASURE 2.5 |
| FB-09 | §8.1 (documentation) | §7.1 | Art. 13, Annex IV | GOVERN 6.1 |
| FB-10, FB-11 | §8.2 (monitoring) | §7.2 | Art. 72 | MANAGE 4.1 |

---

*Domain D5 | Version 1.0 — 2026-05-29*
