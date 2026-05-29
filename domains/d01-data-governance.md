# D1 — Data Governance

> **Domain purpose:** Govern the quality, provenance, lifecycle, and integrity of all data assets used in NXP edge AI systems — including training datasets, fine-tuning datasets, RAG knowledge base documents, on-device sensor data, and inference logs. Ensures that data is trustworthy, traceable, and managed with appropriate retention and access controls throughout the AI lifecycle.
>
> **Primary standards:** ISO/IEC 25012 (Data quality), DAMA-DMBOK (Data Management Body of Knowledge), ISO/IEC 42001 §8.4
>
> **RACI owner:** Model Risk Officer (MRO) — primary; Privacy Officer (DPO) — secondary for personal data

---

## NXP-Specific Context

NXP edge AI deployments present data governance challenges that do not exist in centralized cloud architectures:

1. **RAG knowledge base as the primary information source.** On-device LLMs have no internet access. The local vector DB is the sole source of factual grounding. Document quality, staleness, and provenance directly determine the model's reliability and compliance posture. Governance of the knowledge base is as critical as governance of the model itself.

2. **Sensor data as training input.** Many NXP edge AI systems are trained on sensor streams (cameras, microphones, accelerometers, CAN bus data). The provenance and representativeness of these streams — which physical environments, which hardware revisions, which operating conditions — determines model generalization.

3. **On-device data minimization imperative.** The governance value proposition of on-device inference is that raw data never leaves the device. Governance controls must enforce this — ensuring that neither the AI pipeline nor any monitoring/telemetry mechanism inadvertently exfiltrates raw sensitive data.

4. **No central data lake.** Data is distributed across device fleets. Lineage must be tracked without a central repository, requiring device-local metadata and periodic aggregation.

---

## Control Checklist

### Sub-domain 1.1: Data Source Registry & Provenance (L1)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| DG-01 | **Data source registry:** Maintain a registry of all data sources used across all AI projects. Each entry must record: source name, type (public dataset / proprietary / synthetic / sensor stream), provider/owner, license or data agreement reference, data classification (personal / sensitive / operational / public), and last review date. | 🔴 | L1 | Data source registry document; review dates current |

### Sub-domain 1.2: Training & Fine-Tuning Data (L2)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| DG-02 | **Dataset documentation (Datasheets for Datasets):** For every dataset used in training or fine-tuning, produce a datasheet covering: collection methodology, size, date range, geographic/demographic coverage, known limitations, label methodology, and IP/licensing status. | 🔴 | L2 | Completed datasheet per training dataset |
| DG-03 | **Data quality assessment:** Define and measure data quality dimensions for each training dataset: completeness (missing values), accuracy (label correctness), consistency (schema conformance), timeliness (currency of data), and representativeness (coverage of target deployment conditions). Document acceptance thresholds and measured values. | 🔴 | L2 | Data quality report per dataset with measurements vs. thresholds |
| DG-04 | **Data lineage:** Maintain full traceability from raw source data through all preprocessing, augmentation, and feature engineering steps to the final training-ready dataset. Use version-controlled pipeline definitions (e.g., DVC, MLflow artifacts). | 🔴 | L2 | Data lineage diagram or tool output; pipeline version history |
| DG-05 | **Dataset versioning:** Every training dataset version must be immutably versioned and stored. No dataset version used in a model training run may be overwritten or deleted while the associated model remains in production. | 🔴 | L2 | Dataset version control policy; immutable storage confirmation |

### Sub-domain 1.3: Inference-Time Data (L3)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| DG-06 | **Inference data classification:** Classify all data processed during inference by sensitivity level: (a) public/operational — no restrictions; (b) sensitive — must not be logged in identifiable form; (c) personal — subject to GDPR controls (see D4); (d) regulated — subject to sector-specific rules (PHI, financial transaction data). Define handling rules per class. | 🔴 | L3 | Data classification policy with handling rules per class |
| DG-07 | **Training data poisoning prevention:** Implement integrity controls on the training data pipeline to prevent unauthorized modification of training datasets. Include: access controls on training data storage, hash verification of datasets before training runs, and anomaly detection for unexpected data additions. | 🟡 | L3 | Access control policy; hash verification procedure; anomaly detection configuration |

### Sub-domain 1.4: Validation Data (L5)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| DG-08 | **Test set independence:** Validation and test sets must be strictly held out from training and fine-tuning. Document the split methodology, date of split, and confirm no data leakage between sets. For sensor data, splits must respect temporal and device boundaries (no same-device, same-session data in both train and test). | 🔴 | L5 | Test set split documentation; leakage analysis results |

### Sub-domain 1.5: RAG Knowledge Base Governance (L6/L7)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| DG-09 | **Knowledge base curation policy:** Define and enforce a curation policy for the local RAG vector DB. Specify: (a) document eligibility criteria (only approved, verified sources may be ingested); (b) approval process for new documents (who reviews, what criteria); (c) metadata requirements per document (source, version, date, author, expiry date if applicable); (d) prohibited content categories. | 🔴 | L6 | Knowledge base curation policy; approval workflow documentation |
| DG-10 | **Knowledge base versioning and integrity:** The RAG vector DB must be versioned. Each deployment of a new knowledge base version must be authorized by the MRO. Integrity of the vector DB must be verified at device startup (hash check). Unauthorized modifications must trigger an alert. | 🔴 | L7 | Version history of knowledge base; integrity check implementation; alert configuration |
| DG-11 | **Document staleness management:** Define maximum document age or freshness requirements per document category (e.g., safety manuals max 12 months; product specs max 6 months). Implement an automated staleness alert when documents approach their expiry. Retired documents must be removed from the knowledge base before they expire. | 🟡 | L7 | Staleness policy per document category; expiry tracking implementation; review records |

### Sub-domain 1.6: On-Device Data Retention (L8)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| DG-12 | **On-device data retention policy:** Define retention periods for all data stored on-device: inference logs, sensor cache, RAG query logs, decision audit trails. Implement automated purge after retention period. For personal data, retention must comply with GDPR Article 5(1)(e). Purge procedures must survive device reboot and OTA updates. | 🔴 | L8 | Retention policy per data class; purge implementation; compliance with D4 (privacy) requirements |

---

## Standards Cross-Reference

| Control(s) | ISO/IEC 25012 | DAMA-DMBOK | ISO/IEC 42001 | EU AI Act | GDPR |
|------------|--------------|-----------|--------------|-----------|------|
| DG-01 | §6 (Data provenance) | Ch. 9 (Data lineage) | §8.4 | Art. 10 (training data) | — |
| DG-02, DG-03 | §7 (Quality dimensions) | Ch. 13 (Data quality) | §8.4 | Art. 10(3) | — |
| DG-04, DG-05 | §6 (Traceability) | Ch. 9 (Lineage) | §8.4 | Art. 10, Annex IV | — |
| DG-06, DG-07 | — | Ch. 7 (Security) | §8.5 | Art. 10(5) | Art. 5 |
| DG-08 | §7.2 (Accuracy) | Ch. 13 | §8.5 | Art. 9, Art. 10 | — |
| DG-09, DG-10, DG-11 | §7.5 (Currentness) | Ch. 13 | §8.5 | Art. 10 | — |
| DG-12 | — | Ch. 2 (Data lifecycle) | §8.3 | Art. 18 | Art. 5(1)(e), Art. 17 |

---

*Domain D1 | Version 1.0 — 2026-05-29*
