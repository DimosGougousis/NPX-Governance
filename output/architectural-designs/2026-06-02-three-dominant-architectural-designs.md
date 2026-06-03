# Three Dominant Architectural Designs for AI Governance

> **Date:** 2026-06-02 | **Status:** Architectural proposal — pick one as the master pattern
>
> **Scope:** Each design is a complete, internally-consistent governance architecture for the NXP edge AI portfolio. The three are intentionally orthogonal in their core organizing logic, so adoption is a real strategic choice rather than a cosmetic one.
>
> **Companion:** `00-framework-overview.md` defines the 12-domain × 8-lifecycle × 141-control body of controls. This document describes **three structurally different ways to organise and enforce that body** at the architectural level.

---

## How to Read This Document

The three designs are not stages of maturity and not layers of a stack. They are **three competing answers to one question**:

> *Where does governance authority live?*

| Design | Authority lives in | Best for |
|--------|--------------------|----------|
| **A — The Trust Stack** | The hardware + a chain of attestable roots of trust | Safety-critical regulated edge (automotive, medical, industrial) |
| **B — The Lifecycle Spine** | The engineering toolchain — the tool that produces the model is the thing that governs it | High-velocity internal product teams and customer-tooling plays |
| **C — The Agentic Operating System** | A policy engine + a multi-agent workforce that wraps the entire workflow | Agentic AI products, LLM/RAG pipelines, and systems that govern themselves at runtime |

Read §1 for the architectural thesis, then §2–§4 for each design. §5 is the comparison matrix and the **adoption heuristic** for picking one.

---

## 1. Architectural Thesis — Why "Three Designs" and Not One

AI governance has converged on a common *body of controls* (EU AI Act, ISO/IEC 42001, NIST AI RMF all point at the same risks). The NXP framework already encodes that body: 12 domains, 141 controls, an 8-stage lifecycle, and a RACI matrix.

What has **not** converged is the **system architecture** that enforces those controls. The same 141 controls can be enforced by:

- a stack of attestable hardware + signed firmware + audited device boot (authority rooted in silicon),
- an instrumented engineering toolchain that refuses to emit a non-compliant artifact (authority rooted in the build pipeline), or
- a runtime policy engine orchestrated by specialised governance agents (authority rooted in software that watches software).

Each is a *dominant* design — it sets the shape of every other decision. Picking one early is the single highest-leverage governance choice a programme makes.

The three designs below are described at the same level of fidelity (8 sections each: mental model, architecture, control mapping, enforcement, evidence, failure modes, when to pick, when *not* to pick) so they can be compared directly.

---

## Design A — The Trust Stack

> *Governance authority is anchored in the silicon. Every layer above is an attestable extension of a hardware root of trust.*

### A.1 Mental Model

Picture governance as a **vertical column of attestations**:

```
Physical process
    ↓ signed attestation
Operating-system kernel
    ↓ measured boot + remote attestation
eIQ inference runtime
    ↓ signed model artifact
Quantized model (in NPU memory)
    ↓ on-device decision log
Decision record (append-only, signed)
    ↓ batch sync
Fleet-level audit trail
```

The crucial property: **a forgery in any layer breaks the chain.** A model that wasn't loaded by a measured boot cannot present a valid attestation. A decision record that wasn't emitted by a runtime that was itself measured-validated is rejected by the audit trail. Authority flows *up* through verified measurements, not *down* through policies.

This is the same model used in financial HSMs, secure-element payment, and confidential computing. Applied to AI, it turns governance from a software claim ("our system complies") into a **provable property of the device** ("this device, in this configuration, with this model, has produced this decision, and you can verify the chain cryptographically without trusting the vendor").

### A.2 Architecture — Five Layers, Five Roots of Trust

