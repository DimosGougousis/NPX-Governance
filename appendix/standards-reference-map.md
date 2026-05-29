# Appendix — Standards & Regulatory Reference Map

> **Purpose:** Full cross-reference between external standards/regulations and the governance domains and controls in this framework. Use this appendix to map evidence collected under one standard to multiple applicable requirements, or to identify which controls satisfy a specific regulatory audit point.

---

## EU AI Act — Article-Level Mapping

| EU AI Act Article | Topic | Satisfying Domains & Controls |
|------------------|-------|-------------------------------|
| Art. 5 | Prohibited AI practices | D6: EU-01, EU-02, EU-10 |
| Art. 6 + Annex III | High-risk AI classification | D10: RC-01 |
| Art. 9 | Quality management system (high-risk) | D10: RC-02; D12: FS-01–FS-09 |
| Art. 10 | Training, validation, testing data | D1: DG-01–DG-08; D5: FB-01–FB-06 |
| Art. 11 + Annex IV | Technical documentation | D10: RC-05; D7: XT-06; D9: ML-03, ML-04 |
| Art. 13 | Transparency / instructions for use | D7: XT-01–XT-06; D10: RC-08 |
| Art. 14 | Human oversight (high-risk) | D8: HO-01–HO-09 |
| Art. 15 | Accuracy, robustness, cybersecurity | D9: ML-05–ML-09; D3: SR-06; D5: FB-04 |
| Art. 17 | Quality management documentation | D9: ML-01–ML-04 |
| Art. 18 | Documentation retention (10 years) | D7: XT-09; D9: ML-16; D10: RC-11 |
| Art. 19 + Art. 72 | Logging for high-risk AI | D7: XT-04, XT-07, XT-08 |
| Art. 35 (GDPR Art. 35 ref.) | DPIA for high-risk AI | D4: DP-01; D10: RC-04 |
| Art. 43 | Conformity assessment | D10: RC-06 |
| Art. 50 | Transparency for limited-risk AI | D7: XT-06; D8: HO-06 |
| Art. 51 | GPAI model transparency | D2: LM-01, LM-02 |
| Art. 71 | EU AI Act registration database | D10: RC-07 |
| Art. 72 | Post-market monitoring | D9: ML-13–ML-15; D10: RC-09 |
| Art. 73 | Serious incident reporting (15 days) | D10: RC-09 |

---

## ISO/IEC 42001 — Section Mapping

| ISO/IEC 42001 Section | Topic | Satisfying Controls |
|----------------------|-------|---------------------|
| §4 Context of the organization | Understanding context, stakeholders | D10: RC-02 |
| §5.2 AI policy | Organizational AI policy | D6: EU-10; D10: RC-02 |
| §5.3 Roles and responsibilities | Governance structure | All domain RACI assignments |
| §6.1 Risk management planning | Risk identification and treatment | D12: FS-01–FS-02; D3: SR-01 |
| §6.2 AI objectives | Measurable objectives | D10: RC-02 |
| §8.3 AI system lifecycle | Design, development, deployment, retirement | D9: ML-01–ML-17 |
| §8.4 AI system documentation | Technical and operational documentation | D1: DG-02–DG-05; D7: XT-06 |
| §8.5 Operational controls | Deployment and operation controls | D2: LM-03–LM-11; D4: DP-03–DP-07 |
| §9.1 Monitoring and measurement | Performance evaluation | D9: ML-13–ML-15 |
| §9.2 Internal audit | AIMS audit | D10: RC-10 |
| §9.3 Management review | Governance review | D10: RC-10 |
| §10 Improvement | Corrective actions, continual improvement | D10: RC-10, RC-12 |
| Annex G | Human oversight | D8: HO-01–HO-10 |

---

## NIST AI RMF — Function Mapping

| NIST AI RMF Function / Category | Satisfying Controls |
|----------------------------------|---------------------|
| **GOVERN 1** — Policies, processes, procedures | D6: EU-10; D10: RC-02; D8: HO-01 |
| **GOVERN 2** — Accountability and transparency | D7: XT-01–XT-08; D8: HO-04 |
| **GOVERN 4** — Organizational teams | All domain RACI assignments |
| **GOVERN 6** — Policies for AI risks | D6: EU-01–EU-03; D12: FS-12 |
| **MAP 1** — Context establishment | D12: FS-01–FS-02; D6: EU-01–EU-03 |
| **MAP 2** — Categorization | D10: RC-01; D12: FS-02 |
| **MAP 5** — Impacts, likelihoods, risks | D5: FB-01; D12: FS-01 |
| **MEASURE 2.5** — AI system testing | D3: SR-06; D5: FB-04; D9: ML-05–ML-09 |
| **MEASURE 2.6** — Performance metrics | D9: ML-13–ML-15; D5: FB-10 |
| **MEASURE 2.7** — Validation | D12: FS-07–FS-09 |
| **MANAGE 2** — Risk treatment | D8: HO-01–HO-05; D9: ML-10–ML-12 |
| **MANAGE 3** — Response and recovery | D3: SR-13; D8: HO-08; D9: ML-14 |
| **MANAGE 4** — Residual risk monitoring | D9: ML-13; D5: FB-10; D12: FS-11 |

