# D8 — Human Oversight & Control

> **Domain purpose:** Govern the design, implementation, and operation of human oversight mechanisms for NXP edge AI systems — defining autonomy levels, mandatory intervention points, kill-switch requirements, remote override capabilities, and escalation procedures. Ensures that humans retain meaningful control over AI systems that operate in physical environments.
>
> **Primary standards:** ISO/IEC 42001 Annex G (human oversight), EU AI Act Art. 14 (human oversight for high-risk AI), IEEE 7001 §6 (human control)
>
> **RACI owner:** AI Product Owner (AIPO) — primary; Device Security Lead (DSL) — secondary for kill-switch and remote intervention

---

## NXP-Specific Context

NXP edge devices frequently operate in unmanned or semi-autonomous environments where the traditional assumption of a human operator watching a screen does not hold. This creates governance challenges:

1. **Autonomy levels vary widely across NXP deployments.** A voice assistant on an HMI panel (human always present) has a very different oversight profile from a mobile robot (operating autonomously in a warehouse) or a medical monitoring device (acting on behalf of patients). The governance framework must accommodate this range.

2. **Physical intervention may be delayed or impossible.** For robots or vehicles, a software kill-switch must be complemented by a physical emergency stop (E-stop) that does not depend on the AI software. Governance must bridge the software and physical safety layers.

3. **Remote intervention must be authenticated.** In a fleet context, the ability to remotely disable or override an AI system is also an attack vector. Remote intervention commands must be cryptographically authenticated using the device's hardware root of trust.

---

## Control Checklist

### Sub-domain 8.1: Autonomy Level Classification (L1)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| HO-01 | **Autonomy level classification:** Classify the AI system on the following autonomy scale and document the classification with rationale: **Level 1 — Advisory:** AI outputs are always reviewed by a human before any action is taken. **Level 2 — Human-on-the-loop:** AI acts autonomously but a human monitors and can override. **Level 3 — Human-in-exception:** AI acts autonomously; humans are only involved when AI flags uncertainty or an exception. **Level 4 — Supervised autonomy:** AI acts fully autonomously within a defined operational domain; oversight is periodic, not continuous. The classification must be approved by AIPO and reviewed by FSE for any level ≥ 2. | 🔴 | L1 | Autonomy level classification document; rationale; AIPO approval; FSE review for level ≥ 2 |
| HO-02 | **Mandatory human intervention points:** For each AI function, define the specific triggers that require mandatory human intervention regardless of autonomy level: confidence below threshold, out-of-distribution input detected, safety interlock condition met, or any decision in the prohibited autonomous decisions registry (D12 FS-12). Implement these as non-bypassable system states. | 🔴 | L1 | Mandatory intervention points specification; non-bypassability implementation evidence |

### Sub-domain 8.2: Kill-Switch & Override Design (L5/L6)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| HO-03 | **Software kill-switch:** Implement a software kill-switch that: (a) immediately halts all AI-driven autonomous actions; (b) transitions the device to a defined safe state (human-operated mode or controlled shutdown); (c) is accessible to authorized operators without requiring AI system cooperation; (d) logs the kill-switch activation event. The kill-switch must not be bypassable by the AI system. | 🔴 | L5 | Kill-switch implementation; safe state specification; non-bypassability test; log capture evidence |
| HO-04 | **Override mechanism:** Implement an override mechanism that allows authorized humans to: (a) reject individual AI recommendations; (b) modify AI-suggested parameters before execution; (c) substitute a manual decision for an AI decision. Each override must be logged with the overriding operator's identity, the original AI decision, and the override value. | 🔴 | L5 | Override mechanism implementation; logging evidence; test cases |

### Sub-domain 8.3: Deployment — Operator Training & Interface (L6)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| HO-05 | **Kill-switch operational test:** Before fleet deployment, test the kill-switch in the actual device configuration. Measure time from kill-switch activation to safe state achievement. The measured time must be within the fault-tolerant time interval defined in the functional safety analysis (D12 FS-04). | 🔴 | L6 | Kill-switch operational test results; measured activation-to-safe-state latency |
| HO-06 | **Operator training for AI oversight:** Define minimum competency requirements for operators who oversee AI Level ≥ 2 systems. Include training on: recognizing AI errors and uncertainty signals, executing the override mechanism, activating the kill-switch, and escalating anomalies. Verify training completion before operators are authorized. | 🔴 | L6 | Operator training curriculum; completion records; authorization records |

### Sub-domain 8.4: Operations (L7)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| HO-07 | **Override rate monitoring:** Track the frequency of human overrides of AI decisions. A high override rate indicates the AI is not performing adequately for its autonomy level and may require model revision or autonomy level downgrade. Define a threshold override rate that triggers a formal review. | 🟡 | L7 | Override rate monitoring dashboard; threshold definition; review trigger documentation |
| HO-08 | **Remote intervention authentication:** Implement cryptographically authenticated remote intervention commands (kill-switch, model rollback, operational mode change). Remote commands must be signed with a key managed by the DSL and verified by the device's hardware root of trust. Unauthorized remote commands must be rejected and logged. | 🔴 | L7 | Remote intervention authentication architecture; key management procedure; rejection test results |
| HO-09 | **Automation bias monitoring:** Monitor for evidence of automation bias — operators consistently accepting AI recommendations without meaningful review. Implement periodic calibration exercises where AI recommendations are intentionally incorrect to assess whether operators catch errors. | 🟡 | L7 | Automation bias assessment methodology; calibration exercise results |

### Sub-domain 8.5: Retirement (L8)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| HO-10 | **Oversight capability preservation on retirement:** When a device is decommissioned, ensure the override and kill-switch records are archived with the audit log. If the device is being replaced by a higher-autonomy successor, document the governance review that approved the autonomy level increase. | 🟡 | L8 | Archive records; autonomy level review documentation |

---

## Autonomy Level vs. Required Controls

| Autonomy Level | Required Controls | RACI Escalation |
|---------------|------------------|-----------------|
| Level 1 — Advisory | HO-01, HO-02, HO-04 (override logging) | AIPO approval |
| Level 2 — Human-on-the-loop | All Level 1 + HO-03, HO-05, HO-06 | AIPO + FSE review |
| Level 3 — Human-in-exception | All Level 2 + HO-07, HO-08, HO-09 | AIPO + FSE + GC review |
| Level 4 — Supervised autonomy | All Level 3 + mandatory D12 safety review | Full Governance Committee approval |

---

## Standards Cross-Reference

| Control(s) | ISO/IEC 42001 Annex G | EU AI Act | IEEE 7001 | IEC 61508 |
|------------|----------------------|-----------|-----------|-----------|
| HO-01, HO-02 | §G.1 (human oversight design) | Art. 14(1) | §6.1 | §7.9 |
| HO-03, HO-04 | §G.2 (intervention capability) | Art. 14(4) | §6.3 | §7.11 |
| HO-05, HO-06 | §G.3 (operator training) | Art. 14(3) | §6.2 | §8.2 |
| HO-07, HO-08 | §G.4 (monitoring) | Art. 14(5) | §6.4 | — |
| HO-09 | §G.5 (automation bias) | Art. 14(3)(d) | §6.5 | — |
| HO-10 | §G.1 | Art. 18 | — | — |

---

*Domain D8 | Version 1.0 — 2026-05-29*
