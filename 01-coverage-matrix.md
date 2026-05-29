# NXP Edge AI Governance Framework — Coverage Matrix

> **How to read:** Each cell lists the control IDs that apply at that lifecycle stage for that domain. Cells marked `—` indicate no controls at that stage. Use this matrix to identify gaps in your current governance posture or to plan governance activities for a new project.

---

## Domain × Lifecycle Coverage Matrix

| Domain | L1 Conception | L2 Data | L3 Training | L4 Compression | L5 Validation | L6 Deployment | L7 Operations | L8 Retirement |
|--------|--------------|---------|-------------|----------------|---------------|---------------|---------------|---------------|
| **D1 Data Governance** | DG-01 | DG-02 DG-03 DG-04 DG-05 | DG-06 DG-07 | — | DG-08 | DG-09 | DG-10 DG-11 | DG-12 |
| **D2 LLM/SLM Governance** | LM-01 LM-02 | — | LM-03 LM-04 | LM-05 LM-06 | LM-07 LM-08 LM-09 | LM-10 LM-11 | LM-12 LM-13 LM-14 | LM-15 |
| **D3 Security & Adversarial** | SR-01 | SR-02 | SR-03 | SR-04 SR-05 | SR-06 SR-07 SR-08 | SR-09 SR-10 | SR-11 SR-12 SR-13 | SR-14 |
| **D4 Privacy & Data Protection** | DP-01 DP-02 | DP-03 DP-04 | DP-05 | — | DP-06 | DP-07 | DP-08 DP-09 | DP-10 |
| **D5 Fairness & Bias** | FB-01 | FB-02 FB-03 | FB-04 FB-05 | FB-06 | FB-07 FB-08 | FB-09 | FB-10 FB-11 | — |
| **D6 Ethics & Acceptable Use** | EU-01 EU-02 EU-03 | EU-04 | EU-05 | — | EU-06 EU-07 | EU-08 | EU-09 EU-10 | EU-11 |
| **D7 Explainability & Transparency** | XT-01 | — | XT-02 | XT-03 | XT-04 XT-05 | XT-06 | XT-07 XT-08 | XT-09 |
| **D8 Human Oversight & Control** | HO-01 HO-02 | — | — | — | HO-03 HO-04 | HO-05 HO-06 | HO-07 HO-08 HO-09 | HO-10 |
| **D9 MLOps & Model Lifecycle** | ML-01 | ML-02 | ML-03 ML-04 | ML-05 ML-06 ML-07 | ML-08 ML-09 | ML-10 ML-11 ML-12 | ML-13 ML-14 ML-15 | ML-16 ML-17 |
| **D10 Regulatory Compliance** | RC-01 RC-02 RC-03 | RC-04 | RC-05 | RC-06 | RC-07 RC-08 | RC-09 RC-10 | RC-11 | RC-12 |
| **D11 Environmental** | ES-01 | ES-02 | — | ES-03 | — | ES-04 | ES-05 ES-06 | ES-07 |
| **D12 Functional Safety** | FS-01 FS-02 | FS-03 | FS-04 | FS-05 FS-06 | FS-07 FS-08 FS-09 | FS-10 | FS-11 FS-12 | FS-13 |

---

## Coverage Summary by Stage

| Stage | Critical 🔴 Controls | Important 🟡 Controls | Recommended 🟢 Controls |
|-------|---------------------|----------------------|------------------------|
| L1 Conception | ~18 | ~8 | ~3 |
| L2 Data | ~12 | ~10 | ~4 |
| L3 Training | ~10 | ~8 | ~3 |
| L4 Compression | ~8 | ~5 | ~2 |
| L5 Validation | ~15 | ~10 | ~4 |
| L6 Deployment | ~10 | ~8 | ~3 |
| L7 Operations | ~14 | ~12 | ~5 |
| L8 Retirement | ~6 | ~5 | ~2 |

---

## Coverage Summary by Domain

| Domain | Total Controls | 🔴 Critical | 🟡 Important | 🟢 Recommended |
|--------|---------------|------------|-------------|----------------|
| D1 Data Governance | 12 | 7 | 4 | 1 |
| D2 LLM/SLM Governance | 15 | 8 | 5 | 2 |
| D3 Security & Adversarial | 14 | 9 | 4 | 1 |
| D4 Privacy & Data Protection | 10 | 7 | 2 | 1 |
| D5 Fairness & Bias | 11 | 6 | 4 | 1 |
| D6 Ethics & Acceptable Use | 11 | 6 | 4 | 1 |
| D7 Explainability & Transparency | 9 | 5 | 3 | 1 |
| D8 Human Oversight & Control | 10 | 7 | 2 | 1 |
| D9 MLOps & Model Lifecycle | 17 | 10 | 5 | 2 |
| D10 Regulatory Compliance | 12 | 7 | 4 | 1 |
| D11 Environmental & Sustainability | 7 | 2 | 3 | 2 |
| D12 Functional Safety | 13 | 8 | 4 | 1 |
| **Total** | **141** | **82** | **44** | **15** |

---

## Minimum Viable Governance (MVG) — Pre-Deployment Gate

Before any NXP edge AI system enters production (L6), all of the following 🔴 controls **must** be complete:

| Control | Domain | Stage |
|---------|--------|-------|
| DG-01: Data source registry | D1 | L1 |
| LM-01: Model license review | D2 | L1 |
| SR-01: MITRE ATLAS threat model | D3 | L1 |
| DP-01: DPIA assessment | D4 | L1 |
| EU-01: Ethics review gate | D6 | L1 |
| EU-02: EU AI Act risk classification | D6 | L1 |
| HO-01: Autonomy level classification | D8 | L1 |
| ML-01: Model version registry initialized | D9 | L1 |
| RC-01: Regulatory scope determination | D10 | L1 |
| FS-01: Hazard analysis completed | D12 | L1 |
| FB-04: Pre-deployment bias testing | D5 | L3 |
| ML-05: Quantization accuracy regression | D9 | L4 |
| ML-06: Safety property preservation check | D9 | L4 |
| SR-06: Adversarial robustness testing | D3 | L5 |
| FS-07: Functional safety review sign-off | D12 | L5 |
| ML-10: OTA authorization chain verified | D9 | L6 |
| ML-11: Staged rollout plan approved | D9 | L6 |
| HO-05: Kill-switch tested and operational | D8 | L6 |

---

*Version 1.0 — 2026-05-29*
