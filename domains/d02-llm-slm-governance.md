# D2 — LLM / SLM Governance

> **Domain purpose:** Govern the selection, configuration, deployment, and operation of Large Language Models (LLMs) and Small Language Models (SLMs) on NXP edge devices. Addresses model licensing, system prompt policy, adversarial prompt defense, output safety guardrails, and the full OWASP LLM Top 10 control set in constrained, often offline edge contexts.
>
> **Primary standards:** OWASP LLM Top 10 (2025), MITRE ATLAS, ISO/IEC 42001 §8, EU AI Act Art. 51-53 (GPAI)
>
> **RACI owner:** Model Risk Officer (MRO) — primary

---

## NXP-Specific Context

NXP's eIQ GenAI Flow pipeline enables on-device LLM/SLM inference — typically models in the 1B–13B parameter range quantized to INT4/INT8 for the Neutron NPU. This creates governance challenges distinct from cloud LLM deployments:

1. **Offline operation.** The model operates without cloud connectivity, meaning no runtime content moderation API can be called. All guardrails must be on-device.

2. **HMI contexts.** LLMs on NXP devices commonly power voice assistants, operator panels, or robotic command interpreters. Prompt injection via physical HMI inputs (voice, touchscreen, barcode scan) is a real attack vector that does not exist in typical web LLM deployments.

3. **Knowledge perimeter via local RAG.** The model's knowledge is bounded by the local vector DB. Governance must ensure this boundary is enforced — the model must not hallucinate facts outside the RAG corpus or accept injected context that expands the knowledge perimeter.

4. **Model update cadence.** Foundation models for edge are updated rarely (due to the quantization and HiL validation cycle in D9). This means the governance burden shifts from continuous monitoring to rigorous pre-deployment vetting.

---

## Control Checklist

### Sub-domain 2.1: Model Selection & Licensing (L1)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| LM-01 | **Model license compliance review:** Before selecting any open-source or third-party LLM/SLM for deployment, conduct a license review. Distinguish between research licenses (e.g., original Llama 1 terms), commercial-use licenses (Llama 3, Mistral Apache 2.0), and proprietary licenses. Verify license terms are compatible with the intended use case and deployment scale. | 🔴 | L1 | License review record per model candidate; legal sign-off for commercial use |
| LM-02 | **Model selection criteria documentation:** Document the criteria used to select the deployed model: safety evaluation scores, benchmark performance at target quantization level, license compatibility, NXP hardware compatibility (NPU support, memory footprint), and supplier/provenance. | 🔴 | L1 | Model selection decision record with evaluated alternatives |

### Sub-domain 2.2: System Prompt & Context Governance (L3)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| LM-03 | **System prompt policy:** Maintain a versioned system prompt policy that defines: (a) what instructions must be present in every system prompt (safety constraints, scope limitations, persona definition); (b) what is prohibited from system prompts (instructions that disable safety guardrails, grant elevated permissions, or override refusal policies); (c) who is authorized to modify system prompts and the approval process. | 🔴 | L3 | System prompt policy document; version history; approval records for prompt changes |
| LM-04 | **RAG context perimeter enforcement:** The local RAG pipeline must enforce a strict knowledge perimeter. The model must not accept externally injected context that was not sourced from the approved local vector DB. Implement and test context source verification. | 🔴 | L3 | RAG architecture design showing perimeter enforcement; penetration test results |

### Sub-domain 2.3: Prompt Injection Defense (L4/L5)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| LM-05 | **Prompt injection threat model:** Complete a threat model specifically for prompt injection vectors applicable to the device's input channels (voice input, touchscreen text, QR/barcode data, sensor telemetry interpreted as text, API inputs). Map each vector to a mitigation control. | 🔴 | L4 | Prompt injection threat model document; mitigations per vector |
| LM-06 | **Prompt injection testing:** Prior to deployment, execute a prompt injection test suite against all identified input vectors. Include direct injection, indirect injection (via RAG corpus), jailbreak attempts, and role-play escalation. Maintain a registry of tested attack patterns. | 🔴 | L5 | Prompt injection test report; attack pattern registry; remediation records for any successful attacks |

