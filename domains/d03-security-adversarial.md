# D3 — Security & Adversarial Resilience

> **Domain purpose:** Govern the security posture of NXP edge AI systems against both conventional cyber threats and AI-specific adversarial attacks. Covers the MITRE ATLAS threat landscape for embedded AI, software supply chain integrity (Model SBOM), AI framework CVE management, OTA code signing, and device hardening beyond the NXP hardware root of trust.
>
> **Primary standards:** MITRE ATLAS, ETSI EN 303 645 (IoT cybersecurity), IEC 62443 (industrial security), NIST IR 8259A (IoT device cybersecurity), ISO/SAE 21434 (automotive cybersecurity — reference for all verticals)
>
> **RACI owner:** Device Security Lead (DSL) — primary

---

## NXP-Specific Context

NXP's hardware security stack (secure boot, TrustZone, Hardware Root of Trust, remote attestation) provides a strong device foundation. However, it addresses **physical and firmware-level threats** — not AI-layer threats. This domain extends security governance into the software and AI model layers where:

1. **AI models are attack targets.** A model can be extracted (model theft), corrupted (data/model poisoning), fooled (adversarial examples), or abused (membership inference) through the inference API — none of which the hardware security stack prevents.

2. **The AI framework is an attack surface.** TensorFlow Lite, ONNX Runtime, and the eIQ GenAI Flow runtime itself are software libraries with CVE histories. The governance program must track and patch these as it would any embedded firmware dependency.

3. **OTA model updates are a high-value attack target.** A compromised OTA channel allows an attacker to replace a legitimate model with a malicious one across the entire device fleet. The model signing chain must be as robust as firmware signing.

4. **Supply chain extends to foundation model weights.** When using open-source foundation models (Llama, Mistral, etc.), the model weights themselves are a supply chain artifact. Provenance verification must confirm you are deploying the weights published by the declared source, not a tampered variant.

---

## Control Checklist

### Sub-domain 3.1: Threat Modeling (L1)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| SR-01 | **MITRE ATLAS threat model:** Complete a MITRE ATLAS-based threat model for the AI system before development begins. Map applicable tactics and techniques to the specific deployment context (device type, connectivity, use case, input channels). Include at minimum: AML.T0048 (adversarial examples), AML.T0043 (craft adversarial data), AML.T0054 (LLM prompt injection), AML.T0044 (full model access), AML.T0025 (exfiltrate model). Assign mitigations to each identified technique. | 🔴 | L1 | MITRE ATLAS threat model document; mitigations mapped per technique |

### Sub-domain 3.2: Supply Chain Security (L2/L3)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| SR-02 | **Training data integrity:** Verify the integrity of all third-party training datasets before use. For public datasets, verify against published checksums or hashes. For proprietary data, implement access controls and audit logging on storage. Document provenance of all training data sources in the D1 data source registry. | 🔴 | L2 | Dataset integrity verification records; access control documentation |
| SR-03 | **AI framework and library CVE tracking:** Maintain an inventory of all AI/ML libraries used in the build (TFLite, ONNX Runtime, eIQ SDK, vector DB library, etc.) with their exact versions. Subscribe to CVE feeds for each library. Define a patch policy: critical CVEs patched within 30 days; high severity within 90 days. Track and document patch status. | 🔴 | L3 | Software inventory with versions; CVE monitoring subscription records; patch status log |
| SR-04 | **Model SBOM (Model Bill of Materials):** For every deployed model, produce a Model SBOM documenting: base model name and version, source URL and checksum, fine-tuning datasets (with checksums), quantization toolchain and version, and any third-party model components or adapters used. The SBOM must be signed and stored alongside the model artifact. | 🔴 | L3 | Signed Model SBOM per deployed model version |

### Sub-domain 3.3: Model & Quantization Security (L4)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| SR-05 | **Foundation model weight provenance verification:** When using open-source foundation model weights, verify the SHA-256 checksum of downloaded weights against the checksum published by the original model provider (Hugging Face model card, official release page). Never deploy weights whose provenance cannot be verified. | 🔴 | L4 | Checksum verification records per model download; source URL documentation |

### Sub-domain 3.4: Adversarial Robustness Testing (L5)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| SR-06 | **Adversarial example testing:** Prior to deployment, test the model against adversarial input attacks relevant to the input modality (image: FGSM, PGD; text: character substitution, token manipulation; sensor: noise injection). Measure adversarial robustness metrics and compare against acceptance thresholds. Document testing methodology and results. | 🔴 | L5 | Adversarial robustness test report; acceptance thresholds; pass/fail determination |
| SR-07 | **Model extraction / side-channel resistance:** If the device exposes an inference API (local or networked), assess the risk of model extraction via repeated queries. Implement rate limiting and query logging on the inference endpoint. For high-risk deployments, consider output perturbation to reduce extraction risk. | 🟡 | L5 | Inference API security assessment; rate limiting configuration |
| SR-08 | **Membership inference assessment:** For models trained on sensitive data (medical, biometric, behavioral), assess the risk of membership inference attacks — an adversary determining whether a specific individual's data was in the training set. Document the risk level and mitigations (differential privacy, output regularization). | 🟡 | L5 | Membership inference risk assessment; mitigation documentation |

