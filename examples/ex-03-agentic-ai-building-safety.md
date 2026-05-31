# Example 3 — eIQ Agentic AI Framework: "BuildingSafe" Autonomous Safety Controller

> **NXP Product:** eIQ Agentic AI Framework | **Hardware:** i.MX 95 + Kinara NPU (discrete)
> **Stack:** 3 coordinated agents — VisionAgent (occupancy/hazard detection) + VoiceAgent (NLP on intercom audio) + SafetyOrchestrator (decision model)
> **Use Case:** Autonomous building safety controller. Deployed in hospitals, factories, commercial buildings. Triggers building-wide evacuation, fire suppression, and door lockdown **without per-event human approval**.
> **Governance Type:** Cross-vertical horizontal framework | **Autonomy Level:** Level 3 (Human-in-exception)

---

## Why This Scenario Needs Governance

The BuildingSafe system can evacuate a hospital. Without governance, the question is not *if* a false positive will occur — it's *when*, and what the consequence will be. A false evacuation of a hospital ICU at 2am means moving ventilated patients, crashing monitors, and potential deaths. The governance program for this system is not administrative overhead: it is the technical and organizational structure that determines whether the system is trustworthy enough to have Level 3 autonomy at all.

This is the hardest governance scenario in the NXP portfolio. Every decision made carelessly at L1 compounds through L2–L8 into potential catastrophe.

---

## L1 — Conception & Design

### WHY
A system that can trigger building-wide evacuation without a human approving each event is fundamentally different from an advisory AI. Level 3 autonomy — where AI acts automatically and humans only intervene on exceptions — requires Governance Committee approval, a formal SIL determination, and a prohibited decisions registry that defines exactly which actions the AI can never take autonomously, regardless of confidence. If L1 is weak, no amount of testing at L5 can compensate.

### WHAT (Controls Triggered)
`HO-01` Autonomy Level 3 classification + GC approval | `HO-02` Mandatory intervention points | `FS-01` Hazard analysis (HARA) | `FS-02` SIL-2 determination | `FS-12` Prohibited autonomous decisions registry | `EU-01/02` Ethics review + EU AI Act classification | `DP-01` DPIA trigger assessment | `RC-01/02` Regulatory scope

### HOW
1. **Full Governance Committee meeting** (AIPO + MRO + DSL + FSE + DPO + external safety consultant, minimum 3 hours). This is not a desk review — this is a formal governance gate.
2. **Classify autonomy level**: The SafetyOrchestrator triggers evacuation without per-event human approval. → **Level 3 (Human-in-exception)**. This requires GC approval — document the vote.
3. **Run HARA for all 3 agents**: Separate hazard analysis per agent (VisionAgent, VoiceAgent, SafetyOrchestrator) AND for the multi-agent system as a whole. What happens when agents disagree? What happens when one agent fails?
4. **Determine SIL**: Evacuation trigger in a hospital context → S3 severity, E4 exposure (continuous operation), C1 controllability (occupants cannot easily stop a triggered evacuation) → **SIL 2**.
5. **Build prohibited decisions registry**: Define what the system absolutely cannot do autonomously — see template below.
6. **DPIA**: VisionAgent processes camera feeds of building occupants. These images contain faces → **biometric data under GDPR Art. 9** → DPIA is **mandatory** before data collection.
7. **EU AI Act classification**: Building safety controller — check Annex III point 1 (critical infrastructure). May qualify as **high-risk**. → Initiate Annex IV documentation.

### TOOLS
- **HARA worksheet**: Multi-agent version (template below)
- **GC approval memo**: Formal decision record with named signatories
- **SIL calculation form**: Per IEC 61508 / ISO/IEC TR 5469
- **Prohibited decisions registry**: Version-controlled Markdown

### TEMPLATE — Multi-Agent HARA (Selected Rows)

| # | Agent | Hazardous Event | Failure Mode | Severity | Exposure | Controllability | SIL | Mitigation |
|---|-------|----------------|--------------|----------|----------|----------------|-----|-----------|
| H1 | SafetyOrchestrator | Hospital ICU evacuated during surgery | False positive evacuation trigger (high confidence hallucination) | S3 (potential patient death) | E4 (continuous operation) | C1 (ICU staff cannot quickly stop cascade) | SIL 2 | Dual-model confirmation before evacuation; 90-second delay + human interrupt window |
| H2 | VisionAgent | Fire not detected — no evacuation triggered | Smoke obscures camera; model outside ODD | S3 | E2 | C2 (fire alarm system parallel) | SIL 1 | Parallel smoke detector integration; ODD monitoring; human alert on low-confidence |
| H3 | VoiceAgent | False "FIRE" interpreted from background noise | Audio adversarial — HVAC noise triggers keyword | S2 | E3 | C2 | SIL 1 | Minimum 3-second sustained detection; cross-validation with VisionAgent |
| H4 | Multi-agent | Agents disagree — VisionAgent says fire, VisionAgent says no fire | Agent coordination failure | S2 | E2 | C2 | SIL 1 | Explicit tie-breaking rule: disagreement → alert human operator, do not auto-trigger |
| H5 | SafetyOrchestrator | Lockdown traps occupants with threat | Wrong threat classification → wrong response | S3 | E2 | C0 (locked doors cannot be opened from inside) | SIL 2 | Lockdown requires dual confirmation; 120-second escape window before full lock |

### TEMPLATE — Prohibited Autonomous Decisions Registry

