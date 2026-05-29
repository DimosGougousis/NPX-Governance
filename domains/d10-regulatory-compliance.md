# D10 — Regulatory & Standards Compliance

> **Domain purpose:** Map the NXP edge AI governance framework to specific regulatory obligations and standards requirements, maintain a compliance register, and coordinate regulatory activities across all 12 governance domains. Serves as the authoritative cross-reference between internal controls and external obligations.
>
> **Primary standards:** EU AI Act (Regulation (EU) 2024/1689), ISO/IEC 42001 (AI Management System), NIST AI RMF (GOVERN/MAP/MEASURE/MANAGE), ISO/IEC 23894 (AI risk management guidance)
>
> **RACI owner:** Governance Committee (GC) — primary

---

## NXP-Specific Regulatory Context

NXP edge AI systems may fall under several regulatory regimes simultaneously, depending on use case:

| Deployment Context | Primary Regulation | Key Obligations |
|-------------------|--------------------|-----------------|
| General purpose (cross-vertical) | EU AI Act | Risk classification, AIMS, technical documentation |
| Handling EU personal data | GDPR | DPIAs, Art. 22 automated decisions, data minimization |
| Medical device context | EU MDR/IVDR + IEC 62304 | Software as Medical Device (SaMD) conformity |
| Automotive | UNECE WP.29 R155/R156 + ISO/SAE 21434 | Cybersecurity, OTA management system |
| Industrial machinery | Machinery Regulation (EU) 2023/1230 | Safety requirements for AI in machinery |
| Consumer IoT | EU Cyber Resilience Act (from 2027) | Cybersecurity requirements for connected products |

---

## Control Checklist

### Sub-domain 10.1: EU AI Act Compliance (L1)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| RC-01 | **EU AI Act risk classification:** Classify every AI system under EU AI Act risk categories (Art. 6 and Annex III): prohibited (halt), high-risk (full compliance pathway), limited risk (transparency obligations only), minimal risk (voluntary codes). Document the classification with specific Annex III point reference for any high-risk determination. | 🔴 | L1 | Risk classification document with Annex III citations; legal sign-off |
| RC-02 | **AI Management System (AIMS) establishment:** Establish an AI Management System per ISO/IEC 42001 covering: AI policy (§5.2), roles and responsibilities (§5.3), risk management process (§6.1), objectives (§6.2), operational planning (§8), performance evaluation (§9), and continual improvement (§10). For EU AI Act compliance, the AIMS satisfies Art. 9 (quality management system for high-risk AI) requirements. | 🔴 | L1 | AIMS documentation set; ISO/IEC 42001 gap assessment |
| RC-03 | **NIST AI RMF profile:** Create an organizational AI RMF profile mapping the four NIST functions (GOVERN, MAP, MEASURE, MANAGE) to the specific controls in this governance framework. The profile documents which controls satisfy which RMF subcategories and identifies gaps. Update the profile annually. | 🟡 | L1 | NIST AI RMF organizational profile; gap register |

### Sub-domain 10.2: Data & Privacy Compliance (L2)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| RC-04 | **GDPR DPIA register:** Maintain a register of all Data Protection Impact Assessments conducted under GDPR Art. 35 for AI systems processing personal data. Each entry records: system name, DPIA date, DPO sign-off, review schedule, and any residual risks identified. | 🔴 | L2 | DPIA register; DPO sign-off records |

### Sub-domain 10.3: High-Risk AI Compliance Pathway (L3)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| RC-05 | **High-risk AI technical documentation (Annex IV):** For any system classified as high-risk under EU AI Act Art. 6/Annex III, initiate preparation of Annex IV technical documentation: (1) general description; (2) detailed description of design and development; (3) monitoring, functioning, and control information; (4) validation and testing procedures and results; (5) applied standards; (6) EU declaration of conformity; (7) post-market monitoring plan. This documentation must be maintained for 10 years. | 🔴 | L3 | Annex IV documentation set (in progress or complete); 10-year retention plan |

### Sub-domain 10.4: Validation & Certification (L5)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| RC-06 | **Conformity assessment procedure:** For high-risk AI systems, complete the conformity assessment required by EU AI Act Art. 43. For most high-risk AI (not safety component in Annex I machinery/products), this is an internal assessment following Annex VI. For AI used as safety component in CE-marked machinery, a notified body assessment per Annex VII may be required. Document the conformity assessment procedure chosen and its completion. | 🟡 | L5 | Conformity assessment report; EU declaration of conformity (if applicable) |