| # | Layer | Root of trust | What gets attested | Failure if attestation fails |
|---|-------|---------------|--------------------|------------------------------|
| **L1** | **Hardware** | NXP Immutable Root of Trust (iROM) → Hardware Root of Trust (HRoT) → Secure Enclave | Boot ROM, immutable keys, secure boot chain, eFuse-burned identity | Device never boots to a governed state; no AI ever runs |
| **L2** | **Firmware & OS** | Measured boot (PCR extension) over verified boot | Kernel, eIQ runtime, ML/NPU drivers, inference engine | OS refuses to load the AI runtime; device enters safe state |
| **L3** | **Model & Data** | Signed model artifacts + signed vector DB hash | Model weights (SHA-256), quantization config, RAG corpus hash, calibration set hash | Model cannot be loaded; no inference executes |
| **L4** | **Inference & Decision** | Runtime TPM-style attestation + on-device ledger | Input, model version, output, confidence, decision timestamp | Decision not emitted; audit trail gaps; operator cannot trust the device |
| **L5** | **Fleet & Audit** | Manufacturer-signed device certificates + signed audit batches | Device identity, attestation history, decision batches | Fleet management system rejects the device; OTA blocked |

**Authority flow:** L1 attests L2. L2 attests L3. L3 attests L4. L4 signs decisions and emits them to L5.

**The 18-control pre-deployment gate is enforced at L2→L3 and L3→L4 transitions** — the model cannot enter the device's NPU memory unless the manifest's critical controls are all `complete`, the model's signature is valid, *and* the manifest is itself signed by an authority (AIPO/MRO) whose certificate chains to the fleet CA.

### A.3 Control Mapping — Where Each Domain Lands in the Stack

| Domain | Stack layer (primary) | What is enforced there |
|--------|----------------------|------------------------|
| D1 Data Governance | L3 (corpus hash, datasheet baked into vector DB image) | The RAG corpus is signed; only owner-approved documents enter |
| D2 LLM/SLM Governance | L3 (signed model card + license binding) | The model is bound to its license and intended use at signing time |
| D3 Security & Adversarial | L1 + L2 (HRoT, measured boot, SBOM in firmware) | The model and runtime cannot be tampered with; CVE tracking is enforced at OTA |
| D4 Privacy & Data Protection | L3 + L4 (signed DPIA reference, anonymisation at edge) | PII never leaves device; decision log is hashed and minimised |
| D5 Fairness & Bias | L3 (fairness report baked into signed bundle) | Subgroup metrics travel with the model; deployment cannot skip them |
| D6 Ethics & Acceptable Use | L3 + L4 (prohibited-use registry in runtime) | The runtime refuses actions on the prohibited registry; PAD registry loaded at boot |
| D7 Explainability & Transparency | L4 (on-device XAI, signed explanations) | Every decision carries a signed explanation; XAI is on-device because cloud round-trips are not available |
| D8 Human Oversight & Control | L1 + L4 (hardware kill-switch, runtime authority checks) | Kill-switch is a physical pin; autonomy-level checks run in the runtime |
| D9 MLOps & Model Lifecycle | L3 (signed artifacts, OTA bundle) | OTA updates require new signed bundle; reproducibility baked into model metadata |
| D10 Regulatory & Standards Compliance | L5 (audit trail + EU AI Act registration hash) | Every device's attestation history maps to a regulatory record |
| D11 Environmental & Sustainability | L1 + L4 (power budget, on-device computation) | Inference power is hardware-enforced; sustainability is measurable per device |
| D12 Functional Safety | L1 + L2 + L4 (safety MCU co-processor, safety kernel) | Safety functions live in a co-processor with its own attestation chain |

### A.4 Enforcement — How the Stack Refuses Non-Compliance

Three enforcement primitives, all anchored in hardware:

1. **Signed artifacts only.** The Neutron-converter output is signed by the MRO's signing key. The runtime loads nothing unsigned. The signing key is itself bound to the device's certificate chain — a stolen model cannot run on a different device.

2. **Measured boot with PCR extension.** The boot chain is measurable end-to-end. The manifest is itself an artifact the device measures at boot: if the expected L1 control state doesn't match, the runtime refuses to load the model. The EU AI Act risk class for the loaded model is in the manifest, and the runtime enforces scope (e.g. a "minimal-risk" model cannot be promoted to a safety function at runtime).

3. **Hardware kill-switch with attestation.** D8 HO-05 (kill-switch) is a physical pin backed by a safety MCU. The kill-switch test is itself a signed event. If the kill-switch fails, the device cannot enter the governed state — this is enforced at L1, not by software.

### A.5 Evidence — What the Stack Produces