```markdown
## Prohibited Autonomous Decisions Registry — BuildingSafe v1.0
Approved by: Governance Committee | Date: 2026-06-01 | Next review: 2027-06-01

The following decisions MUST NOT be taken autonomously by any agent without human override confirmation:

| # | Decision | Rationale | Require Human Confirm? | Timeout Behavior |
|---|---------|-----------|----------------------|-----------------|
| PAD-01 | Full building evacuation in a hospital during active surgery | Irreversible patient harm risk from surgery interruption | YES — 90-second interrupt window | If no human response in 90s: EVACUATE (patient safety > surgery continuity) |
| PAD-02 | Complete building lockdown (all doors locked, no exit) | Trapping occupants with an active threat is worse than evacuation | YES — 120-second escape window first | Auto-lock only after escape window |
| PAD-03 | Fire suppression system activation in server/data rooms | Suppression agents (CO2/FM-200) are lethal to occupants present | YES — facility manager confirmation | If facility manager unreachable: activate suppression + issue max evacuation alert |
| PAD-04 | Cancellation of an already-triggered evacuation | False negatives during evacuation are fatal | YES — two authorized persons must confirm | System cannot un-ring the alarm autonomously |
| PAD-05 | Remote door unlock for emergency services | Authentication risk if triggered by spoofed signal | YES — authenticated emergency services request | No autonomous unlock without verified auth |
```

### TEMPLATE — GC Approval Memo

```markdown
## Governance Committee Approval Memo — BuildingSafe v1.0
Meeting date: 2026-06-02 | Quorum: 6/6 members present

### Vote: Approve Level 3 Autonomy for SafetyOrchestrator
Motion: Grant Level 3 (Human-in-exception) autonomy to the SafetyOrchestrator agent
for the actions defined in the Autonomy Scope below, subject to conditions.

Autonomy scope APPROVED:
- Fire alert activation ✓
- Partial evacuation zone activation ✓
- Emergency service notification ✓

Requiring human confirmation (per Prohibited Decisions Registry):
- Full building evacuation (PAD-01) — 90-second interrupt window
- Full lockdown (PAD-02) — 120-second escape window
- Suppression activation in occupied areas (PAD-03)

Vote: 6 APPROVE / 0 REJECT / 0 ABSTAIN ✓

Conditions:
1. SIL-2 validation must be independently assessed (FS-09)
2. Pilot deployment limited to non-clinical building zones for first 90 days
3. Override rate >5%/week triggers mandatory review

Signed: [AIPO, MRO, DSL, DPO, FSE, External Safety Consultant]
```

---

## L2 — Data

### WHY
VisionAgent is trained on camera footage of building interiors. Those frames contain faces of real people going about their day. Under GDPR Article 9, facial images used to train an AI are **biometric data** — a special category requiring explicit legal basis. You cannot simply scrape CCTV footage and train on it. Additionally, if the training data only covers one building type (office buildings) and BuildingSafe is deployed in a hospital, the model's representation of hospital-specific scenarios (bed-blocked corridors, patients in gowns, medical equipment) will be systematically absent — and the bias test at L3 will catch it, but only if the training data analysis was done rigorously at L2.

### WHAT (Controls Triggered)
`DP-01` DPIA (mandatory — biometric data) | `DP-03` Data minimization | `DG-02` Dataset documentation per agent | `FB-02` Representativeness analysis | `FB-03` Sensor hardware diversity | `DG-04` Data lineage

### HOW
1. **Complete DPIA before any data collection**: The DPIA must document legal basis for processing biometric data (likely legitimate interest for safety purposes — document the balancing test).
2. **Apply data minimization immediately at capture**: Do not store identifiable frames. Convert to anonymized feature vectors (skeleton poses, occupancy maps, heat signatures) before training. Store anonymized data only.
3. **Build per-agent dataset documentation**: Three separate datasheets — one for VisionAgent, one for VoiceAgent, one for SafetyOrchestrator (which trains on event logs, not raw sensor data).
4. **Run representativeness analysis**: Does the training data cover hospitals (gurneys, IV poles, patients in beds, narrow clinical corridors)? Office buildings? Factories? Document coverage and gaps.

### TOOLS
- **DPIA template**: From `domains/d04-privacy-data-protection.md` DP-01
- **Anonymization pipeline**: OpenCV-based face blurring + pose estimation before frame storage
- **Dataset representativeness matrix**: Table template (below)

### TEMPLATE — 3-Agent Dataset Documentation Summary

```markdown
## Dataset Documentation — BuildingSafe v1.0 Agents

### VisionAgent Training Data
Dataset: building-vision-v1.0 (anonymized)
Size: 2.4M frames, 180 building-hours
Anonymization: All faces blurred with Gaussian blur before storage; only pose + occupancy heatmaps retained
Legal basis for biometric processing: Legitimate interest (safety) — balancing test completed 2026-05-15

### Representativeness Matrix
| Building Type | Frames | Occupancy Range | Covered Scenarios | Gaps |
|--------------|--------|----------------|------------------|------|
| Office (open plan) | 900K | 0-200 persons | Normal, fire drill, evacuation | Lockdown scenario underrepresented |
| Hospital (general ward) | 600K | 0-80 persons | Normal, code blue | ICU equipment, night shift gaps |
| Hospital (ICU) | 120K | 0-20 persons | Normal only | Emergency scenarios absent — MITIGATION: synthetic augmentation planned |
| Factory floor | 480K | 0-150 persons | Normal, fire drill | Smoke/fire scenarios underrepresented |
| Commercial (retail) | 300K | 0-500 persons | Normal, crowd | Emergency scenarios absent |

Representativeness gaps requiring mitigation before deployment in ICU contexts:
- ICU emergency scenarios: Supplement with synthetic data (3D rendered scenarios)
- Night shift: Collect additional low-light footage

### VoiceAgent Training Data
Dataset: intercom-audio-v1.0 (anonymized)
Size: 400 hours
Anonymization: Voice pitch shifted ±20% to prevent speaker identification
Coverage: English (US, UK, AU), Emergency keywords: FIRE, HELP, EVACUATION, GAS LEAK, ARMED PERSON
Gaps: Non-English languages — deployment restricted to English-language facilities in v1.0

### SafetyOrchestrator Training Data
Dataset: safety-event-logs-v1.0 (synthetic + historical anonymized)
Size: 50,000 simulated scenarios
Source: Fault injection simulation + historical anonymized building safety logs
Note: No personal data in orchestrator training data ✓
```

