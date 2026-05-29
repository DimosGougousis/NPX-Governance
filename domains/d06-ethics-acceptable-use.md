# D6 — Ethics & Acceptable Use

> **Domain purpose:** Govern the ethical boundaries of NXP edge AI deployments — defining what use cases are permissible, establishing an ethics review gate before development begins, maintaining a prohibited use case registry, and ensuring that AI autonomy levels are proportionate to validated capability and societal acceptability.
>
> **Primary standards:** EU AI Act Art. 5 (prohibited AI practices), OECD AI Principles (2023 revision), IEEE 7000 (Ethical concerns in system design)
>
> **RACI owner:** AI Product Owner (AIPO) — primary; Governance Committee (GC) — secondary

---

## NXP-Specific Context

NXP edge devices are deployed in physically proximate, often intimate contexts — operating rooms, vehicle cabins, factory floors, public spaces. This creates ethical risks that differ from cloud AI:

1. **Physical world consequences.** Unlike a web recommendation, an edge AI decision can directly actuate physical systems. The ethical analysis must address not just discriminatory or privacy-violating outputs but also physical harm potential.

2. **Autonomous operation without human oversight.** Edge devices frequently operate without a human operator present. The acceptable use framework must define which decisions an AI system may make autonomously and which require human oversight — independent of the human oversight controls in D8.

3. **Dual-use risk.** The same NXP hardware and eIQ GenAI Flow pipeline that powers a legitimate medical assistant or industrial robot can be repurposed for surveillance, autonomous weapons, or social control applications. Export control and dual-use assessment are essential.

---

## Control Checklist

### Sub-domain 6.1: Ethics Review Gate (L1)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| EU-01 | **Ethics review gate:** Before any AI project proceeds to data collection or development (L2), the AI Product Owner must submit the proposed use case for an ethics review. The review assesses: (a) whether the use case falls within the prohibited uses registry; (b) whether the use case raises ethical concerns not covered by the registry; (c) whether the proposed autonomy level is proportionate and appropriate. The review must produce a written disposition (approved / approved with conditions / rejected). | 🔴 | L1 | Ethics review submission form; written disposition from review body (AIPO + GC) |
| EU-02 | **EU AI Act risk classification:** Classify the AI system under the EU AI Act risk hierarchy: (a) prohibited (Art. 5); (b) high-risk (Art. 6, Annex III); (c) limited risk (Art. 50); (d) minimal risk. Document the classification rationale. For high-risk systems, initiate compliance activities per Art. 9-15. For prohibited systems, halt development immediately. | 🔴 | L1 | EU AI Act risk classification document with rationale; compliance activity plan for high-risk |
| EU-03 | **Dual-use risk assessment:** Assess whether the AI system or its components could be repurposed for dual-use applications (surveillance, autonomous weapons, social scoring, mass control). Document the assessment and any technical or contractual mitigations applied (use restriction in license, design features preventing dual-use repurposing). | 🔴 | L1 | Dual-use risk assessment; mitigation documentation |

### Sub-domain 6.2: Training Data Ethics (L2)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| EU-04 | **Consent and data ethics in training data:** For training data collected from human subjects, verify: (a) informed consent was obtained for the specific purpose of AI training; (b) data collection methods were ethical and transparent; (c) participants could withdraw consent; (d) vulnerable populations (children, patients, employees) were not subjected to coercive data collection. | 🔴 | L2 | Consent records or waiver documentation; ethical collection methodology |

### Sub-domain 6.3: Model Development Ethics (L3)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| EU-05 | **Environmental proportionality:** Assess whether the computational cost of training the model is proportionate to the benefit. For large foundation model fine-tuning runs, estimate the carbon footprint (reference D11 ES-02). If the footprint is significant, document justification and alternatives considered (smaller model, fewer training epochs, existing open-source model as base). | 🟢 | L3 | Proportionality assessment; carbon footprint estimate; alternatives considered |