### Sub-domain 2.4: Output Safety & Guardrails (L5)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| LM-07 | **Output guardrails definition:** Define and implement on-device output guardrails covering: prohibited content categories (hazardous instructions, PII exposure, confidential system information); confidence thresholds below which the model must decline to answer; and scope enforcement (refusal of requests outside the defined use case). Guardrails must function without cloud connectivity. | 🔴 | L5 | Guardrails configuration document; test results demonstrating each guardrail fires correctly |
| LM-08 | **Hallucination detection and mitigation:** Implement controls to detect and limit hallucinations, particularly factual assertions outside the local RAG corpus. Approaches include: RAG citation enforcement (model must cite source documents), confidence scoring, and fallback to "I don't know" responses when retrieval confidence is low. | 🔴 | L5 | Hallucination testing methodology and results; citation enforcement test cases |
| LM-09 | **Refusal policy and graceful failure:** Define how the model behaves when a request is out of scope, a guardrail fires, or inference fails. Responses must not expose system internals, model architecture, or the content of the system prompt. Test refusal responses explicitly. | 🟡 | L5 | Refusal policy document; test cases for each refusal trigger; sample refusal responses |

### Sub-domain 2.5: OWASP LLM Top 10 Control Mapping (L5/L6)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| LM-10 | **OWASP LLM01 — Prompt Injection:** Controls LM-05 and LM-06 address this. Confirm all input channels have injection mitigations. | 🔴 | L5–L6 | Cross-reference to LM-05, LM-06 test reports |
| LM-11 | **OWASP LLM02 — Insecure Output Handling:** Ensure all LLM outputs are sanitized before being rendered in any downstream system (HMI display, control system, API response). Specifically: no code execution from LLM output unless explicitly intended and sandboxed; no HTML/script injection in display contexts. | 🔴 | L6 | Output handling architecture review; sanitization test cases |

### Sub-domain 2.6: Operations & Monitoring (L7)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| LM-12 | **OWASP LLM03 — Training Data Poisoning (monitoring):** Monitor for indicators that the local RAG corpus or any fine-tuning data has been tampered with. Implement integrity checks (hash verification) on the vector DB and training artifacts at startup and on a defined schedule. | 🔴 | L7 | Integrity check procedure; scheduled verification logs |
| LM-13 | **OWASP LLM06 — Sensitive Information Disclosure:** Monitor production inference logs (where available) for patterns indicating the model is exposing PII, PHI, system configurations, or other sensitive data it should not have in its context window. Alert and remediate. | 🔴 | L7 | Log monitoring configuration; alert definition; incident procedure for disclosure events |
| LM-14 | **OWASP LLM08 — Excessive Agency:** If the LLM is connected to tools, APIs, or physical actuators, define and enforce a minimum-privilege tool use policy. The model must only be able to invoke the specific tools required for its defined use case. Review and audit tool permissions quarterly. | 🟡 | L7 | Tool permission policy; quarterly audit records |

### Sub-domain 2.7: Retirement (L8)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| LM-15 | **Model retirement and knowledge base purge:** When retiring an LLM/SLM version, purge the associated system prompt configuration, RAG vector DB contents, and any cached conversation state from all devices. Retain the model card and evaluation artifacts per the D9 retirement procedure. | 🔴 | L8 | Retirement checklist completion record; purge confirmation |

---

## OWASP LLM Top 10 Full Mapping

| OWASP ID | Vulnerability | Primary Control | Stage |
|----------|--------------|-----------------|-------|
| LLM01 | Prompt Injection | LM-05, LM-06 | L4–L5 |
| LLM02 | Insecure Output Handling | LM-11 | L6 |
| LLM03 | Training Data Poisoning | LM-12 (monitoring) + D1 (DG-07 at training) | L7 |
| LLM04 | Model Denial of Service | D3 (SR-11), D9 (ML-14) | L7 |
| LLM05 | Supply Chain Vulnerabilities | D3 (SR-03, SR-04) | L3–L4 |
| LLM06 | Sensitive Information Disclosure | LM-07, LM-13 | L5, L7 |
| LLM07 | Insecure Plugin Design | LM-14 | L7 |
| LLM08 | Excessive Agency | LM-14 | L7 |
| LLM09 | Overreliance | D8 (HO-01, HO-02), LM-08 | L1, L5 |
| LLM10 | Model Theft | D3 (SR-09, SR-10) | L6 |

---

## Standards Cross-Reference

| Control(s) | OWASP LLM Top 10 | MITRE ATLAS | ISO/IEC 42001 | EU AI Act |
|------------|-----------------|-------------|--------------|-----------|
| LM-01, LM-02 | — | — | §8.3 | Art. 53 (GPAI transparency) |
| LM-03, LM-04 | LLM01 | AML.T0054 | §8.5 | Art. 9 |
| LM-05, LM-06 | LLM01 | AML.T0051 | §8.5 | Art. 15 |
| LM-07, LM-08, LM-09 | LLM02, LLM06 | AML.T0048 | §8.5 | Art. 9, Art. 15 |
| LM-10–LM-14 | LLM01–LLM08 | Multiple | §9.1 | Art. 72 |
| LM-15 | — | — | §8.3 | Art. 18 |

---

*Domain D2 | Version 1.0 — 2026-05-29*