---

## L3 — Training & Development

### WHY
The SafetyOrchestrator is a multi-input decision model: it takes structured outputs from VisionAgent (occupancy map, threat probability, confidence) and VoiceAgent (keyword, confidence, duration) and produces an action decision. Its training must define precise decision boundaries — at what combination of VisionAgent + VoiceAgent confidence does it trigger an alert vs. an evacuation? What is the tie-breaking rule when agents contradict? These constraints are governance artifacts, not just hyperparameters. An untrained model discovering these boundaries from data alone, without explicit governance constraints, is a safety failure waiting to happen.

### WHAT (Controls Triggered)
`ML-03/04` Reproducible training + model card (per agent) | `FS-04` Fail-safe behavior specification | `LM-03` SafetyOrchestrator system prompt/constraint policy | `DG-07` Training data poisoning prevention

### HOW
1. **Write the SafetyOrchestrator constraint policy**: Document the decision thresholds, tie-breaking rules, and prohibited action combinations (see template).
2. **Define per-agent fail-safe behaviors**: If VisionAgent crashes, what does SafetyOrchestrator do? → Default to alert (fail-safe direction is "assume danger"). Document this explicitly.
3. **Log all training runs** in `governance/training-runs/` — one YAML per agent per run.
4. **Draft model cards for all 3 agents**: Cover intended use, out-of-scope uses, known limitations.

### TOOLS
- **MLflow**: Per-agent run tracking
- **Constraint policy**: Versioned Markdown in `governance/orchestrator-constraints-v1.0.md`
- **Model card template**: From `domains/d07-explainability-transparency.md` XT-06

### TEMPLATE — SafetyOrchestrator Constraint Policy

```markdown
## SafetyOrchestrator Decision Constraint Policy — BuildingSafe v1.0
Version: 1.0 | Approved by: FSE + MRO | Date: 2026-06-08

### Decision Thresholds
| Action | Trigger Condition | Minimum Duration | Override Window |
|--------|-----------------|-----------------|----------------|
| Alert (audible) | VisionAgent P(hazard) ≥ 0.6 OR VoiceAgent keyword confidence ≥ 0.8 | 3 seconds sustained | Operator can cancel within 30s |
| Zone evacuation | VisionAgent P(fire) ≥ 0.85 AND VoiceAgent keyword ≥ 0.7 | 5 seconds sustained | Operator interrupt: 60s |
| Full evacuation | P(fire) ≥ 0.95 AND VoiceAgent ≥ 0.85 AND duration ≥ 8s | 8 seconds sustained | 90s interrupt window (PAD-01) |
| Emergency services notification | Zone evacuation triggered | Immediate | Manual cancel within 30s |

### Agent Disagreement Rules
- VisionAgent says FIRE, VoiceAgent says NO HAZARD: → Alert mode + 60s interrupt window (human must confirm)
- VisionAgent says NO HAZARD, VoiceAgent says FIRE: → Alert mode + 60s interrupt window
- Both agents LOW CONFIDENCE (<0.6): → Alert human operator; no autonomous action
- One agent OFFLINE: → Alert human operator immediately; do not auto-trigger critical actions

### Fail-Safe Behavior (per FS-04)
VisionAgent offline: → SafetyOrchestrator enters "audio-only mode"; evacuation threshold doubled; human alert mandatory
VoiceAgent offline: → "visual-only mode"; evacuation threshold increased; human alert mandatory
SafetyOrchestrator offline: → Both sub-agents continue operating in standalone alert mode; no evacuation without human
All agents offline: → Fail-safe state: building-wide alert tone + "Manual emergency protocols in effect"
Network to facility management system down: → Continue operating; queue alerts for when connectivity restores

### Prohibited Action Combinations
- SafetyOrchestrator CANNOT simultaneously trigger evacuation AND lockdown (contradiction)
- SafetyOrchestrator CANNOT cancel an active fire suppression once triggered
- SafetyOrchestrator CANNOT lower a triggered alert below "Alert" level without human confirmation
```

---

## L4 — Compression & Quantization

### WHY
BuildingSafe operates at SIL-2. For a SIL-2 AI function, quantization is not a performance optimization — it is a safety-critical transformation that requires formal validation. When VisionAgent is quantized from FP32 to INT8 for the Kinara NPU, does it still reliably detect fire at P≥0.85? Does the SafetyOrchestrator still respect its decision thresholds after quantization? A 0.01 shift in output probability that is invisible in aggregate accuracy testing could push the orchestrator across a critical decision boundary.

### WHAT (Controls Triggered)
`ML-05/06` Quantization accuracy regression + safety property preservation | `FS-05` SIL-2 safety quantization validation | `FS-06` Non-determinism characterization | `ML-07` Quantization config documentation