### Sub-domain 6.4: Validation (L5)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| EU-06 | **Vulnerable population impact assessment:** Before deploying any AI system that interacts with or makes decisions about individuals, assess whether it could adversely affect vulnerable populations: children, elderly, cognitively impaired individuals, people in crisis, economically disadvantaged users. Define mitigations for identified risks. | 🔴 | L5 | Vulnerable population impact assessment; mitigations per identified risk |
| EU-07 | **Autonomy proportionality review:** Review whether the level of autonomy granted to the AI system (per D8 HO-01) is proportionate to: (a) its validated performance characteristics; (b) the severity of consequences of errors; (c) societal expectations in the deployment context. Escalate to the Governance Committee for any system operating at high autonomy in safety-relevant contexts. | 🔴 | L5 | Autonomy proportionality review document; GC sign-off for high-autonomy systems |

### Sub-domain 6.5: Operations (L7)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| EU-08 | **Prohibited use monitoring:** Implement monitoring to detect if a deployed AI system is being used for prohibited or unapproved purposes (e.g., a vision AI deployed for quality inspection being repurposed for employee surveillance). Define alert triggers and response procedures including immediate system suspension. | 🟡 | L7 | Use monitoring configuration; alert definition; response procedure |
| EU-09 | **Ethics incident response:** Define a procedure for responding to ethics incidents: AI system causing unanticipated harm, use of AI system for prohibited purposes discovered, bias or discrimination identified. Include escalation to AIPO, GC, and legal counsel. For serious incidents, include regulatory notification procedure. | 🔴 | L7 | Ethics incident procedure; escalation path documentation |

### Sub-domain 6.6: Prohibited Uses Registry (Permanent)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| EU-10 | **Prohibited uses registry maintenance:** Maintain a board-approved registry of AI use cases that are categorically prohibited within the organization. The registry must include at minimum all uses prohibited by EU AI Act Art. 5. Review and update the registry annually and upon new regulatory guidance. | 🔴 | All | Approved registry; annual review records |
| EU-11 | **Registry communication and training:** Ensure all teams involved in AI development are aware of the prohibited uses registry. Include it in onboarding materials and annual AI literacy training. Document that developers have acknowledged the registry. | 🟡 | L8 | Training records; developer acknowledgment records |

---

## EU AI Act Art. 5 Prohibited Practices — Edge AI Relevance

| Prohibition | Art. 5 Reference | Edge AI Risk |
|-------------|-----------------|--------------|
| Subliminal manipulation beyond conscious awareness | Art. 5(1)(a) | HMI AI influencing operator decisions via subliminal cues |
| Exploiting vulnerabilities of specific groups | Art. 5(1)(b) | AI targeting elderly or cognitively impaired users |
| Biometric categorization for sensitive inferences | Art. 5(1)(c) | On-device vision AI inferring political views, sexual orientation |
| Real-time remote biometric ID in public spaces | Art. 5(1)(h) | NXP camera AI identifying individuals in public — generally prohibited |
| Social scoring by public authorities | Art. 5(1)(e) | Not typically applicable to edge devices |
| Predictive policing on personal characteristics | Art. 5(1)(f) | Security camera AI flagging individuals based on inferred characteristics |
| Facial recognition database scraping | Art. 5(1)(g) | Vision AI building unauthorized face databases from device streams |

---

## Standards Cross-Reference

| Control(s) | EU AI Act | OECD AI Principles | IEEE 7000 | ISO/IEC 42001 |
|------------|-----------|-------------------|-----------|--------------|
| EU-01, EU-02 | Art. 5, Art. 6, Annex III | Principle 1.1 (inclusive growth) | §7.3 | §6.1 (risk assessment) |
| EU-03 | Art. 5 (implicitly) | Principle 1.4 (security) | §7.4 | §8.4 |
| EU-04 | Art. 10 | Principle 1.2 (human-centred) | §7.2 | §8.4 |
| EU-05 | Recital 27 | Principle 1.5 (environment) | — | §8.5 |
| EU-06, EU-07 | Art. 9, Art. 14 | Principle 1.3 (transparency) | §7.5 | §8.5 |
| EU-08, EU-09 | Art. 72 | Principle 2.4 (accountability) | §8.2 | §9.3 |
| EU-10, EU-11 | Art. 5 | All principles | §6.2 | §5.2 |

---

*Domain D6 | Version 1.0 — 2026-05-29*
