    # Architecture Layer / Gate / Agent Matrix

    > **Date:** 2026-06-02 | **Status:** Companion matrix to [2026-06-02-three-dominant-architectural-designs.md](2026-06-02-three-dominant-architectural-designs.md)
    >
    > **Purpose:** Single-table view of **where governance work happens** in each of the three dominant designs — A's hardware layers, B's pipeline gates, C's runtime agents — and how they map to the 12 domains, the 8 lifecycle stages, and the 18-control pre-deployment gate.

    ---

    ## 1. The Master Matrix — Structural Elements Side by Side

    | # | Design A — Trust Stack LAYER | Design B — Lifecycle Spine GATE | Design C — Agentic OS AGENT | eIQ stage | Lifecycle stage | 12-domain primary owners | Pre-deployment gate (🔴) controls enforced |
    |---|------------------------------|--------------------------------|-----------------------------|-----------|------------------|--------------------------|------------------------------------------|
    | 0 | — | **G0** Conception Gate | **Orchestrator** (intake + A2A dispatch) | eIQ Stage 0 (Environment) + Stage 2 (Model select) | L1 Conception & Design | D6, D8, D10, D12 | EU-01, EU-02, HO-01, RC-01, FS-01, DG-01, ML-01, SR-01, DP-01, LM-01 |
    | 1 | **L1** Hardware (iROM → HRoT → Secure Enclave) | **G1** Data Gate | **DataCustodian** | eIQ Stage 1 (Dataset Curator) | L2 Data | D1, D4, D5 | DG-02, DG-03, DG-04, DP-03, FB-02 |
    | 2 | **L2** Firmware & OS (measured boot, PCR extension) | **G2** Training Gate | **ModelRegistrar** + **Trainer** | eIQ Stage 2 (Model Zoo / BYOM) + Stage 3 (Train) | L3 Training & Development | D2, D3, D6, D9 | LM-03, SR-04, EU-05, ML-03, ML-04, FB-04, DG-07, FS-04, XT-02 |
    | 3 | **L3** Model & Data (signed artifacts, corpus hash) | **G3** Quantization Gate (HARD GATE #1) | **Quantizer** | eIQ Stage 4 (Neutron Converter) | L4 Compression & Quantization | D5, D9, D12 | ML-05, ML-06, ML-07, FB-06, FS-05, ES-03 |
    | 4 | (L3 continues — RAG vector DB signed image) | **G4** Validation Gate | **Validator** | eIQ Stage 5 (eIQ Portal Validate) | L5 Validation & Testing | D2, D3, D5, D7, D8, D12 | SR-06, SR-07, SR-08, FB-07, LM-06, LM-08, XT-04, XT-05, FS-07, FS-08, FS-09, HO-03, HO-04 |
    | 5 | **L4** Inference & Decision (runtime attestation + on-device ledger) | **G5** Profiling Gate | **Profiler** | eIQ Stage 6 (on-device profiling) | L5 Validation & Testing (cont.) + L6 Deployment prep | D9, D11, D12 | ES-03, ML-09, FS-06 |
    | 6 | (L4 continues — on-device XAI, kill-switch) | **G6** Deployment Gate (HARD GATE #2) | **Deployer** | eIQ Stage 7 (export to inference engine) | L6 Deployment & Fleet Management | D3, D8, D9, D11, D12 | ML-10, ML-11, ML-12, SR-09, HO-05, HO-06, ES-04 |
    | 7 | (L4+L5 — decision log → fleet sync) | **G7** Retirement Gate | **Orchestrator** (retire mode) | (eIQ decommission + fleet recall) | L8 Retirement | D1, D4, D9, D10 | DG-12, DP-10, ML-16, ML-17, RC-12 |
    | 8 | **L5** Fleet & Audit (manufacturer-signed certificates, notarised log) | (manifest ledger is the "L5" of B — it is what the gates all write to) | **SafetyAdjudicator** (cross-cutting — evaluates every consequential action against HARA, SIL, PAD, autonomy level) | Cross-stage | Cross-stage (L1–L8) | D6, D8, D12 | EU-10, HO-01, HO-02, FS-12, EU-03 (review) |
    | 9 | (cross-cutting) | (cross-cutting) | **Policy Engine** (the sole writer of the manifest — Rego/OPA) | Cross-stage | Cross-stage | D9 (enforces manifest writes for all domains) | All 18 (the gate evaluates the union) |

    > **Reading rule:** Same row = same governance *concern* addressed by structurally different means. Column tells you the *how*. Rows 0–7 follow the model lifecycle. Rows 8–9 are cross-cutting.

    ---

    ## 2. The Master Matrix — By Governance Concern (rows = concern, columns = design)

    | Governance concern | A — Trust Stack (layer + mechanism) | B — Lifecycle Spine (gate + mechanism) | C — Agentic OS (agent + mechanism) |
    |--------------------|------------------------------------|----------------------------------------|-------------------------------------|
    | **Identity & authenticity of the model** | L3 — signed model artifact (SHA-256), bound to device cert chain | G2/G3 — model registry entry + signed Neutron output | ModelRegistrar — SBOM generation + `byom.sha256_verify` MCP call |
    | **Identity & authenticity of the data** | L3 — signed RAG vector DB image; corpus hash in manifest | G1 — datasheet, PII scan, representativeness report; signed at G1 | DataCustodian — datasheet gate, PII scan, DPIA reference check |
    | **Identity & authenticity of the device** | L1+L5 — iROM/HRoT + manufacturer-signed device certificate | (out of scope; assumes device is trusted) | (out of scope; assumed via Stack A or device attestation) |
    | **Identity & authenticity of the decision** | L4 — runtime TPM-style attestation, on-device signed ledger | G6 — deployment authorisation + signed artifact; decision log via runtime | Deployer + SafetyAdjudicator — every action carries agent signature |
    | **License compliance** | L3 — license binding baked into signed model bundle | G0 — license review at conception, recorded in manifest | ModelRegistrar — `license.check_allowlist` MCP call |
    | **EU AI Act risk class** | L3+L4 — class in signed manifest, enforced at runtime (scope) | G0 — class set, used at every subsequent gate | Orchestrator + SafetyAdjudicator — class check on every action |
    | **DPIA / privacy** | L3+L4 — signed DPIA reference, anonymisation at edge | G0/G1 — DPIA assessment; G1 — PII scan; G6 — no PII in decision log | DataCustodian + SafetyAdjudicator — DPIA gate + runtime PII policy |
    | **Bias / fairness** | L3 — fairness report in signed bundle, deployed with model | G1/G3/G4 — representativeness, subgroup regression, subgroup confusion | Validator + SafetyAdjudicator — subgroup regression at validation; runtime bias monitor |
    | **Security & adversarial** | L1+L2 — HRoT, measured boot, SBOM in firmware; CVE on OTA | G0/G3/G4/G6 — threat model, adversarial probes, signed artifact | ModelRegistrar + Validator — FGSM/PGD/`garak` probe suite; SBOM MCP |
    | **Functional safety (HARA, SIL, FSE, PAD)** | L1+L2+L4 — safety MCU co-processor, its own attestation chain | G0/G3/G4/G5/G6 — HARA, safety regression, FSE sign-off, determinism, 4-party auth | Validator + SafetyAdjudicator + Deployer — HARA, SIL, FSE sign-off, PAD registry check, 4-party auth |
    | **Human oversight & kill-switch** | L1+L4 — physical kill-switch pin + safety MCU; autonomy scope in runtime | G0/G4/G6 — autonomy level, FSE sign-off, kill-switch test | Orchestrator + SafetyAdjudicator — autonomy scope check, PAD registry, human-in-the-loop trigger |
    | **Explainability** | L4 — on-device XAI, signed explanation per decision | G4 — XAI validation; G6 — model card publication | Validator + SafetyAdjudicator — XAI validation; explanation attached to every decision |
    | **Environmental / sustainability** | L1+L4 — power budget enforced in hardware; per-device metric | G5/G6 — power budget assertion; sustainability KPI at G6 | Profiler + Orchestrator — power assert + sustainability KPI dashboard |
    | **Reproducibility (MLOps)** | L3 — toolchain fingerprint, dataset hash, model version in manifest | All gates — manifest captures every parameter; reproducibility in run log | All stage workers — manifest writes at every stage; seed, framework version, dataset hash |
    | **Regulatory traceability** | L5 — audit trail, EU AI Act registration hash linked to device serial | Manifest — git-versioned YAML; audit log JSONL hash-chained | Audit log — every policy decision logged with manifest ref |
    | **Tamper detection** | L1+L2+L3 — measured boot + signed artifacts (cryptographic) | Manifest — git hash + signed deltas (log-integrity) | Manifest + audit log — hash-chained; tamper breaks the chain |
    | **Recovery from compromise** | Fleet CA revoke + re-attest | Re-run gate; restore manifest from git | Re-evaluate policy; roll back manifest version |
    | **Pre-deployment gate (18 controls)** | L2→L3 + L3→L4 transitions refuse to load if any 🔴 missing | G6 refuses to emit signed artifact if any 🔴 missing | Policy Engine refuses `deploy.export` if any 🔴 missing |

    ---

    ## 3. The Master Matrix — By Control ID (How the same control is enforced in each design)

    | Control ID | Domain | A — Trust Stack (where enforced) | B — Lifecycle Spine (where enforced) | C — Agentic OS (where enforced) |
    |------------|--------|----------------------------------|----------------------------------------|----------------------------------|
    | DG-01 | D1 | L3 — corpus registry in signed manifest | G0/G1 — registry initialised, populated at G1 | DataCustodian — `manifest.write` with datasheet |
    | DG-02 | D1 | L3 — datasheet baked into signed vector DB image | G1 — datasheet YAML required | DataCustodian — `datasheet.validate` MCP |
    | DG-03 | D1 | L3 — corpus hash in signed bundle | G1 — PII scan, representativeness report | DataCustodian — `pii_scanner.scan`, `representativeness.report` |
    | DG-04 | D1 | L3 — lineaged via signed bundles | G1 + G2 — lineage in manifest | DataCustodian — `lineage.record` |
    | DG-07 | D1 | L3 — poisoning defense signed into bundle | G2 — poisoning check at training | Trainer — `poisoning.detect` MCP |
    | DG-12 | D1 | L5 — retirement hash in audit trail | G7 — retirement record | Orchestrator (retire mode) — `retire.execute` |
    | LM-01 | D2 | L3 — license binding in signed bundle | G0 — license review | ModelRegistrar — `license.check_allowlist` |
    | LM-03 | D2 | L3 — system prompt signed in bundle | G2 — model card draft | Trainer + Orchestrator — `model_card.draft` |
    | LM-05/06 | D2 | L4 — runtime refuses prompt injection; on-device filter | G4 — `garak` prompt-injection probe | Validator — `garak.run` MCP |
    | LM-08 | D2 | L4 — output policy in runtime | G4 — output validation | Validator — `output.policy_check` |
    | SR-01 | D3 | L1 — threat model baked into firmware | G0 — threat model required | Orchestrator (intake) — `threatmodel.require` |
    | SR-04/05 | D3 | L3 — SBOM in signed bundle | G2 — SBOM generation, CVE check | ModelRegistrar — `sbom.generate` |
    | SR-06 | D3 | L4 — runtime adversarial detection | G4 — FGSM/PGD probe suite | Validator — `fgsm.run`, `pgd.run` MCP |
    | SR-09 | D3 | L3 — model artifact signature | G6 — artifact signing | Deployer — `artifact.sign` MCP |
    | DP-01 | D4 | L3 — signed DPIA reference in bundle | G0 — DPIA assessment | Orchestrator (intake) — `dpia.require` |
    | DP-03 | D4 | L4 — anonymisation at edge | G1 — minimisation at intake | DataCustodian — `minimisation.enforce` |
    | DP-10 | D4 | L5 — purge in audit trail | G7 — data purge | Orchestrator (retire) — `data.purge` |
    | FB-02 | D5 | L3 — representativeness in bundle | G1 — representativeness report | DataCustodian — `representativeness.report` |
    | FB-04 | D5 | L3 — subgroup metrics in bundle | G4 — disaggregated confusion matrix | Validator — `subgroup.confusion` |
    | FB-06 | D5 | L3 — fairness regression in bundle | G3 — subgroup regression after quantize | Quantizer — `eval.run_subgroup` |
    | EU-01 | D6 | L3+L4 — prohibited-use registry at runtime | G0 — ethics review | Orchestrator (intake) — `prohibited.check` |
    | EU-02 | D6 | L3 — class in manifest, enforced at runtime scope | G0 — class assignment | Orchestrator + SafetyAdjudicator — class check |
    | EU-10 | D6 | L5 — registry + audit trail | G0 — registry in manifest | SafetyAdjudicator — runtime registry check |
    | XT-04 | D7 | L4 — on-device XAI, signed | G4 — XAI validation | Validator — `shap.compute` |
    | XT-06 | D7 | L3 — model card in signed bundle | G6 — model card publication | Deployer — `model_card.publish` |
    | HO-01 | D8 | L4 — autonomy level enforced at runtime | G0 — autonomy classification | Orchestrator + SafetyAdjudicator — scope check |
    | HO-05 | D8 | L1+L4 — physical pin + safety MCU; attestation | G6 — kill-switch test | Deployer + SafetyAdjudicator — `killswitch.test` |
    | ML-01 | D9 | L3 — version in manifest | G0 — registry initialised | ModelRegistrar — `registry.write` |
    | ML-05/06 | D9 | L3 — regression report in bundle | G3 — three regression reports | Quantizer — `eval.run_general`, `eval.run_safety` |
    | ML-10/11 | D9 | L3 — OTA bundle signed | G6 — deployment authorisation | Deployer — `deploy.authorize` |
    | ML-12 | D9 | L5 — fleet registration | G6 — per-device registration | Deployer — `fleet.register` |
    | RC-01 | D10 | L5 — registration hash | G0 — scope determination | Orchestrator (intake) — `scope.determine` |
    | RC-09 | D10 | L5 — incident in audit trail | G6 — incident routing | SafetyAdjudicator — `incident.route` |
    | ES-01 | D11 | L1 — power budget in hardware | G5 — power assertion | Profiler — `power.assert_budget` |
    | ES-04 | D11 | L5 — per-device sustainability metric | G6 — sustainability KPI | Deployer — `sustainability.record` |
    | FS-01 | D12 | L1+L2 — HARA in safety MCU firmware | G0 — HARA required | SafetyAdjudicator — `hara.require` |
    | FS-04 | D12 | L1+L4 — fail-safe in safety MCU | G0/G4 — fail-safe spec + FSE sign-off | SafetyAdjudicator — `failsafe.verify` |
    | FS-05 | D12 | L3 — safety regression in bundle | G3 — safety regression | Quantizer — `eval.run_safety` |
    | FS-07 | D12 | L4 — safety test signed event | G4 — FSE sign-off | Validator — `fse.signoff` |
    | FS-09 | D12 | L1+L2 — independent assessment recorded | G4 — independence recorded | Validator + SafetyAdjudicator — independence check |
    | FS-12 | D12 | L4 — PAD registry loaded at boot | G0 — PAD registry + G4 — verification | SafetyAdjudicator — PAD check on every action |

    ---

    ## 4. Cross-Cutting Concerns — The Three Layers of Authority in One View

    | Concern | A — Trust Stack | B — Lifecycle Spine | C — Agentic OS |
    |---------|-----------------|---------------------|-----------------|
    | **The Manifest / Ledger** | A signed YAML embedded in the device's secure storage; every manifest write requires a new signature | A Git-versioned YAML in the project repo; the manifest is the only source of truth for all gates | A Git-versioned YAML **or** a hash-chained DB; only the **Policy Engine** can write it |
    | **The Manifest Writer** | The MRO's signing key (off-device); verified on-device at boot | The gate worker (CI runner, eIQ wrapper, OTA pipeline) — every write is signed | **The Policy Engine** (not the workers) — workers only propose deltas |
    | **The Authority Check** | Measured boot + remote attestation | Gate pre-conditions + signed deltas | `(agent_id, tool, payload, manifest_state) → decision` via OPA/Rego |
    | **Failure Mode** | "Boot fails → no AI runs" | "Gate fails → no AI ships" | "Policy blocks → no AI acts" |
    | **Recovery** | Re-attest; fleet CA key rotation | Re-run gate; restore from git | Re-evaluate policy; roll back manifest version |
    | **Tamper Evidence** | Cryptographic (PCR values, signed artifacts, hash-chained decision log) | Git history + signed deltas + hash-chained audit log | Hash-chained audit log; manifest_ref optimistic concurrency |
    | **What is Non-Bypassable** | The hardware kill-switch, the signed model loader, the measured boot | The CI gate, the gate contract, the manifest pre-conditions | The Policy Engine, the typed MCP tool registry, the signed manifest writer |
    | **18-Control Pre-Deployment Gate** | Enforced at L2→L3 + L3→L4 transitions | Enforced at G6 (the union of all gates) | Enforced by `policy.evaluate_gate` on every `deploy.export` |

    ---

    ## 5. RACI Mapping — Who Holds Which Role in Each Design

    | Role | A — Trust Stack | B — Lifecycle Spine | C — Agentic OS |
    |------|-----------------|---------------------|-----------------|
    | **AIPO (AI Product Owner)** | Signs the use-case manifest; signs the deployment authorisation | Approves Gate G0; signs the manifest at G6 | Owns the Orchestrator's `project.intent`; approves gate requests |
    | **MRO (Model Risk Officer)** | Holds the model-signing key; signs every artifact at L3 | Owns G1–G4; signs the manifest at every gate | Owns the ModelRegistrar, Trainer, Quantizer, Validator agents' signing keys |
    | **DSL (Device Security Lead)** | Owns L1+L2 attestation chain; signs fleet CA certs; signs SBOMs | Owns G2 (SBOM) and G6 (signed artifact); security exception authority | Owns the SecurityAdjudicator; sets the MCP tool registry; signs security policies |
    | **DPO (Data Protection Officer)** | Signs the DPIA reference baked into L3 bundles | Signs the DPIA at G0; data minimisation policy at G1 | Owns DataCustodian privacy policy; signs DPIA MCP call |
    | **FSE (Functional Safety Engineer)** | Owns the safety MCU firmware; signs safety attestation chain | Signs the FSE row in G4; signs the 4-party authorisation in G6 | Owns SafetyAdjudicator; signs safety-class policies; cannot be overridden except by authorised human |
    | **GC (Governance Committee)** | Approves EU AI Act class; approves PAD registry; signs fleet CA policy | Approves framework changes; approves exceptions to gates | Approves the policy library; approves autonomy-level changes; approves agent authorisations |
    | **External Auditor** | Verifies the device's attestation chain end-to-end | Reads the manifest; walks the gate history | Walks the audit log; replays policy decisions |

    ---

    ## 6. The 18-Control Pre-Deployment Gate — Where It Lives in Each Design

    The 18 controls from [01-coverage-matrix.md](../01-coverage-matrix.md) that constitute the Minimum Viable Governance pre-deployment gate. Where each is enforced in each design:

    | # | Control | Stage | A — Trust Stack | B — Lifecycle Spine | C — Agentic OS |
    |---|---------|-------|-----------------|---------------------|-----------------|
    | 1 | DG-01: Data source registry | L1 | L3 manifest | G0 init | ModelRegistrar `registry.write` |
    | 2 | LM-01: Model license review | L1 | L3 bundle signature | G0 review | ModelRegistrar `license.check_allowlist` |
    | 3 | SR-01: MITRE ATLAS threat model | L1 | L1 firmware threat | G0 model | Orchestrator `threatmodel.require` |
    | 4 | DP-01: DPIA assessment | L1 | L3 signed DPIA ref | G0 assessment | Orchestrator `dpia.require` |
    | 5 | EU-01: Ethics review gate | L1 | L3+L4 registry | G0 ethics | Orchestrator `prohibited.check` |
    | 6 | EU-02: EU AI Act risk classification | L1 | L3 manifest class | G0 classification | Orchestrator `scope.classify` |
    | 7 | HO-01: Autonomy level classification | L1 | L4 runtime scope | G0 autonomy | Orchestrator `autonomy.classify` |
    | 8 | ML-01: Model version registry | L1 | L3 manifest version | G0 init | ModelRegistrar `version.write` |
    | 9 | RC-01: Regulatory scope | L1 | L5 registration | G0 scope | Orchestrator `scope.determine` |
    | 10 | FS-01: Hazard analysis | L1 | L1+L2 HARA | G0 HARA | SafetyAdjudicator `hara.require` |
    | 11 | FB-04: Pre-deployment bias testing | L3 | L3 bundle metrics | G4 confusion matrix | Validator `subgroup.confusion` |
    | 12 | ML-05: Quantization accuracy regression | L4 | L3 regression report | G3 general regression | Quantizer `eval.run_general` |
    | 13 | ML-06: Safety property preservation | L4 | L3 safety regression | G3 safety regression | Quantizer `eval.run_safety` |
    | 14 | SR-06: Adversarial robustness | L5 | L4 runtime defense | G4 adversarial probe | Validator `fgsm.run`/`pgd.run`/`garak.run` |
    | 15 | FS-07: Functional safety review | L5 | L4 safety attestation | G4 FSE sign-off | Validator `fse.signoff` |
    | 16 | ML-10: OTA authorization chain | L6 | L3+L5 OTA signature | G6 deployment auth | Deployer `deploy.authorize` |
    | 17 | ML-11: Staged rollout plan | L6 | L5 fleet policy | G6 rollout plan | Deployer `rollout.plan` |
    | 18 | HO-05: Kill-switch operational | L6 | L1+L4 hardware test | G6 kill-switch test | Deployer + SafetyAdjudicator `killswitch.test` |

    **The single most important property across all three designs:** in every case, the 18 controls are checked *before* the deployable artifact is produced. The artifact cannot exist without the controls. The mechanism differs (signed model bundle / G6 manifest gate / `BlockedByPolicy` exception) but the property is the same.

    ---

    ## 7. The Agentic Harness (Build-Time Reference) — Where It Maps

    The `output/2026-06-01-agentic-harness-design.md` already defines a 10-agent build-time harness (Orchestrator + 9 stage workers). Each design here uses a *subset* of those agents to enforce the runtime/lifecycle concerns:

    | Harness Agent (build-time) | Primary Design | Role in Each Design |
    |----------------------------|----------------|---------------------|
    | **Orchestrator** | All three | A: external — drives the device attestation flow. B: external — drives the gates. C: **native** — the runtime orchestrator. |
    | **EnvironmentSteward** | B | Provisions the eIQ baseline (pinned toolchain) and the manifest scaffold |
    | **DataCurator** | B, C | B: G1 worker. C: DataCustodian. |
    | **ModelRegistrar** | B, C | B: G2 worker. C: ModelRegistrar. |
    | **Trainer** | B, C | B: G2 worker. C: Trainer. |
    | **QuantizationGatekeeper** | B, C | B: G3 gate. C: Quantizer (with the three regressions as MCP calls). |
    | **Validator** | B, C | B: G4 gate. C: Validator. |
    | **Profiler** | B, C | B: G5 gate. C: Profiler. |
    | **DeploymentGatekeeper** | B, C | B: G6 gate (the only signer). C: Deployer. |
    | **PipelineAssembler** | C (GenAI) | C: PipelineAssembler for GenAI Flow / Agentic pipelines. |
    | **(new) SafetyAdjudicator** | C | C only — runtime arbiter of safety decisions |
    | **(new) Policy Engine** | C | C only — the only manifest writer; the heart of the Agent OS |

    **Implication for adoption:** if you build **C**, you are essentially building the harness. If you build **B**, you are building the gates — the harness's build-time workers are the gates. If you build **A**, you are providing the device that the harness's output runs on.

    ---

    ## 8. Reading the Matrix in Practice

    ### 8.1 When the user asks "where is bias tested?"

    - **A:** L3 — the fairness report is baked into the signed model bundle. If the bundle is missing the report, the runtime refuses to load.
    - **B:** G1, G3, G4 — representativeness at intake, subgroup regression after quantization, subgroup confusion at validation.
    - **C:** DataCustodian (representativeness), Quantizer (subgroup regression), Validator (subgroup confusion), SafetyAdjudicator (runtime bias monitor).

    ### 8.2 When the user asks "what stops a non-governed model from running on the device?"

    - **A:** The signed model loader at L3 — the runtime loads nothing unsigned. The signing key is bound to the device certificate chain. The manifest is measured at boot; if the expected L1 control state doesn't match, the runtime refuses to load.
    - **B:** Out of scope — the Spine governs the *build*, not the device. A Spine-built model that has not passed G6 is unsigned; the device (Stack A) refuses to load it. Spine + Stack together close the loop.
    - **C:** The ModelRegistrar's `agents_authorized` MCP policy, plus the Deployer's `policy.evaluate_gate` call. The Policy Engine refuses `deploy.export` if any 🔴 control is incomplete.

    ### 8.3 When the user asks "what stops an LLM agent from taking a prohibited action at runtime?"

    - **A:** The prohibited-use registry is loaded at boot into the runtime. The runtime enforces PAD checks before every action.
    - **B:** The registry is in the manifest at G0, checked at every gate. Once the model is built, the Spine is not in the runtime loop.
    - **C:** The SafetyAdjudicator evaluates every consequential action against the HARA, the SIL classification, the PAD registry, and the autonomy-level scope. The Policy Engine blocks the action if PAD says so.

    ### 8.4 When the user asks "what's the cheapest governance we can ship in Q3?"

    - **A:** Most expensive. Requires secure boot, attestation infrastructure, signing keys, fleet CA.
    - **B:** Medium. Requires CI integration, manifest schema, gate scripts. Most of the rest of the framework is reusable.
    - **C:** Medium-High. Requires policy DSL, agent runtime, MCP tool registry. But many components are reusable across designs (manifest schema, audit log, policy library).

    ---

    ## 9. The One-Page Summary

    > **Design A — Trust Stack (Layers):** Authority in silicon. L1 Hardware → L2 Firmware/OS → L3 Model/Data → L4 Inference/Decision → L5 Fleet/Audit. 5 layers, 5 roots of trust.
    >
    > **Design B — Lifecycle Spine (Gates):** Authority in the build pipeline. G0 Conception → G1 Data → G2 Train → G3 Quantize (hard) → G4 Validate → G5 Profile → G6 Deploy (hard) → G7 Retire. 8 gates, 1 manifest, 1 source of truth.
    >
    > **Design C — Agentic OS (Agents):** Authority in software. Orchestrator + DataCustodian + ModelRegistrar + Trainer + Quantizer + Validator + Profiler + Deployer + **SafetyAdjudicator** + **Policy Engine**. 10 components, 1 policy.
    >
    > **All three together:** A makes the device trustworthy; B makes the artifact trustworthy; C makes the action trustworthy. The 18-control pre-deployment gate is enforced at L2→L3+L3→L4 (A), at G6 (B), and at `policy.evaluate_gate` (C) — the same property, three different mechanisms.

    ---

    *Version 1.0 — 2026-06-02 | Companion to [2026-06-02-three-dominant-architectural-designs.md](2026-06-02-three-dominant-architectural-designs.md)*