### HOW
1. **Per-agent quantization**: Each of the 3 agents is quantized separately (VisionAgent and VoiceAgent for Kinara NPU; SafetyOrchestrator may run on i.MX 95 CPU in FP16 due to decision-model size).
2. **Safety-critical test vectors for each agent**: Build a set of inputs where the correct output is known and safety-critical. For VisionAgent: 20 frames with fire present at various severity levels. For VoiceAgent: 15 audio clips with "FIRE" in various acoustic conditions.
3. **SafetyOrchestrator threshold preservation test**: Verify that INT8/FP16 outputs still cross decision thresholds at the same input values as FP32. Even a 1% probability shift on a 0.85 threshold matters.
4. **Non-determinism measurement**: Run the same safety-critical inputs 10 times on Kinara NPU hardware. Quantify output variance. If safety-critical outputs are non-deterministic, implement deterministic inference mode.

### TOOLS
- **NXP eIQ Toolkit + Kinara SDK**: For INT8 conversion and Kinara NPU deployment
- **pytest safety validation suite**: `scripts/agent_safety_quant_test.py` — see Automation Playbook
- **Non-determinism measurement script**: Run 10× and compute output variance

### TEMPLATE — Per-Agent Safety Quantization Validation Report

```markdown
## Safety Quantization Validation Report — BuildingSafe v1.0
Date: 2026-06-15 | FSE: [Name] | SIL: 2

### VisionAgent — INT8 (Kinara NPU)
Safety test vectors: 20 fire-present frames at severity levels S1-S4
| Severity | FP32 P(fire) | INT8 P(fire) | FP32 Threshold Cross | INT8 Threshold Cross | Match? |
|---------|-------------|-------------|---------------------|---------------------|--------|
| S1 (smoke) | 0.71 | 0.69 | Alert (≥0.60) ✓ | Alert (≥0.60) ✓ | ✅ |
| S2 (flames visible) | 0.88 | 0.87 | Zone evac (≥0.85) ✓ | Zone evac (≥0.85) ✓ | ✅ |
| S3 (active fire) | 0.97 | 0.96 | Full evac (≥0.95) ✓ | Full evac (≥0.95) ✓ | ✅ |
| S4 (structural fire) | 0.99 | 0.98 | Full evac ✓ | Full evac ✓ | ✅ |
All 20 vectors: 20/20 threshold match ✅

### VoiceAgent — INT8 (Kinara NPU)
Safety test vectors: 15 audio clips (FIRE keyword, varying SNR)
| SNR (dB) | FP32 Confidence | INT8 Confidence | FP32 Action | INT8 Action | Match? |
|---------|----------------|----------------|------------|------------|--------|
| 30 (quiet room) | 0.96 | 0.95 | Alert ✓ | Alert ✓ | ✅ |
| 15 (moderate noise) | 0.83 | 0.81 | Alert ✓ | Alert ✓ | ✅ |
| 5 (loud factory) | 0.62 | 0.58 | Alert ✓ | Alert ✓ | ✅ |
| 0 (very noisy) | 0.41 | 0.38 | No action ✓ | No action ✓ | ✅ |
All 15 vectors: 15/15 threshold match ✅

### SafetyOrchestrator — FP16 (i.MX 95 CPU)
Decision threshold boundary test: 50 input combinations spanning all 4 decision levels
Threshold boundary violations: 0 ✅
Non-determinism test (10 runs, same input): Max output variance = 0.003 — within tolerance ✅

### Overall SIL-2 Quantization Sign-Off
VisionAgent: PASS ✅ | VoiceAgent: PASS ✅ | SafetyOrchestrator: PASS ✅
FSE sign-off: [Name] | Date: 2026-06-16
```

---

## L5 — Validation & Testing

### WHY
A false positive hospital evacuation is itself a patient safety incident. The validation regime for BuildingSafe must produce evidence that the system handles the worst-case governance scenario — an ICU evacuation trigger — with appropriate safeguards. Functional safety review at SIL-2 requires independent assessment. Adversarial testing must include real-world edge cases: HVAC noise triggering VoiceAgent, a smoke machine test triggering VisionAgent, sun glare creating false fire signatures. These are not hypotheticals — they are documented causes of real building safety system failures.

### WHAT (Controls Triggered)
`FS-07` Functional safety review sign-off (SIL-2) | `FS-08` Fail-safe activation testing | `FS-09` Independent safety validation (SIL-2 mandatory) | `HO-03/04` Kill-switch + override testing | `SR-06` Adversarial robustness

### HOW
1. **Independent safety validation** (SIL-2 requirement): Engage an independent assessor (not on the development team) to review the safety case. This is not optional at SIL-2.
2. **Fail-safe activation test**: For each failure mode in the HARA, trigger the failure in a test environment and measure: does the system reach safe state? How quickly?
3. **Kill-switch test**: Can the facility manager manually override any active alert/evacuation from the control panel within 60 seconds?
4. **Adversarial testing**: Test VisionAgent with sunlight glare, fog, flash photography. Test VoiceAgent with HVAC noise, crowd noise, multiple simultaneous speakers.
5. **False positive measurement**: Run the system in passive monitoring mode on a test building for 72 hours. Count false alerts. Target: <1 per 24 hours.

### TOOLS
- **Functional safety review form**: Formal document with independent assessor sign-off
- **Test execution log**: Timestamped evidence of each test and result
- **False positive monitoring log**: 72-hour passive monitoring data

### TEMPLATE — Functional Safety Review Sign-Off Form