---

## GDPR — Article Mapping

| GDPR Article | Topic | Satisfying Controls |
|--------------|-------|---------------------|
| Art. 5 | Principles (purpose limitation, minimization) | D4: DP-03; D1: DG-12 |
| Art. 6 | Legal basis for processing | D4: DP-02 |
| Art. 13–14 | Information to data subjects | D7: XT-06 (model card); D4: DP-07 |
| Art. 15 | Right of access | D4: DP-08; D7: XT-07 |
| Art. 17 | Right to erasure | D4: DP-10; D1: DG-12; D9: ML-17 |
| Art. 22 | Automated decision-making | D4: DP-01, DP-02, DP-08; D8: HO-04 |
| Art. 25 | Privacy by design and default | D4: DP-03, DP-04 |
| Art. 33 | Breach notification (72h) | D4: DP-09 |
| Art. 35 | Data Protection Impact Assessment | D4: DP-01; D10: RC-04 |

---

## IEC 61508 / ISO 26262 — Mapping for Functional Safety

| Standard Section | Topic | Satisfying Controls |
|-----------------|-------|---------------------|
| IEC 61508 §7.4 | Hazard and risk analysis | D12: FS-01 |
| IEC 61508 §7.9 | Safety requirements specification | D12: FS-02, FS-04 |
| IEC 61508 §7.11 | Integration and testing | D12: FS-05, FS-08 |
| IEC 61508 §8.3 | Independence requirements (SIL ≥ 2) | D12: FS-09 |
| ISO 26262 Part 3 §7 | HARA | D12: FS-01 |
| ISO 26262 Part 6 | Software level (quantization) | D12: FS-05, FS-06 |
| ISO/IEC TR 5469 §6.3 | AI risk classification | D12: FS-02 |
| ISO/IEC TR 5469 §6.4 | Safety design for AI | D12: FS-03, FS-04, FS-10 |
| ISO/IEC TR 5469 §6.5 | AI safety validation | D12: FS-07, FS-08 |

---

## MITRE ATLAS — Technique Coverage Summary

| MITRE ATLAS Tactic | Covered Techniques | Covering Controls |
|-------------------|-------------------|-------------------|
| Reconnaissance | AML.T0000 (model discovery) | D3: SR-07, SR-11 |
| Resource Development | AML.T0002 (acquire access) | D3: SR-03, SR-04 |
| ML Attack Staging | AML.T0043 (craft adversarial) | D3: SR-06 |
| Exfiltration | AML.T0025 (exfiltrate model), AML.T0044 | D3: SR-07, SR-09, SR-10 |
| Impact | AML.T0057 (denial of ML service) | D3: SR-11 |
| Persistence | AML.T0020 (poison training) | D1: DG-07; D2: LM-12 |
| Defense Evasion | AML.T0054 (prompt injection) | D2: LM-05, LM-06 |

---

## Key Acronym Reference

| Acronym | Meaning |
|---------|---------|
| AIMS | AI Management System (ISO/IEC 42001) |
| AIPO | AI Product Owner |
| ASIL | Automotive Safety Integrity Level (ISO 26262) |
| DPIA | Data Protection Impact Assessment (GDPR Art. 35) |
| DSL | Device Security Lead |
| DPO | Data Protection Officer |
| FSE | Functional Safety Engineer |
| FTTI | Fault-Tolerant Time Interval |
| GC | Governance Committee |
| GPAI | General-Purpose AI (EU AI Act Title VIII) |
| HARA | Hazard Analysis and Risk Assessment |
| HRoT | Hardware Root of Trust |
| MRO | Model Risk Officer |
| NIST AI RMF | National Institute of Standards and Technology AI Risk Management Framework |
| ODD | Operational Design Domain |
| OTA | Over-The-Air (model/firmware update) |
| RAG | Retrieval-Augmented Generation |
| SBOM | Software Bill of Materials |
| SIL | Safety Integrity Level (IEC 61508) |
| SLM | Small Language Model |
| TRCM | Trust, Risk, and Cybersecurity Management (file) |
| XAI | Explainable Artificial Intelligence |

---

*Appendix | Version 1.0 — 2026-05-29*
