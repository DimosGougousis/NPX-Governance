# NXP Edge AI Governance Framework — Overview

> **Version:** 1.0 | **Date:** 2026-05-29 | **Classification:** Internal / Public
>
> **Scope:** Cross-vertical operational governance framework for AI/ML systems deployed on NXP edge hardware (i.MX 93, i.MX 8M Plus, i.MX 95, automotive/medical-grade MPUs with Neutron NPUs) powered by the NXP eIQ GenAI Flow pipeline.
>
> **Not in scope:** Cloud-hosted AI, web application AI, financial sector AI (see companion DNB SAFEST checklist).

---

## 1. Purpose

This framework provides the governance policies, roles, control checklists, and compliance evidence templates required to deploy AI systems responsibly on NXP edge devices. It covers **12 governance domains** across an **8-stage AI lifecycle**, and is designed to satisfy the management requirements of ISO/IEC 42001, the risk controls of the NIST AI RMF, the regulatory obligations of the EU AI Act, and the device-level security requirements of NXP hardware.

It is structured as a **Domain × Lifecycle Matrix**: practitioners navigate by domain (what aspect of governance they need) or by lifecycle stage (what stage of deployment they are at). Each domain document contains a prioritized control checklist with evidence requirements.

---

## 2. The Three-Tiered Governance Spine

The framework is built on three interlocking governance tiers that form its backbone:

```
┌──────────────────────────────────────────────────────────┐
│  TIER 1 — MANAGEMENT & COMPLIANCE                        │
│  ISO/IEC 42001 (AI Management System)                    │
│  Defines: policies, roles, continuous improvement,       │
│  EU AI Act AIMS obligations                              │
└──────────────────┬───────────────────────────────────────┘
                   │
┌──────────────────▼───────────────────────────────────────┐
│  TIER 2 — RISK ENGINEERING                               │
│  NIST AI RMF (GOVERN · MAP · MEASURE · MANAGE)          │
│  Defines: edge-specific risk controls — local privacy,  │
│  limited observability, model update drift               │
└──────────────────┬───────────────────────────────────────┘
                   │
┌──────────────────▼───────────────────────────────────────┐
│  TIER 3 — DEVICE & OPERATIONS                            │
│  NXP Hardware + IoT Security                             │
│  Defines: secure boot, remote attestation, hardware      │
│  root of trust, local failsafes, Neutron NPU controls    │
└──────────────────────────────────────────────────────────┘
```

The **12 governance domains** in this framework are cross-cutting disciplines that operate across all three tiers. The table below shows each domain's primary tier alignment:

| Domain | Primary Tier | Secondary Tier |
|--------|-------------|----------------|
| D1 Data Governance | Tier 2 (Risk) | Tier 1 (Management) |
| D2 LLM / SLM Governance | Tier 2 (Risk) | Tier 3 (Device) |
| D3 Security & Adversarial Resilience | Tier 3 (Device) | Tier 2 (Risk) |
| D4 Privacy & Data Protection | Tier 1 (Management) | Tier 2 (Risk) |
| D5 Fairness & Bias | Tier 2 (Risk) | Tier 1 (Management) |
| D6 Ethics & Acceptable Use | Tier 1 (Management) | Tier 2 (Risk) |
| D7 Explainability & Transparency | Tier 2 (Risk) | Tier 3 (Device) |
| D8 Human Oversight & Control | Tier 1 (Management) | Tier 3 (Device) |
| D9 MLOps & Model Lifecycle | Tier 2 (Risk) | Tier 3 (Device) |
| D10 Regulatory & Standards Compliance | Tier 1 (Management) | Tier 2 (Risk) |
| D11 Environmental & Sustainability | Tier 1 (Management) | Tier 3 (Device) |
| D12 Functional Safety | Tier 3 (Device) | Tier 2 (Risk) |

---

## 3. Framework Architecture

### The 8 Lifecycle Stages

| # | Stage | Description |
|---|-------|-------------|
| **L1** | Conception & Design | Use case approval, ethics review, AI Act risk classification, DPIA trigger assessment |
| **L2** | Data | Collection, labeling, curation, RAG knowledge base build, lineage documentation |
| **L3** | Training & Development | Model development, dataset versioning, bias testing, model card draft |
| **L4** | Compression & Quantization | INT8/INT4 conversion, ONNX/TFLite export, accuracy regression testing, safety property preservation |
| **L5** | Validation & Testing | Security testing, fairness testing, adversarial robustness, XAI validation, functional safety review |
| **L6** | Deployment & Fleet Management | OTA update authorization, staged rollout (canary → fleet), device registration, final model card |
| **L7** | Operation & Monitoring | Drift detection, incident response, telemetry governance, human oversight monitoring |
| **L8** | Retirement | End-of-life procedures, data purge, device decommission, documentation archiving |

### The 12 Governance Domains

| # | Domain | Primary Standards |
|---|--------|-------------------|
| **D1** | Data Governance | ISO/IEC 25012, DAMA-DMBOK |
| **D2** | LLM / SLM Governance | OWASP LLM Top 10, MITRE ATLAS |
| **D3** | Security & Adversarial Resilience | ETSI EN 303 645, IEC 62443, MITRE ATLAS |
| **D4** | Privacy & Data Protection | GDPR Art. 22/25, ISO 29101 |
| **D5** | Fairness & Bias | IEEE 7003, ISO/IEC TR 24027 |
| **D6** | Ethics & Acceptable Use | EU AI Act Art. 5, OECD AI Principles |
| **D7** | Explainability & Transparency | IEEE 7001, ISO/IEC TR 29119-11 |
| **D8** | Human Oversight & Control | ISO/IEC 42001 Annex G, EU AI Act Art. 14 |
| **D9** | MLOps & Model Lifecycle | ISO/IEC 42001 §6, NXP eIQ GenAI Flow |
| **D10** | Regulatory & Standards Compliance | EU AI Act, ISO/IEC 42001, NIST AI RMF |
| **D11** | Environmental & Sustainability | ISO 14001, GHG Protocol Scope 3 |
| **D12** | Functional Safety | IEC 61508, ISO 26262 principles |