```markdown
## Functional Safety Review — BuildingSafe v1.0
SIL: 2 | Date: 2026-06-22 | Independent Assessor: [External Safety Firm Name]
Assessment basis: IEC 61508 §8.3, ISO/IEC TR 5469 §6.5

### Review Scope
- [x] HARA completeness: All 5 hazards addressed, mitigations documented
- [x] SIL determination: Methodology correct, SIL 2 confirmed
- [x] Fail-safe behavior: Specifications complete and tested
- [x] Quantization safety validation: SIL-2 validation complete (report: quant-validation-v1.0.md)
- [x] Prohibited decisions registry: Complete, GC-approved
- [x] Agent disagreement rules: Formally specified and tested
- [x] Kill-switch: Tested, 45-second activation-to-safe-state measured

### Fail-Safe Activation Test Results
| Failure Mode | Trigger Method | Time to Safe State | Target (FTTI) | Pass/Fail |
|-------------|---------------|-------------------|---------------|-----------|
| VisionAgent crash | Kill process | 3.2 seconds | <10 seconds | ✅ PASS |
| VoiceAgent crash | Kill process | 2.8 seconds | <10 seconds | ✅ PASS |
| SafetyOrchestrator crash | Kill process | 1.1 seconds (watchdog) | <5 seconds | ✅ PASS |
| Network disconnection | Unplug ethernet | 4.0 seconds (buffer flush) | <10 seconds | ✅ PASS |
| Power-cycle during active alert | Hard power off | Alert resumes on boot in <15s | <30 seconds | ✅ PASS |

### Adversarial Test Results (Selected)
| Test | Input | Expected | Actual | Pass/Fail |
|------|-------|---------|--------|-----------|
| Sunlight glare on lens | Direct sun at 30° incidence | No fire detection | No false detection | ✅ PASS |
| HVAC noise (90dB) | Continuous HVAC | No false keyword | 1 false "FIRE" at 94dB | ⚠️ Mitigated: raised audio threshold to 0.85 at SNR <5dB |
| Smoke machine (prop) | Non-fire smoke, 10m | Alert (correct) | Alert triggered at 4.2s | ✅ PASS |
| Crowd noise + real fire keyword | 15 people talking + "FIRE" | Alert | Alert triggered 6.1s | ✅ PASS |

### False Positive Rate (72-hour passive test, empty office building)
Total false alerts: 2 in 72 hours (once from window reflection; once from steam cleaner)
Rate: 0.67 per 24 hours — within target (<1/24h) ✅

### Overall Determination
This assessment confirms BuildingSafe v1.0 meets SIL-2 functional safety requirements
with the following conditions:
1. Audio threshold adjusted to 0.85 at SNR <5dB (implemented in v1.0.1)
2. Pilot deployment limited to non-clinical zones for first 90 days per GC condition

Independent Assessor sign-off: [Name, Firm, Cert No.] | 2026-06-23
FSE sign-off: [Name] | 2026-06-23
```

---

## L6 — Deployment & Fleet Management

### WHY
Deploying BuildingSafe to a hospital requires an authorization chain that goes beyond standard software governance. The hospital has its own safety officer, accreditation obligations, and liability framework. The standard 3-party OTA authorization (MRO + AIPO + DSL) is insufficient — the facility's safety authority must sign off. Getting this wrong means the hospital deploys a system that the building's safety officer did not formally accept responsibility for — a liability and regulatory gap that becomes critical in a post-incident investigation.

### WHAT (Controls Triggered)
`ML-10` Extended 4-party OTA authorization | `ML-11` Staged rollout (non-clinical → clinical) | `ML-12` Per-device agent version tracking | `HO-05/06` Kill-switch operational test + operator training | `FS-10` ODD enforcement | `RC-07` EU AI Act high-risk registration (if classified high-risk)

### HOW
1. **Extended authorization chain**: Add Facility Safety Officer as a required sign-off (4th party) for any hospital deployment.
2. **Staged rollout — building type progression**: Stage 1: car park (no clinical risk). Stage 2: office/admin areas. Stage 3: general wards (90 days, per GC condition). Stage 4: clinical areas including ICU (only after Stage 3 review).
3. **Facility Safety Officer training**: Minimum 4-hour training on: what the system does, how to interpret alerts, how to cancel/override, kill-switch procedure, incident reporting.
4. **ODD enforcement configuration**: Upload the building floor plan and define the ODD (e.g., camera coverage zones, areas where VisionAgent operates outside its validation). Configure ODD monitoring to alert when a camera zone falls outside the validated envelope (e.g., after construction).
5. **Per-device agent version tracking**: Every device (i.MX 95 unit) must report VisionAgent version, VoiceAgent version, and SafetyOrchestrator version to the fleet management system.

### TOOLS
- **Extended deployment authorization form**: 4-party sign-off (below)
- **ODD monitoring configuration**: YAML floor plan mapping — see Automation Playbook
- **Training completion records**: Per-facility, per-authorized operator

### TEMPLATE — Extended 4-Party Deployment Authorization Form