| Evidence type | Where it lives | Verification path |
|---------------|----------------|-------------------|
| **Boot attestation** | L1→L2 PCR values, stored in secure enclave | Manufacturer + customer verify via remote attestation |
| **Model provenance** | Signed model manifest, signed SBOM | Anyone verifies via the manufacturer's signing chain |
| **Decision record** | On-device append-only ledger, batched to fleet | Decision hash chain verifiable offline, independent of vendor |
| **Compliance attestation** | EU AI Act registration hash linked to device serial | EU AI Act database cross-references device identity |
| **Audit trail** | Hash-chained log, periodically anchored to notarisation service | Independent third party verifies chain integrity |

The key property: **the customer, the regulator, and the auditor can each verify the chain independently.** They do not need to trust NXP, the customer, or the auditor — they verify the chain.

### A.6 Failure Modes

| Failure | Stack behaviour | What it tells you |
|---------|-----------------|-------------------|
| Boot PCR mismatch | Device enters safe state, refuses to load AI | Hardware tamper detected |
| Unsigned model | Runtime refuses to load; model cannot execute | Supply chain integrity broken |
| Manifest drift (model claims EU-AI-Act-class X, deployment context requires Y) | Runtime blocks inference; flag for re-classification | Misclassification caught at runtime, not in audit |
| Kill-switch test fails | Device cannot enter governed state; flagged for service | Hardware safety check failed |
| Decision ledger grows unbounded | Roll-forward to fleet, hash anchor on-chain, prune on-device | Storage pressure managed cryptographically |
| Fleet CA key compromise | Revoke + rotate; all devices re-attest | Trust anchor recovery path |

### A.7 When to Pick the Trust Stack

- **You ship into regulated safety contexts** (automotive ISO 26262 ASIL-B+, medical IEC 62304/ISO 14971, industrial IEC 61508 SIL 2+).
- **Your customers demand tamper-evidence**, not trust: defence, critical infrastructure, healthcare.
- **Your competitive moat is "verifiable"**: you can prove the device is governed, not just claim it.
- **Your product has a long operational lifetime** (10+ years) where post-deployment attestation matters more than fast iteration.

### A.8 When *Not* to Pick the Trust Stack

- Your product is advisory-only, minimal-risk under the EU AI Act, and ships in consumer volumes where every second of boot time and every kilobyte of secure storage matters more than attestation.
- You are early-stage and don't yet have a clear threat model — the stack is expensive to build before you know what you need to attest.
- Your customers deploy the model themselves (e.g. cloud LLM vendors); the stack is built for devices NXP controls, not for the model itself.

---

## Design B — The Lifecycle Spine

> *Governance authority is anchored in the engineering toolchain. The build pipeline is the governance pipeline; an artifact that survives the pipeline is, by construction, governed.*

### B.1 Mental Model

Picture governance as a **horizontal sequence of gates**:

```
Conception ──▶ Data ──▶ Train ──▶ Quantize ──▶ Validate ──▶ Profile ──▶ Deploy
   │             │          │           │            │             │            │
   ▼             ▼          ▼           ▼            ▼             ▼            ▼
 Gate-0      Gate-1     Gate-2     Gate-3       Gate-4        Gate-5       Gate-6
 (L1)         (L2)       (L3)       (L4)         (L5)         (L6)         (L7)
   │             │          │           │            │             │            │
   └─────────────┴──────────┴───────────┴────────────┴─────────────┴────────────┘
                                          │
                            Governance manifest (the ledger)
```

The crucial property: **no artifact passes the next gate without a written, signed, hash-chained control update.** A dataset without a datasheet cannot enter training. A quantized model without a regression report cannot be validated. A model without an Annex IV technical documentation pack cannot be deployed. The CI pipeline *is* the governance pipeline.

This is "shift-left" governance in its mature form: the most expensive retrofit (governance as audit) is replaced by the cheapest discipline (a pre-commit hook in the build pipeline). Every team that uses the toolchain gets governance for free, because the toolchain refuses to emit non-compliant artifacts.

### B.2 Architecture — Eight Gates, One Manifest, One Source of Truth