### Sub-domain 3.5: OTA Security (L6)

| ID | Control | Purpose | Priority | Stage | Evidence Required |
|----|---------|---------|----------|-------|-------------------|
| SR-09 | **Model artifact signing:** All model artifacts (weights, quantization configs, ONNX/TFLite files, RAG vector DB packages) must be cryptographically signed by the DSL before being published to the OTA distribution channel. Devices must verify the signature before loading any new model artifact. Use NXP's hardware-backed key storage for signature verification keys. | 🔴 | L6 | Signing infrastructure documentation; key management procedure; device-side verification implementation |
| SR-10 | **OTA channel security:** The OTA distribution channel (server and transport) must use mutual TLS (mTLS) for all model update transfers. The device must authenticate the OTA server (certificate pinning or equivalent). OTA packages must include an integrity manifest (hash of all included files) signed separately from individual artifacts. | 🔴 | L6 | OTA channel security architecture; mTLS configuration; integrity manifest implementation |

### Sub-domain 3.6: Runtime Security (L7)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| SR-11 | **Inference rate limiting and DoS protection:** Implement rate limiting on the local inference API to prevent Model Denial of Service attacks (OWASP LLM04) — where an attacker floods the inference engine with expensive requests to exhaust NPU resources. Define maximum requests/second per calling application and enforce at the runtime level. | 🔴 | L7 | Rate limiting configuration; load test results demonstrating protection |
| SR-12 | **Runtime anomaly detection:** Monitor inference patterns for anomalous behavior indicative of attack or misuse: unusual query rates, queries probing model boundaries (potential extraction), repeated adversarial-pattern inputs, or queries seeking model architecture information. Alert on anomalies. | 🟡 | L7 | Anomaly detection configuration; alert threshold definitions; example alert |
| SR-13 | **Security incident playbook for AI systems:** Maintain a specific incident response playbook for AI-layer security incidents (adversarial attack detected, model theft suspected, poisoning detected, OTA compromise suspected). Define escalation paths to DSL and AIPO. For OTA compromise: immediate fleet-wide rollback procedure. | 🔴 | L7 | AI security incident playbook; tested rollback procedure |

### Sub-domain 3.7: Retirement Security (L8)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| SR-14 | **Cryptographic key rotation on model retirement:** When a model version is retired and its signing keys are no longer needed, rotate or revoke the associated cryptographic keys. Document key lifecycle from creation through revocation. Ensure retired key material is securely destroyed. | 🟡 | L8 | Key lifecycle policy; revocation records |

---

## MITRE ATLAS Technique Coverage Map

| Technique ID | Name | NXP Control | Domain |
|-------------|------|-------------|--------|
| AML.T0048 | Adversarial example crafting | SR-06 | D3 |
| AML.T0043 | Craft adversarial data (training) | SR-02, DG-07 | D3, D1 |
| AML.T0054 | LLM prompt injection | LM-05, LM-06 | D2 |
| AML.T0044 | Full model access (extraction) | SR-07, SR-09 | D3 |
| AML.T0025 | Exfiltrate model | SR-07, SR-09, SR-10 | D3 |
| AML.T0020 | Poison training data | SR-02, DG-07, LM-12 | D3, D1, D2 |
| AML.T0027 | Membership inference | SR-08 | D3 |
| AML.T0040 | ML supply chain compromise | SR-03, SR-04, SR-05 | D3 |

---

## Standards Cross-Reference

| Control(s) | MITRE ATLAS | ETSI EN 303 645 | IEC 62443 | NIST AI RMF |
|------------|-------------|----------------|-----------|-------------|
| SR-01 | All relevant techniques | §4.1 | §SR-2 | MAP 5.1 |
| SR-02, SR-03, SR-04, SR-05 | AML.T0040, AML.T0020 | §4.7 (software updates) | §SR-3 | MANAGE 2.4 |
| SR-06, SR-07, SR-08 | AML.T0048, AML.T0044 | — | §SR-5 | MEASURE 2.5 |
| SR-09, SR-10 | AML.T0044, AML.T0025 | §4.4 (secure communication) | §SR-7 | MANAGE 3.2 |
| SR-11, SR-12, SR-13 | AML.T0057 (denial of ML service) | §4.3 (keep sw updated) | §FR-6 | MANAGE 4.1 |
| SR-14 | — | §4.5 (secure storage) | §SR-1 | GOVERN 4.2 |

---

*Domain D3 | Version 1.0 — 2026-05-29*