```markdown
## Deployment Authorization — BuildingSafe v1.0.1
Date: 2026-07-05 | Facility: St. Mary's Hospital, Building B (Non-clinical only — Stage 2)

### Pre-deployment checklist
- [x] L1 decision record + GC approval complete
- [x] DPIA complete + DPO sign-off
- [x] SIL-2 independent validation complete (report: fs-review-v1.0.md)
- [x] Quantization validation: All 3 agents PASS
- [x] Fail-safe activation test: All 5 failure modes PASS
- [x] False positive rate: 0.67/24h in test environment ✅
- [x] Prohibited decisions registry: Accepted by Facility Safety Officer
- [x] ODD defined for Building B: 24 cameras in admin areas, floors 1-3
- [x] Kill-switch tested at facility: Activation to safe state = 38 seconds ✅
- [x] Operator training: 3 authorized operators trained (Building B security team)

### Authorization signatures (4-party requirement)
| Role | Name | Organization | Sign-off | Date |
|------|------|-------------|---------|------|
| Model Risk Officer | [MRO Name] | Development team | ✓ | 2026-07-04 |
| AI Product Owner | [AIPO Name] | Development team | ✓ | 2026-07-04 |
| Device Security Lead | [DSL Name] | Development team | ✓ | 2026-07-05 |
| Facility Safety Officer | [FSO Name] | St. Mary's Hospital | ✓ | 2026-07-05 |

### Staged rollout plan (this deployment: Stage 2 — non-clinical only)
| Stage | Scope | Duration | Advance criteria |
|-------|-------|----------|-----------------|
| Stage 1 (complete) | Car park, Building B | 4 weeks | <1 false alert/24h ✅ |
| Stage 2 (this) | Admin floors 1-3, Building B | 90 days | <0.5 false alerts/24h; 0 missed real alerts |
| Stage 3 | General wards, Building B | 90 days post GC review | Independent audit required |
| Stage 4 | ICU, Operating theatres | Post Stage 3 approval | External clinical safety audit required |

### Emergency rollback
Fleet rollback time target: <2 hours
Rollback command: `buildsafe-fleet rollback --facility stmarys-bldg-b --version v0.9.2`
```

---

## L7 — Operation & Monitoring

### WHY
After deployment, the most dangerous governance failure is silent ODD drift. When the hospital renovates a ward and relocates walls, the VisionAgent camera zones may no longer match the validated building model — and nobody tells the AI system. The model continues operating with full confidence in a physical environment it was never validated for. The governance program must catch this before a real emergency exposes it. Additionally, if the override rate creeps up (staff are canceling alerts frequently), this is a governance signal — either the model is generating too many false positives, or staff have developed automation bias and are over-canceling.

### WHAT (Controls Triggered)
`FS-11` Safety performance monitoring | `FS-12` Prohibited decisions registry (ongoing compliance) | `HO-07` Override rate monitoring | `HO-08` Remote intervention authentication | `HO-09` Automation bias monitoring | `ML-13` Model drift | `XT-07/08` Audit log retention + decision explainability | `DG-10` Corpus/model integrity

### HOW
1. **Daily**: Automated safety metrics check — false positive rate, override rate, agent confidence distributions.
2. **Weekly**: Review override rate trend. If >5% weekly overrides: investigate with facility staff.
3. **Monthly**: Download and review a sample of audit log entries for explainability quality. Can you explain every triggered alert from the log?
4. **On ODD change alert**: When the ODD monitoring system flags a camera zone as outside the validated envelope (e.g., VisionAgent detection area now partially blocked by a new wall) → immediate human review required before the zone can continue operating autonomously.
5. **Quarterly**: Automation bias calibration exercise — present 10 scripted alert scenarios to building security staff, including intentional false positives. Measure if staff are over-canceling.

### TOOLS
- **Safety monitoring dashboard**: Grafana (or equivalent) — see ODD monitoring YAML in Automation Playbook
- **Weekly safety metrics report**: Template (below)
- **Automation bias calibration**: Structured exercise protocol

### TEMPLATE — Weekly Safety Metrics Report

```markdown
## BuildingSafe Weekly Safety Metrics — St. Mary's Hospital, Building B
Period: 2026-07-07 to 2026-07-13 | Prepared by: MRO

### Alert Activity
| Metric | This Week | Last Week | Threshold | Status |
|--------|----------|----------|-----------|--------|
| Total alerts triggered | 12 | 14 | — | — |
| True positive alerts (confirmed by facility) | 10 | 13 | — | — |
| False positives | 2 | 1 | <3/week | ✅ |
| False positive rate | 0.29/day | 0.14/day | <0.5/day | ✅ |
| Alerts cancelled by operator within 30s | 3 | 2 | <5/week | ✅ |
| Override rate (% of alerts overridden) | 25% | 14% | <20% | ⚠️ ELEVATED |

### Model Confidence Distributions
VisionAgent mean confidence (all detections): 0.82 (baseline: 0.83) — no drift ✅
VoiceAgent mean confidence (all detections): 0.78 (baseline: 0.79) — no drift ✅

### ODD Status
Camera zones within ODD: 24/24 ✅
ODD alerts this week: 0

### Action Required
⚠️ Override rate elevated (25% vs 20% threshold). Investigation: 2 of the 3 overrides were by
Security Officer B, all on same camera zone (Zone 7 — loading bay). Possible cause: sun glare
during afternoon delivery hours. Recommend: Review Zone 7 VisionAgent calibration for afternoon
light conditions.

Signed: [MRO Name] | 2026-07-14
```

---

## L8 — Retirement & Handover

### WHY
BuildingSafe has a 10-15 year deployment lifecycle in a hospital environment. Retirement does not mean simply unplugging the system — it means maintaining safety continuity during the transition to a successor system. A hospital cannot be unprotected during changeover. Additionally, SIL-2 safety case documentation must be archived for 10 years, and any personal data (VisionAgent anonymized training contributions, audit logs) must be handled per GDPR. The safety case archive is a legal artifact — it may be required in a post-incident investigation years after the system is retired.

### WHAT (Controls Triggered)
`FS-13` Safety case archiving (10 years) | `ML-16` End-of-life procedure | `ML-17` Device data purge | `RC-11` Regulatory documentation archiving | `XT-09` Audit log archiving | `DP-10` Personal data purge