### NXP eIQ GenAI Flow Integration Points

The NXP eIQ GenAI Flow pipeline provides built-in governance hooks that satisfy specific controls:

| Pipeline Feature | Governance Hook | Satisfies Controls |
|-----------------|-----------------|-------------------|
| On-device LLM/SLM inference | Data minimization — no logs to cloud | DG-10, DP-01, DP-02 |
| Local RAG over compressed vector DB | Knowledge base perimeter enforcement | DG-03, DG-04, LM-09 |
| Local decision logging | Audit trail and traceability | XT-01, XT-02, ML-07 |
| Hardware Root of Trust (HRoT) | Secure model loading, boot chain | SR-01, SR-02, ML-03 |
| Neutron NPU quantization pipeline | INT8/INT4 conversion governance | ML-05, ML-06 |
| Secure boot + remote attestation | Device integrity verification | SR-01, FS-04 |

---

## 4. Governance Roles

### Role Definitions

| Role | Accountability | Domain Ownership |
|------|---------------|-----------------|
| **AI Product Owner (AIPO)** | Use case approval, ethics review gate, business risk | D6, D8 |
| **Model Risk Officer (MRO)** | Model validation, fairness testing, MLOps, drift | D1, D2, D5, D9 |
| **Device Security Lead (DSL)** | Hardware security, MITRE ATLAS threat map, SBOM, OTA | D3 |
| **Privacy Officer / DPO** | GDPR compliance, DPIAs, data protection controls | D4 |
| **Functional Safety Engineer (FSE)** | SIL/ASIL assessment, fail-safe design, hazard analysis | D12 |
| **Governance Committee (GC)** | Framework maintenance, regulatory compliance, escalation | D7, D10, D11 |

### RACI Matrix — Lifecycle Stages × Roles

> **R** = Responsible (does the work) | **A** = Accountable (owns outcome) | **C** = Consulted | **I** = Informed

| Lifecycle Stage | AIPO | MRO | DSL | DPO | FSE | GC |
|-----------------|------|-----|-----|-----|-----|----|
| **L1** Conception & Design | A | C | C | C | C | R |
| **L2** Data | C | R | I | A | I | I |
| **L3** Training & Development | I | A | I | C | C | I |
| **L4** Compression & Quantization | I | A | C | I | R | C |
| **L5** Validation & Testing | C | R | A | C | A | C |
| **L6** Deployment & Fleet | A | C | R | I | C | I |
| **L7** Operation & Monitoring | C | R | A | C | C | I |
| **L8** Retirement | A | C | C | R | I | C |

---

## 5. Control Priority Guide

Each control in the domain checklists carries a priority indicator:

| Priority | Meaning | Default Action |
|----------|---------|----------------|
| 🔴 **Critical** | Non-negotiable. Absence creates material safety, legal, or security risk. | Must be implemented before any deployment |
| 🟡 **Important** | Strongly recommended. Absence indicates governance immaturity. | Implement within first operational quarter |
| 🟢 **Recommended** | Best practice. Differentiates mature programs. | Implement during continuous improvement cycles |

---

## 6. How to Use This Framework

**For a new AI project (starting at L1):**
1. Read D6 (Ethics & Acceptable Use) — determine if the use case is permissible
2. Read D10 (Regulatory Compliance) — classify the AI system under EU AI Act
3. Read D12 (Functional Safety) — determine SIL/ASIL level if applicable
4. Complete all 🔴 controls for L1 before proceeding to L2
5. Progress stage by stage, completing 🔴 controls before advancing

**For an existing deployment (entering at L7):**
1. Use the coverage matrix (`01-coverage-matrix.md`) to identify gaps
2. Prioritize all 🔴 controls not yet in place
3. Build evidence for each completed control

**For compliance evidence gathering:**
1. Each control specifies an "Evidence Required" column
2. Collect that evidence into a TRCM (Trust, Risk, and Cybersecurity Management) file
3. Reference `appendix/standards-reference-map.md` to map evidence to specific standard requirements

---

## 7. Document Structure

```
NPX-Governance/
├── 00-framework-overview.md          ← This file
├── 01-coverage-matrix.md             ← 12×8 executive coverage matrix
├── domains/
│   ├── d01-data-governance.md
│   ├── d02-llm-slm-governance.md
│   ├── d03-security-adversarial.md
│   ├── d04-privacy-data-protection.md
│   ├── d05-fairness-bias.md
│   ├── d06-ethics-acceptable-use.md
│   ├── d07-explainability-transparency.md
│   ├── d08-human-oversight-control.md
│   ├── d09-mlops-model-lifecycle.md
│   ├── d10-regulatory-compliance.md
│   ├── d11-environmental-sustainability.md
│   └── d12-functional-safety.md
└── appendix/
    └── standards-reference-map.md
```

---

*Version 1.0 — 2026-05-29 | Maintained by Governance Committee*
