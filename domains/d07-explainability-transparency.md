# D7 — Explainability & Transparency

> **Domain purpose:** Govern the explainability of AI decisions made by NXP edge devices, the integrity and completeness of on-device audit trails, model documentation standards, and transparency obligations to operators, end users, and regulators. Ensures that every significant AI decision is traceable, explainable, and logged without compromising device performance or privacy.
>
> **Primary standards:** IEEE 7001 (Transparency of autonomous systems), ISO/IEC TR 29119-11 (AI testing — explainability), EU AI Act Art. 13 (transparency for high-risk AI), Art. 50 (transparency obligations)
>
> **RACI owner:** Governance Committee (GC) — primary; Model Risk Officer (MRO) — secondary

---

## NXP-Specific Context

On-device processing is the framework's primary auditability differentiator: because all inference happens locally, complete logs of every query, response, and decision can be maintained without cloud dependency. However, this advantage creates obligations:

1. **Log integrity on constrained hardware.** Audit logs stored on the device itself can be tampered with if the device is compromised. Log integrity must be enforced — either through hardware-backed tamper protection or by signing log entries with the device's hardware key.

2. **Lightweight XAI.** Full SHAP/LIME computations are impractical on resource-constrained NXP devices. Governance must define which XAI approaches are feasible at the Neutron NPU's computational budget (attention visualization, confidence scores, RAG citation enforcement) vs. which require offline post-hoc analysis.

3. **RAG citation as built-in explainability.** When the model cites the specific local document(s) that grounded its response, this is a natural explainability mechanism. Governance should mandate this and define what a valid citation looks like.

---

## Control Checklist

### Sub-domain 7.1: Explainability Architecture (L1)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| XT-01 | **Explainability approach per AI function:** For each AI function, define the explainability approach to be implemented: (a) inherently interpretable model (decision tree, linear model) — no additional XAI needed; (b) lightweight XAI on-device (confidence scores, attention visualization, RAG citations); (c) post-hoc offline XAI (SHAP/LIME computed off-device after log retrieval). Document the rationale for the choice, including computational budget constraints. | 🔴 | L1 | Explainability approach document per AI function |

### Sub-domain 7.2: Training-Time Documentation (L3)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| XT-02 | **Global model explanation:** Produce a global explanation of model behavior — which input features or document categories most strongly influence outputs. For RAG systems, document which document categories most frequently contribute to responses. Update at each major model version change. | 🟡 | L3 | Global model explanation artifact; feature importance or document category analysis |

### Sub-domain 7.3: Quantization Explainability (L4)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| XT-03 | **Quantization explanation impact assessment:** Assess whether quantization changes the model's explanation behavior (e.g., changes which features are most salient according to the XAI method). If the explainability method is sensitive to quantization, document this limitation and use the post-quantization model for all XAI computations. | 🟡 | L4 | Explainability comparison: FP32 vs. quantized model explanations on representative samples |

### Sub-domain 7.4: Validation (L5)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| XT-04 | **Audit log design and integrity:** Design the on-device audit log to capture for every significant AI decision: (a) timestamp; (b) input summary (not raw PII — a hash or abstracted description); (c) model version and quantization format; (d) output / decision; (e) confidence score; (f) RAG citations (for LLM-based decisions); (g) human action taken (if applicable). Log entries must be integrity-protected (e.g., signed with device hardware key or chained hash). | 🔴 | L5 | Audit log specification; integrity protection implementation; sample log entries |
| XT-05 | **XAI fidelity validation:** Validate that the chosen XAI method accurately reflects the model's actual decision process — not a simplified approximation that is misleading. For RAG citation enforcement, test that cited documents actually influenced the response. | 🟡 | L5 | XAI fidelity test results; RAG citation grounding tests |

### Sub-domain 7.5: Deployment (L6)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| XT-06 | **Model card (final, deployment-ready):** Produce a final model card for each deployed model covering: intended use, out-of-scope uses, training data summary, performance metrics (overall and disaggregated), known limitations, fairness assessment summary, explainability approach, and contact for questions. Publish the model card to the internal governance repository. For EU AI Act high-risk systems, the model card forms part of the Annex IV technical documentation. | 🔴 | L6 | Final model card per deployed model version; repository location |

### Sub-domain 7.6: Operations (L7)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| XT-07 | **Audit log retention and retrieval:** Define and implement the on-device audit log retention period (minimum: the duration required by applicable regulations — EU AI Act Art. 19 requires 10 years for high-risk AI technical documentation; operational logs may differ). Implement a log retrieval mechanism for authorized access (incident investigation, regulatory inspection, data subject access request). | 🔴 | L7 | Retention policy; retrieval capability demonstration; authorized access controls |
| XT-08 | **Decision explainability on demand:** For any AI-assisted decision that affects an individual or a safety-relevant outcome, maintain the ability to reconstruct and explain the decision after the fact: which model version, which input, which RAG documents, what the confidence was. Test this capability annually. | 🔴 | L7 | Decision reconstruction capability test; access procedure documentation |

### Sub-domain 7.7: Retirement (L8)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| XT-09 | **Audit log archiving on retirement:** When a device is decommissioned, retrieve and archive its audit logs before data purge. Archive the audit log for the required retention period. Logs must be archived in an integrity-preserved, auditable format. | 🟡 | L8 | Log archiving procedure; archive location and access controls; retention confirmation |

---

## Standards Cross-Reference

| Control(s) | IEEE 7001 | ISO/IEC TR 29119-11 | EU AI Act | GDPR |
|------------|-----------|--------------------|-----------|----|
| XT-01 | §5.1 (transparency levels) | §6.1 (XAI approach) | Art. 13(1) | — |
| XT-02 | §5.3 (global transparency) | §6.3 | Art. 13(1)(d) | — |
| XT-03, XT-04 | §5.4 (traceability) | §6.5 | Art. 19 (logging) | Art. 5(1)(f) |
| XT-05 | §5.2 (explanation fidelity) | §6.4 | Art. 13 | — |
| XT-06 | §5.5 (documentation) | §7.1 | Art. 11, Annex IV | — |
| XT-07, XT-08 | §5.4 | §7.2 | Art. 19, Art. 72 | Art. 15 |
| XT-09 | — | — | Art. 18 | Art. 17 |

---

*Domain D7 | Version 1.0 — 2026-05-29*