| Gate | Lifecycle stage | Hard requirement | What cannot happen without it |
|------|-----------------|------------------|-------------------------------|
| **G0** | L1 Conception | Ethics review signed, EU AI Act class set, autonomy level assigned, hazard analysis done | No dataset, no model, no project file can be created |
| **G1** | L2 Data | Datasheet, PII scan, representativeness report, DPIA (if needed) | Dataset cannot be ingested into the curated set |
| **G2** | L3 Training | Dataset manifest + model manifest, DPIA gate passed, seed captured | Training run cannot start |
| **G3** | L4 Quantize | Three regression reports (general / safety / subgroup), calibration set hash | Quantized model cannot advance to validation |
| **G4** | L5 Validate | Subgroup confusion matrix, adversarial probes, XAI validation, FSE sign-off for SIL≥1 | Validated model cannot advance to profiling |
| **G5** | L6 Profile | Latency, memory, power budget assertion, determinism check (if safety) | Profiled model cannot advance to deploy |
| **G6** | L7 Deploy | L1–L5 critical controls all `complete`, deployment authorisation signed, 4-party sign-off for safety-critical | No signed, deployable artifact is produced |
| **G7** | L8 Retire | Decommission plan, data purge, model recall from fleet | Model cannot be removed from catalog |

**The governance manifest** (Git-versioned YAML) is the single source of truth. Every gate writes to it. Every gate reads from it. The gate's contract is: *read the manifest, verify pre-conditions, perform the eIQ operation, write the result, sign the result*. The manifest itself is signed at every commit by the role that performed the work (AIPO, MRO, DSL, FSE).

### B.3 Control Mapping — Where Each Domain Lands in the Gates

| Domain | Gate(s) that fire | What is enforced |
|--------|-------------------|------------------|
| D1 Data Governance | G0, G1, G2 | Datasheet at G1; lineage and poisoning checks at G2 |
| D2 LLM/SLM Governance | G0, G3, G4, G6 | License review at G0; quantization tests at G3; adversarial/prompt-injection probes at G4 |
| D3 Security & Adversarial | G0, G3, G4, G6 | Threat model at G0; SBOM at G2; adversarial at G4; signed artifact at G6 |
| D4 Privacy & Data Protection | G0, G1, G6 | DPIA at G0/G1; data minimisation enforced at G1; no PII in decision log at G6 |
| D5 Fairness & Bias | G1, G3, G4 | Representativeness at G1; subgroup regression at G3; subgroup confusion at G4 |
| D6 Ethics & Acceptable Use | G0 | Ethics review + EU AI Act class + prohibited-use registry check |
| D7 Explainability & Transparency | G4, G6 | XAI validation at G4; model card publication at G6 |
| D8 Human Oversight & Control | G0, G4, G6 | Autonomy level at G0; FSE sign-off at G4 (if SIL); kill-switch test at G6 |
| D9 MLOps & Model Lifecycle | All gates | Manifest, lineage, reproducibility, version registry |
| D10 Regulatory & Standards Compliance | G0, G6, G7 | Scope determination at G0; Annex IV at G6; retirement record at G7 |
| D11 Environmental & Sustainability | G5, G6 | Power budget at G5; sustainability KPI at G6 |
| D12 Functional Safety | G0, G3, G4, G5, G6 | HARA + SIL at G0; safety regression at G3; FSE sign-off at G4; determinism at G5; 4-party sign-off at G6 |

### B.4 Enforcement — How the Spine Refuses Non-Compliance

The enforcement primitive is the **gate contract** — a small piece of code (a shell script, a Rego policy, a Python validator) that:

1. Reads the governance manifest
2. Verifies that all pre-conditions for the gate are `complete`
3. Performs the eIQ operation (import, train, quantize, validate, profile, deploy)
4. Writes the result to the manifest
5. Signs the manifest delta with the worker's key

Every gate is a *failure-closed* contract. If the manifest is missing a control, the gate does not proceed. The gates are the only path from one lifecycle stage to the next — there is no back door.

**Concrete enforcement layers:**