### HOW
1. **Successor system validation before decommission**: BuildingSafe v1.0 continues operating until the successor system has completed its own L5 validation. A parallel running period of minimum 30 days (both systems monitoring, neither triggering) is required.
2. **Safety case archive**: Package the full safety case (HARA, SIL determination, quantization validation, functional safety review, all audit logs) into a signed, tamper-evident archive.
3. **Secure data purge**: Run certified secure erase on all i.MX 95 device storage. Issue destruction certificates.
4. **Audit log transfer**: Transfer all alert logs to facility's building management system for the 10-year legal retention period.

### TEMPLATE — Retirement and Handover Checklist with Safety Continuity Gate

```markdown
## BuildingSafe Retirement Checklist — St. Mary's Hospital, Building B
Initiated: 2031-03-01 | Completed: 2031-04-15

### Safety Continuity Gate (MUST complete before decommission begins)
- [x] Successor system (BuildingSafe v3.0) has completed L5 independent safety validation ✓
- [x] Parallel monitoring period: 30 days complete (2031-03-01 to 2031-03-31)
- [x] Parallel period anomalies: 0 (both systems agreed on all 47 alerts) ✓
- [x] Successor system authorized for operational handover ✓

### Data and Documentation
| Task | Status | Date | Notes |
|------|--------|------|-------|
| Audit logs exported from all devices | ✅ | 2031-03-05 | 4.7M events, transferred to BMS |
| Audit log archive integrity verified | ✅ | 2031-03-06 | SHA-256 verified |
| Safety case archive packaged | ✅ | 2031-04-01 | governance-archive/buildsafe-v1.0/ |
| Safety case archive signed + timestamped | ✅ | 2031-04-01 | Qualified electronic timestamp |
| 10-year retention confirmed | ✅ | 2031-04-01 | Retention until 2041-04-01 |
| All device storage securely erased | ✅ | 2031-04-10 | Certified secure erase |
| Certificates of destruction issued | ✅ | 2031-04-10 | Cert ref: COD-2031-012 |
| EU AI Act de-registration (if registered) | ✅ | 2031-04-12 | EU database notified |
| Model registry updated: RETIRED | ✅ | 2031-04-15 | |

FSE sign-off (safety continuity confirmed): [Name] | 2031-04-15
DPO sign-off (personal data destruction confirmed): [Name] | 2031-04-15
Facility Safety Officer sign-off: [Name] | 2031-04-15
```

---

## Governance Summary Table

| Stage | Controls Triggered | Key Evidence Produced |
|-------|-------------------|----------------------|
| L1 | HO-01/02, FS-01/02/12, EU-01/02, DP-01, RC-01/02 | Multi-agent HARA, GC approval memo, SIL-2 determination, Prohibited decisions registry, DPIA |
| L2 | DP-01/03, DG-02/04, FB-02/03 | DPIA, Per-agent dataset documentation, Representativeness matrix |
| L3 | ML-03/04 (×3), FS-04, LM-03, DG-07 | Per-agent training run logs, SafetyOrchestrator constraint policy, 3 model card drafts |
| L4 | ML-05/06/07, FS-05/06 | Per-agent safety quantization validation report |
| L5 | FS-07/08/09, HO-03/04, SR-06 | Independent safety review report, Fail-safe activation test results, Adversarial test log |
| L6 | ML-10/11/12, HO-05/06, FS-10 | Extended 4-party deployment authorization, Staged rollout log, ODD config |
| L7 | FS-11/12, HO-07/08/09, ML-13, XT-07/08 | Weekly safety metrics reports, ODD alert log, Automation bias calibration results |
| L8 | FS-13, ML-16/17, RC-11, XT-09, DP-10 | Safety case archive, Retirement checklist, Certificates of destruction |

---

## Automation Playbook

### 1. Governance Manifest (`governance/governance-manifest.yaml`)

```yaml
# governance/governance-manifest.yaml
project: "buildsafe-v1.0"
product: "eIQ Agentic AI Framework"
hardware: "i.MX 95 + Kinara NPU"
agents: ["VisionAgent", "VoiceAgent", "SafetyOrchestrator"]
autonomy_level: 3
eu_ai_act_classification: "high_risk_pending_review"
sil: 2
gc_approved: true
gc_approval_date: "2026-06-02"

stages:
  L1_conception:
    status: complete
    controls:
      HO-01: { status: complete, evidence: "governance/l1-gc-approval-memo.md" }
      FS-01: { status: complete, evidence: "governance/l1-hara-multiagent.md" }
      FS-02: { status: complete, evidence: "governance/l1-sil-determination.md" }
      FS-12: { status: complete, evidence: "governance/prohibited-decisions-registry.md" }
      DP-01: { status: complete, evidence: "governance/l1-dpia.md" }
      EU-02: { status: complete, evidence: "governance/l1-eu-ai-act-classification.md" }

  L2_data:
    status: complete
    controls:
      DP-01: { status: complete, evidence: "governance/dpia-vision-biometric.md" }
      DG-02: { status: complete, evidence: "governance/dataset-docs/" }
      FB-02: { status: complete, evidence: "governance/representativeness-matrix.md" }

  L3_training:
    status: complete
    controls:
      LM-03: { status: complete, evidence: "governance/orchestrator-constraints-v1.0.md" }
      FS-04: { status: complete, evidence: "governance/orchestrator-constraints-v1.0.md" }
      ML-04: { status: complete, evidence: "governance/model-cards/" }

  L4_quantization:
    status: complete
    controls:
      ML-05: { status: complete, evidence: "governance/reports/quant-regression-v1.0.md" }
      ML-06: { status: complete, evidence: "governance/reports/quant-regression-v1.0.md" }
      FS-05: { status: complete, evidence: "governance/reports/sil2-quant-validation.md" }
      FS-06: { status: complete, evidence: "governance/reports/nondeterminism-test.md" }

  L5_validation:
    status: complete
    controls:
      FS-07: { status: complete, evidence: "governance/reports/safety-review-v1.0.md" }
      FS-09: { status: complete, evidence: "governance/reports/independent-assessment.md" }
      HO-03: { status: complete, evidence: "governance/kill-switch-test.md" }

  L6_deployment:
    status: in_progress
    controls:
      ML-10: { status: complete, evidence: "governance/deployment-auth-stmarys-stage2.md" }
      ML-11: { status: in_progress, evidence: "governance/rollout-log.md" }
      FS-10: { status: complete, evidence: "governance/odd-config-stmarys.yaml" }

  L7_operations:
    status: in_progress
    controls:
      FS-11: { status: in_progress, evidence: "governance/safety-metrics/" }
      HO-07: { status: in_progress, evidence: "governance/override-rate-log.md" }

  L8_retirement:
    status: pending
```