### Sub-domain 10.5: Deployment (L6)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| RC-07 | **EU AI Act registration (high-risk):** For high-risk AI systems placed on the EU market, register the system in the EU database for high-risk AI systems (Art. 71) before or at the time of deployment. Maintain registration records. | 🔴 | L6 | EU AI Act database registration confirmation |
| RC-08 | **Operator/deployer obligations notification:** If the AI system is deployed by a third party (customer, partner) as the EU AI Act "deployer," provide the deployer with the instructions for use (Art. 13), relevant technical documentation, and notification of their obligations under Art. 26. Document the notification. | 🟡 | L6 | Instructions for use document; deployer notification records |

### Sub-domain 10.6: Operations (L7)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| RC-09 | **Post-market monitoring (EU AI Act Art. 72):** Implement a post-market monitoring plan for high-risk AI systems per EU AI Act Art. 72. The plan must cover: performance monitoring, serious incident reporting (Art. 73 — report to market surveillance authority within 15 days of becoming aware), and near-miss reporting. Integrate with the D9 MLOps monitoring controls. | 🔴 | L7 | Post-market monitoring plan; incident reporting procedure; integration with D9 monitoring |
| RC-10 | **ISO/IEC 42001 continual improvement cycle:** Conduct at minimum an annual internal audit of the AI Management System per ISO/IEC 42001 §9.2. Conduct a management review per §9.3. Document findings, corrective actions, and improvement plans. | 🟡 | L7 | Annual audit records; management review minutes; corrective action log |

### Sub-domain 10.7: Retirement (L8)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| RC-11 | **Regulatory documentation archiving:** Upon system retirement, archive all regulatory documentation (Annex IV technical docs, conformity assessment, EU database registration, DPIAs) for the required retention period. EU AI Act Art. 18 requires 10 years from system retirement for high-risk AI. | 🔴 | L8 | Archive records; retention period confirmation |

### Sub-domain 10.8: Framework Maintenance (Ongoing)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| RC-12 | **Regulatory watch and framework update:** Assign responsibility to the Governance Committee for monitoring relevant regulatory developments (EU AI Act implementing acts, ISO/IEC 42001 updates, NIST AI RMF updates, sector-specific regulations). Update this governance framework within 90 days of material regulatory changes. | 🟡 | All | Regulatory watch procedure; update log with change dates |

---

## Standards Cross-Reference Matrix

| Framework | Key Articles/Sections | Satisfied By |
|-----------|----------------------|-------------|
| **EU AI Act** | Art. 5 (prohibited) | D6 (EU-01, EU-02, EU-10) |
| | Art. 9 (quality management) | D12 (FS-01–FS-09), RC-02 |
| | Art. 10 (training data) | D1 (DG-01–DG-08), D5 (FB-01–FB-06) |
| | Art. 11 + Annex IV (technical docs) | RC-05, D7 (XT-06), D9 (ML-04) |
| | Art. 13 (transparency) | D7 (XT-01–XT-06) |
| | Art. 14 (human oversight) | D8 (HO-01–HO-09) |
| | Art. 15 (accuracy/robustness) | D9 (ML-05–ML-09), D3 (SR-06) |
| | Art. 72 (post-market monitoring) | D9 (ML-13–ML-15), RC-09 |
| **ISO/IEC 42001** | §5.2 (AI policy) | RC-02, D6 (EU-10) |
| | §6.1 (risk management) | D12 (FS-01–FS-02), D3 (SR-01) |
| | §8.3 (lifecycle) | D9 (all ML controls) |
| | §8.4 (documentation) | D1 (DG-02–DG-05), D7 (XT-06) |
| | §9.1 (performance eval) | D9 (ML-13–ML-15), RC-10 |
| **NIST AI RMF** | GOVERN 1.1–6.2 | RC-02, D6, D8 |
| | MAP 1–5 | D12 (FS-01–FS-02), D5 (FB-01) |
| | MEASURE 2.5–2.7 | D3 (SR-06), D5 (FB-04), D9 (ML-05) |
| | MANAGE 3–4 | D8 (HO-07–HO-09), D9 (ML-13–ML-17) |
| **GDPR** | Art. 22 (automated decisions) | D4 (DP-01, DP-02, DP-08) |
| | Art. 25 (privacy by design) | D4 (DP-03, DP-04) |
| | Art. 35 (DPIA) | RC-04, D4 (DP-01) |

---

*Domain D10 | Version 1.0 — 2026-05-29*