- **Pre-commit hooks** in the project repo (block commits that would write to the manifest without a signed delta).
- **CI pipeline** in the project repo (block merges that would skip a gate).
- **eIQ toolchain wrappers** (block eIQ Portal operations when the manifest is in an invalid state).
- **OTA pipeline** (block signed artifacts that don't have a green G6).

The **18-control pre-deployment gate** is the *intersection* of all gates — it is what every gate collectively enforces before deploy. Any team that uses the pipeline gets this enforcement by default; they cannot bypass it without explicit Governance Committee exception.

### B.5 Evidence — What the Spine Produces

| Evidence type | Where it lives | How it's verified |
|---------------|----------------|-------------------|
| **Governance manifest** | Git-versioned YAML in the project repo | Anyone with the repo can read the manifest and the history |
| **Gate reports** | `governance/reports/<gate>-<model>.<ext>` | Files are referenced from the manifest; their hashes are part of the manifest state |
| **Signed artifacts** | Object store; signatures are part of the artifact | The signing key chains to the role's certificate; signatures are publicly verifiable |
| **Audit log** | `governance/audit/YYYY-MM-DD.jsonl` (append-only, hash-chained) | Independent verifier walks the chain; tampered entries break the chain |
| **Gate pre-conditions** | Stored in the manifest as `controls.<id>.status` | Auditor reads the manifest; pre-conditions are self-evident |

### B.6 Failure Modes

| Failure | Spine behaviour | What it tells you |
|---------|-----------------|-------------------|
| Gate G1 fails (datasheet missing) | Training cannot start; CI red | Data governance discipline broke; team needs coaching |
| Manifest drift (manual edit detected) | Pipeline rejects; signer identity logged | Governance integrity issue; investigate |
| Gate G3 fails (subgroup regression > 2%) | Quantized model cannot advance; report shows the gap | Fairness regression; fix the model or change the dataset |
| Gate G6 fails (one L1–L5 control incomplete) | No signed deployable artifact; only unsigned test artifacts | Pre-deployment gate enforced; this is the most important property |
| Toolchain version drifts (locked version replaced) | Pipeline fails; CVE check mandatory | Toolchain governance integrity broken |
| Worker (e.g. CI runner) compromised | Signed deltas carry the worker's identity; investigation starts from the manifest | Workforce compromise detected and traceable |

### B.7 When to Pick the Lifecycle Spine

- **You have many product teams** building AI features into NXP-based systems, and you need governance to be the path of least resistance.
- **You ship the eIQ toolchain itself** (or a derivative): the spine is the *only* way to embed governance into the toolchain, so every customer inherits it.
- **Your product iteration velocity is high**: shift-left governance pays off the most when the cadence is weekly or faster.
- **You want customers to adopt your governance posture**: the spine is reusable across customer organisations, because the gates and the manifest schema are portable.

### B.8 When *Not* to Pick the Lifecycle Spine

- You ship *finished* AI systems to safety-critical contexts where the device itself needs to be attestable (the Trust Stack is required). The Spine and the Stack are complementary — the Stack attests the device, the Spine governs how the model was made.
- You have a single team and a single product; the Spine's gates are overkill, and a lighter-weight checklist suffices.
- Your product is a model you publish (e.g. an open-source LLM) — the Spine is built for the build pipeline, not the published artifact.

---

## Design C — The Agentic Operating System

> *Governance authority is anchored in a policy engine and a multi-agent workforce. Governance is a runtime concern; the system watches itself.*

### C.1 Mental Model

Picture governance as a **set of specialised agents** wrapping every AI action at runtime:

```
┌────────────────────────────────────────────────────────────────┐
│                   Orchestrator Agent                           │
│       (plans, dispatches, surfaces governance manifest)        │
└────────┬───────┬───────┬───────┬───────┬───────┬───────┬───────┘
         │       │       │       │       │       │       │
         ▼       ▼       ▼       ▼       ▼       ▼       ▼
    DataCustodian   ModelRegistrar   Quantizer   Validator   Profiler
                                                 │            │
                                                 ▼            ▼
                                          SafetyAdjudicator  Deployer
                                                 │            │
                                                 └──────┬─────┘
                                                        ▼
                                              PolicyEngine (the
                                              only thing that
                                              writes the manifest)
```

The crucial property: **the same governance hooks that govern build-time actions also govern runtime actions.** A model registry check at training time is the same policy the runtime calls when loading a model. A fairness check at validation is the same policy the runtime calls before serving a decision. A prohibited-use registry check at conception is the same policy the runtime calls before taking an action. **There is one policy, not two.**

This is the only design that scales to *agentic* AI — systems where the AI itself takes consequential actions, where the action loop is fast (sub-second), and where the set of possible actions is open-ended. You cannot gate an LLM agent's tool calls with a CI pipeline; the gate must be in the loop.

### C.2 Architecture — The Governance Kernel and the Agent Workforce

| Component | Role | Authority |
|-----------|------|-----------|
| **Policy Engine** (e.g. OPA/Rego) | Evaluates `(action, context) → ALLOW \| BLOCK \| REQUIRE_EVIDENCE \| REQUIRE_APPROVAL` | **Sole writer** of the governance manifest |
| **Manifest Ledger** | Git-versioned YAML or hash-chained DB | Single source of truth for governance state |
| **Hook Library** | Typed MCP tools with Rego policy bindings | The surface every agent uses to act |
| **Orchestrator** | Translates intent into agent calls; enforces stage state machine | The only agent that talks to the user |
| **Stage Workers** (e.g. DataCustodian, ModelRegistrar, Trainer, Quantizer, Validator, Profiler, Deployer, SafetyAdjudicator) | Single-responsibility agents, one per governance concern | Call hooks; cannot bypass the Policy Engine |
| **Audit Log** | Append-only, hash-chained, JSONL | The independent record of every action |

**Authority flow:** Worker → proposes an action via MCP tool call → Policy Engine evaluates → Decision recorded in the audit log and (if ALLOW or BLOCK with side-effect) the manifest → Worker acts or fails.

### C.3 Control Mapping — Where Each Domain Lands in the Agent OS

| Domain | Agent(s) primary | What is enforced |
|--------|------------------|------------------|
| D1 Data Governance | DataCustodian, SafetyAdjudicator | Datasheet gate, lineage tracking, data minimisation, PII policy |
| D2 LLM/SLM Governance | ModelRegistrar, Validator | License + EU AI Act check, prompt-injection probes, output policy |
| D3 Security & Adversarial | ModelRegistrar, Validator | SBOM, signed artifacts, adversarial probe suite (FGSM/PGD/garak) |
| D4 Privacy & Data Protection | DataCustodian, SafetyAdjudicator | DPIA gate, anonymisation policy, decision-log minimisation |
| D5 Fairness & Bias | Validator, SafetyAdjudicator | Subgroup regression, subgroup confusion at validation, runtime bias monitor |
| D6 Ethics & Acceptable Use | Orchestrator, SafetyAdjudicator | Prohibited-use registry checked at every action; autonomy-level scope check |
| D7 Explainability & Transparency | Validator, SafetyAdjudicator | XAI validation; on-device XAI; explanation attached to every decision |
| D8 Human Oversight & Control | SafetyAdjudicator, Orchestrator | Autonomy-level scope check; PAD registry check; human-in-the-loop trigger |
| D9 MLOps & Model Lifecycle | All stage workers | Manifest writes at every stage; reproducibility metadata |
| D10 Regulatory & Standards Compliance | Orchestrator, SafetyAdjudicator | EU AI Act class check; Annex IV doc on demand; serious incident routing |
| D11 Environmental & Sustainability | Profiler, Orchestrator | Power budget at every profile run; sustainability KPI dashboard |
| D12 Functional Safety | Validator, SafetyAdjudicator, Deployer | HARA, SIL, FSE sign-off, determinism, 4-party authorisation, PAD registry |

**The SafetyAdjudicator is the most distinctive agent.** It exists because agentic AI needs a runtime arbiter of safety decisions. It evaluates every action that could have safety consequences against the HARA, the SIL classification, the PAD registry, and the autonomy-level scope. It is the only agent that can override an Orchestrator decision — and it can be overridden only by an authorised human (a privilege that is itself gated by the runtime's role-based access control).

### C.4 Enforcement — How the Agent OS Refuses Non-Compliance

Two enforcement primitives:

1. **MCP tool calls are policy-bound.** Every MCP tool the agents can invoke is declared in a manifest with a Rego policy binding. The Policy Engine evaluates `(agent_id, tool, payload, current_manifest_state) → decision`. The agent cannot invoke an unregistered tool. The agent cannot bypass the policy. The agent's call is *typed* — the schema is part of the policy.

2. **Manifest is the only ledger, and only the Policy Engine writes it.** Workers call `policy.evaluate` and `manifest.read`. The Policy Engine writes the manifest in response. This makes the manifest tamper-evident by construction — workers cannot edit the manifest even if compromised.

**The 18-control pre-deployment gate is enforced by the Policy Engine** in the same way a Kubernetes admission controller enforces deployment constraints. Any deployment action that would skip a 🔴 control returns `BLOCK` and emits a `BlockedByPolicy` exception with a remediation hint.

### C.5 Evidence — What the Agent OS Produces

| Evidence type | Where it lives | How it's verified |
|---------------|----------------|-------------------|
| **Policy decisions** | Audit log, hash-chained | Independent verifier walks the chain; tampered entries break the chain |
| **Manifest snapshots** | Git-versioned YAML | Public; verifiable by anyone with the repo |
| **MCP tool invocations** | Audit log, with policy decision and context | Auditable end-to-end |
| **Agent decisions** | Decision log, signed by the agent's key | Independently verifiable |
| **Safety adjudications** | Dedicated log, with override history | Auditable for SIL-2/3 reviews |
| **A2A messages** | Logged with manifest ref | Optimistic-concurrency-safe; stale refs detected |

### C.6 Failure Modes

| Failure | Agent OS behaviour | What it tells you |
|---------|---------------------|-------------------|
| Agent A calls MCP tool X without being in `agents_authorized` | `UnauthorizedCaller` returned; Orchestrator receives tamper alert | Possible agent compromise or policy misconfiguration |
| Agent proposes an action that would skip a 🔴 control | `BlockedByPolicy` returned; remediation hint attached | Governance gate working as designed |
| Policy Engine goes offline | All agent actions queue; fail-closed — no action proceeds without a decision | Trust anchor failure; investigate before re-enabling |
| SafetyAdjudicator is overridden by an unauthorised actor | Override is logged; tamper alert raised | Either a real attack, or a role assignment error |
| Manifest version conflict (two workers editing at once) | Stale `manifest_ref` returned; worker re-reads | Optimistic concurrency working |
| Agent's LLM hallucinates an action | Policy Engine evaluates the action; if not in the registry, `BLOCK` | The policy is the safety net for LLM hallucination |

### C.7 When to Pick the Agentic OS

- **Your product is itself an agentic AI system** (e.g. ex-03 BuildingSafe, or any LLM-driven automation). You cannot bolt governance onto an LLM agent's tool calls after the fact — the gate has to be in the loop.
- **You build the platform other agentic systems are built on.** A2A + MCP are the standard contracts; the Agent OS is the reference implementation.
- **Your runtime action loop is fast** (sub-second) and your risk surface is open-ended. Pre-deployment gates are not enough — you need runtime policy.
- **You need self-governing systems** that can detect their own non-compliance, escalate to humans, and remediate without external orchestration.

### C.8 When *Not* to Pick the Agentic OS

- Your AI is batch and offline (e.g. weekly fraud scoring): the Lifecycle Spine is sufficient and simpler.
- You ship a single low-risk advisory product: a Trust Stack with minimal policy hooks is enough; the Agent OS is overkill.
- You do not yet have a clear policy DSL or a runtime that can enforce it. Building the OS before you have a clear policy is the wrong order — start with the controls (the framework body), then choose the architecture.

---

## 5. Comparison Matrix

| Dimension | A — Trust Stack | B — Lifecycle Spine | C — Agentic OS |
|-----------|-----------------|---------------------|-----------------|
| **Authority anchor** | Hardware (silicon) | Toolchain (build pipeline) | Policy engine + agents (software) |
| **Time of enforcement** | Boot + runtime | Build + deploy | Runtime (every action) |
| **Best for** | Safety-critical regulated edge | High-velocity product teams | Agentic AI products |
| **Audit strength** | Strongest (cryptographic, device-attested) | Strong (hash-chained, git-versioned) | Strongest at runtime (every action logged) |
| **Velocity of iteration** | Slowest (certification cycles) | Fastest (CI-driven) | Fast (runtime policy) |
| **Build cost** | Highest (requires device, secure boot, attestation infrastructure) | Medium (requires CI integration, manifest schema) | Medium-High (requires policy DSL, agent infrastructure) |
| **Customer effort** | Lowest at deployment (governance comes with the device) | Lowest at design (governance comes with the toolchain) | Lowest at runtime (policy comes with the agent runtime) |
| **Failure model** | "Boot fails → no AI runs" | "Gate fails → no AI ships" | "Policy blocks → no AI acts" |
| **Recovery** | Re-attest + revoke | Re-run gate | Re-evaluate policy |
| **Standalone or layered?** | Standalone for regulated; complements B+C for full stack | Standalone for high-velocity; complements A+C | Standalone for agentic; complements A+B for full stack |
| **Standards alignment** | ISO 26262, IEC 61508, IEC 62443, EU AI Act | EU AI Act, ISO/IEC 42001, NIST AI RMF | EU AI Act, ISO/IEC 42001, NIST AI RMF, OWASP LLM Top 10, MITRE ATLAS |

---

## 6. The Hybrid Pattern — All Three Composed

The three designs are not mutually exclusive. The most defensible production architecture **layers them**:

```
┌──────────────────────────────────────────────────────────────┐
│  C — Agentic OS (runtime policy engine, agent governance)    │
│  └─ wraps the live system, governs every action at runtime   │
└──────────────────────────────────────────────────────────────┘
        ▲
        │ governed by
        │
┌──────────────────────────────────────────────────────────────┐
│  B — Lifecycle Spine (build pipeline, gates, manifest)      │
│  └─ produces the model and runtime that C governs            │
└──────────────────────────────────────────────────────────────┘
        ▲
        │ attested by
        │
┌──────────────────────────────────────────────────────────────┐
│  A — Trust Stack (hardware, secure boot, attestation)        │
│  └─ makes the device that runs B and C trustworthy           │
└──────────────────────────────────────────────────────────────┘
```

**In words:** the Trust Stack makes the *device* trustworthy. The Lifecycle Spine makes the *model and runtime* trustworthy by enforcing the build-time gates. The Agentic OS makes the *action* trustworthy by evaluating every consequential action against a runtime policy. The three together cover the full governance surface: silicon → artifact → action.

For the NXP edge AI portfolio, this composition is the target architecture. The question is which of the three is the **entry point** — the one to build first, given the current portfolio, customer base, and competitive posture.

---

## 7. Adoption Heuristic — Which to Build First

| If your dominant reality is... | Start with... | Then layer... |
|-------------------------------|---------------|---------------|
| You ship safety-critical edge devices into automotive, medical, or industrial (i.MX 95 / Kinara NPU territory) | **A — Trust Stack** | B at build time, C for agentic features |
| You build the eIQ toolchain itself and want governance to be a property of the toolchain | **B — Lifecycle Spine** | A on customer devices, C for GenAI Flow |
| Your product IS an agentic AI system, or you build the platform other agentic systems run on | **C — Agentic OS** | B to govern the build, A if the runtime is on regulated edge |
| You have all three concerns but limited engineering capacity | **B — Lifecycle Spine** (highest leverage per engineer-hour) | C when the first agentic product ships, A when the first safety-critical product ships |

**The Lifecycle Spine (B) is the recommended default for an internal-first programme** because it is the design with the highest leverage per unit of investment: one instrumented pipeline governs every downstream model. The Trust Stack (A) is the recommended default for any product that will be deployed in a regulated safety context. The Agentic OS (C) is the recommended default for any team building the next generation of agentic AI products.

---

## 8. Companion Artifacts This Design Will Produce

Once the dominant architecture is chosen, the following implementation files follow (parallel to the harness build sequence in `output/2026-06-01-agentic-harness-design.md`):

- **If A is chosen:** `output/trust-stack/` — eIQ Runtime + Manifest Schema + Attestation Endpoints + Audit Verifier.
- **If B is chosen:** `output/lifecycle-spine/` — Gate implementations, manifest schema, CI integration templates, customer adoption guide.
- **If C is chosen:** `output/agentic-os/` — GovernanceKernel, A2A/MCP schemas, nine agent implementations, policy library.
- **If hybrid:** a phased build sequence (A first if regulated, B first if toolchain-led, C first if agentic-led) and an integration spec for the layered architecture.

---

*Version 1.0 — 2026-06-02 | Maintained by Governance Committee*