### 2. GitHub Actions Governance Gate (`.github/workflows/governance-gate.yml`)

```yaml
name: BuildingSafe Governance Gate — Block Deployment Without SIL-2 Evidence

on:
  push:
    tags:
      - 'deploy-*'

jobs:
  check-governance:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pip install pyyaml
      - name: Check governance manifest
        run: python scripts/check_governance_gate.py governance/governance-manifest.yaml
      - name: Verify SIL-2 specific gates
        run: python scripts/check_sil2_gate.py governance/governance-manifest.yaml
        # Fails if: FS-05, FS-06, FS-07, FS-09 not all 'complete'
        # Fails if: gc_approved != true
        # Fails if: prohibited_decisions_registry not referenced
```

### 3. Per-Agent Drift Monitor (`scripts/agent_drift_monitor.py`)

```python
"""
agent_drift_monitor.py
Monitors per-agent confidence score distributions in production.
Compares current 7-day window against deployment baseline using KS test.
Alerts if any agent shows statistically significant drift.
"""
import json, os
from scipy import stats
import numpy as np

BASELINE_STATS_PATH = "governance/baselines/confidence-baseline.json"
RECENT_LOGS_PATH    = "data/inference-logs/recent-7d.jsonl"
DRIFT_PVALUE_THRESHOLD = 0.05  # Alert if KS test p-value < 0.05
AGENTS = ["VisionAgent", "VoiceAgent", "SafetyOrchestrator"]

def load_baseline():
    with open(BASELINE_STATS_PATH) as f:
        return json.load(f)

def load_recent_confidences():
    confidences = {agent: [] for agent in AGENTS}
    with open(RECENT_LOGS_PATH) as f:
        for line in f:
            log = json.loads(line)
            agent = log.get("agent")
            if agent in AGENTS:
                confidences[agent].append(log["confidence_score"])
    return confidences

def check_drift():
    baseline = load_baseline()
    recent = load_recent_confidences()
    alerts = []

    for agent in AGENTS:
        if len(recent[agent]) < 100:
            print(f"⚠️  {agent}: insufficient recent data ({len(recent[agent])} samples)")
            continue

        ks_stat, p_value = stats.ks_2samp(baseline[agent]["confidences"], recent[agent])
        if p_value < DRIFT_PVALUE_THRESHOLD:
            alerts.append({
                "agent": agent,
                "ks_statistic": round(ks_stat, 4),
                "p_value": round(p_value, 4),
                "recent_mean": round(np.mean(recent[agent]), 4),
                "baseline_mean": round(np.mean(baseline[agent]["confidences"]), 4)
            })
            print(f"🔴 DRIFT DETECTED — {agent}: KS={ks_stat:.4f}, p={p_value:.4f}")
        else:
            print(f"✅ {agent}: No drift (p={p_value:.4f})")

    if alerts:
        print(f"\n⚠️  {len(alerts)} agent(s) show drift. Escalate to MRO.")
        # TODO: send_alert(alerts)
        exit(1)

if __name__ == "__main__":
    check_drift()
```

### 4. ODD Monitoring Configuration (`governance/odd-config-stmarys.yaml`)

```yaml
# ODD monitoring configuration — St. Mary's Hospital, Building B
facility: "stmarys-hospital-bldg-b"
deployment_stage: 2
ood_alert_enabled: true

camera_zones:
  - zone_id: "zone-01"
    location: "Ground floor lobby"
    coverage_validated: true
    floor_plan_hash: "sha256:a3f1c2..."
    alert_if_coverage_drops_below: 0.80  # 80% of validated coverage area
  - zone_id: "zone-07"
    location: "Loading bay"
    coverage_validated: true
    floor_plan_hash: "sha256:b4d2e1..."
    alert_if_coverage_drops_below: 0.80
    known_issue: "Sun glare afternoons 14:00-17:00 — audio-only mode during this window"
    audio_only_window: "14:00-17:00"

ood_triggers:
  - trigger: "camera_coverage_below_threshold"
    action: "alert_human_operator"
    escalation: "do_not_auto_trigger_evacuation"
  - trigger: "floor_plan_hash_mismatch"
    action: "alert_human_operator + suspend_zone"
    escalation: "mro_notification_within_4_hours"
  - trigger: "agent_offline"
    action: "activate_failsafe_mode"
    escalation: "immediate_alert"
```

---

*Example 3 | BuildingSafe — eIQ Agentic AI Framework on i.MX 95 + Kinara NPU | Version 1.0 — 2026-05-29*
