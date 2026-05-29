# D4 — Privacy & Data Protection

> **Domain purpose:** Govern the handling of personal data and sensitive information processed by NXP edge AI systems, ensuring compliance with GDPR and the privacy-by-design principles of ISO 29101. Addresses automated decision-making rights, DPIA requirements, on-device PII minimization, and consent management at the edge.
>
> **Primary standards:** GDPR (Regulation (EU) 2016/679), ISO/IEC 29101 (Privacy architecture framework), ISO/IEC 27701 (Privacy information management)
>
> **RACI owner:** Privacy Officer / DPO — primary

---

## NXP-Specific Context

The privacy proposition of NXP on-device AI is powerful: sensitive data (faces, voices, health indicators, location, behavioral patterns) can be processed locally without being transmitted to the cloud. However, this proposition only holds if it is actively governed. Privacy-by-design for edge AI requires:

1. **Proving data doesn't leave.** The governance program must demonstrate — not merely assert — that personal data processed during inference does not exit the device. This requires telemetry policy controls and technical enforcement.

2. **GDPR Article 22 compliance for automated decisions.** If the AI system makes decisions about individuals that produce legal or similarly significant effects (access control, medical triage, creditworthiness), Article 22 requires either explicit consent, contractual necessity, or a legal obligation basis — plus the right to human review.

3. **DPIA triggers.** Large-scale processing of sensitive categories (biometric data, health data, children's data) using novel technology (AI on edge devices) typically triggers a Data Protection Impact Assessment under GDPR Article 35.

---

## Control Checklist

### Sub-domain 4.1: Privacy by Design at L1 (Conception)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| DP-01 | **DPIA trigger assessment:** At project conception, assess whether a Data Protection Impact Assessment (DPIA) is required under GDPR Article 35. A DPIA is required if the processing involves: biometric data for identification, health or location data at scale, behavioral profiling, or use of new technology (AI inference on personal data). Document the assessment outcome. If DPIA is required, complete it before data collection begins. | 🔴 | L1 | DPIA trigger assessment document; if triggered: completed DPIA |
| DP-02 | **Legal basis determination for automated processing:** For each processing activity involving personal data, determine and document the legal basis under GDPR Article 6. For automated decision-making affecting individuals, also assess Article 22 obligations (right to opt-out, right to explanation, right to human review). | 🔴 | L1 | Processing activity register entry; legal basis documentation; Article 22 assessment |

### Sub-domain 4.2: Data Collection & Processing Controls (L2)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| DP-03 | **Data minimization enforcement:** Implement technical controls ensuring that only the minimum necessary personal data is collected and retained on-device. Specifically: (a) no raw biometric data (face images, voice recordings) stored beyond the inference context; (b) no personally identifiable sensor data transmitted to the cloud unless explicitly required and consented; (c) inference results stored in aggregate or anonymized form where possible. | 🔴 | L2 | Data minimization architecture review; technical control implementation evidence |
| DP-04 | **On-device PII detection and scrubbing:** For AI systems that process unstructured text (voice transcription, OCR, operator input), implement on-device PII detection to identify and redact personal identifiers (names, addresses, ID numbers, health data) before any logging or telemetry. Document the detection methodology and false negative rate. | 🟡 | L2 | PII detection implementation; false negative rate assessment; redaction test results |

### Sub-domain 4.3: Training Data Privacy (L3)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| DP-05 | **Training data privacy compliance:** For any training dataset containing personal data: (a) confirm legal basis for processing (consent or legitimate interest); (b) confirm data subjects' rights are addressable (if consent basis, withdrawal mechanisms exist); (c) assess whether anonymization or pseudonymization is sufficient to eliminate personal data from the training set; (d) document retention period for training data containing personal data. | 🔴 | L3 | Training data privacy assessment; legal basis documentation; anonymization assessment |

### Sub-domain 4.4: Validation & Pre-Deployment (L5)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| DP-06 | **Memorization / data leakage testing:** Before deploying a model trained on personal or sensitive data, test whether the model has memorized specific training data points that could be extracted through inference (a form of privacy leakage). Use canary testing or membership inference attack simulation. Document findings and mitigations. | 🟡 | L5 | Memorization test results; mitigation documentation |

### Sub-domain 4.5: Deployment (L6)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| DP-07 | **Telemetry data governance:** Define precisely what operational telemetry is collected from the device and transmitted off-device. For every telemetry data point: classify it (personal / sensitive / operational), identify the legal basis for transmission, and ensure it cannot be combined with other data to re-identify individuals. Publish a telemetry data map. | 🔴 | L6 | Telemetry data map; legal basis per data point; re-identification risk assessment |

### Sub-domain 4.6: Operations (L7)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| DP-08 | **Data subject rights procedures:** Implement and test procedures for exercising data subject rights related to AI-processed personal data: right of access (Article 15), right to erasure (Article 17), right to object to automated processing (Article 22), and right to explanation. Document response time (default: 30 days). | 🔴 | L7 | Data subject rights procedure; test evidence of each right being exercisable; response time SLA |
| DP-09 | **Privacy incident response:** Define and test a procedure for privacy incidents related to edge AI processing (unauthorized data access, PII logged in error, personal data exfiltrated via OTA vulnerability). Include GDPR Article 33 breach notification timeline (72 hours to supervisory authority). | 🔴 | L7 | Privacy incident procedure; breach notification template; test records |

### Sub-domain 4.7: Retirement (L8)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| DP-10 | **Personal data purge on device retirement:** When a device is decommissioned, securely erase all personal data from on-device storage, including: inference caches, audit logs containing personal data, RAG query logs, and any biometric templates. Provide a certificate of erasure for regulated data categories. | 🔴 | L8 | Erasure procedure; completion records; certificates of erasure where required |

---

## Standards Cross-Reference

| Control(s) | GDPR | ISO 29101 | ISO 42001 | EU AI Act |
|------------|------|-----------|-----------|-----------|
| DP-01, DP-02 | Art. 35, Art. 6, Art. 22 | §6 (privacy requirements) | §8.4 | Art. 10(5), Art. 9 |
| DP-03, DP-04 | Art. 5(1)(c), Art. 25 | §7.1 (data minimization) | §8.5 | Art. 10 |
| DP-05 | Art. 5, Art. 6, Art. 17 | §7.2 (purpose limitation) | §8.4 | Art. 10 |
| DP-06 | Art. 5(1)(f) (integrity) | §7.4 (data quality) | §8.5 | Art. 15 |
| DP-07 | Art. 13-14 (transparency) | §7.1 | §8.5 | Art. 13 |
| DP-08, DP-09 | Art. 15, 17, 22, 33 | §8 (rights implementation) | §9.3 | Art. 14 |
| DP-10 | Art. 17 (right to erasure) | §7.3 | §8.3 | Art. 18 |

---

*Domain D4 | Version 1.0 — 2026-05-29*
