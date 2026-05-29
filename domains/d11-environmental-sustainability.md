# D11 — Environmental & Sustainability

> **Domain purpose:** Govern the environmental impact of AI systems deployed on NXP edge devices — including training energy consumption, on-device inference power budgets, hardware lifecycle management, and end-of-life data sanitization. Ensures that AI deployment decisions account for environmental costs and hardware sustainability.
>
> **Primary standards:** ISO 14001 (Environmental management), GHG Protocol Scope 3 (value chain emissions), EU AI Act Recital 27 (energy efficiency), IEEE 7010 (AI and wellbeing)
>
> **RACI owner:** Governance Committee (GC) — primary

---

## NXP-Specific Context

NXP's Neutron NPU and energy-efficient MPU architecture is specifically designed for low-power inference — this is a genuine environmental differentiator over cloud AI. However, governance must actively manage this advantage:

1. **Training vs. inference energy.** The energy cost of training a foundation model (often on cloud GPUs) can dwarf the inference savings from on-device deployment. Governance must account for the full lifecycle energy footprint, not just inference.

2. **Fleet scale effects.** A power budget overrun of 500mW per device is negligible for one device but significant for a 10,000-device fleet operating continuously. Power governance must think at fleet scale.

3. **Hardware lifecycle.** NXP edge devices have hardware lifecycles of 5-15 years in industrial, automotive, and medical contexts. End-of-life responsibilities (data erasure, component recycling) extend the governance obligation well beyond software retirement.

---

## Control Checklist

### Sub-domain 11.1: Design-Time Energy Governance (L1)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| ES-01 | **NPU power budget definition:** For each AI function, define a maximum inference power budget for the Neutron NPU (in mW or % of total device power budget). The budget must account for thermal constraints, battery life requirements (for battery-powered devices), and fleet-scale aggregate impact. Power budgets must be reviewed by the hardware team before model selection. | 🟡 | L1 | Power budget specification per AI function; hardware review sign-off |

### Sub-domain 11.2: Training Energy (L2)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| ES-02 | **Training carbon footprint estimation:** For significant training runs (fine-tuning runs > 1 GPU-hour, full pre-training runs of any scale), estimate the carbon footprint using a standard methodology (ML CO2 Impact calculator, CodeCarbon, or equivalent). Report the estimate in the model card. If the estimated footprint is material, document the justification and alternatives evaluated. | 🟡 | L2 | Carbon footprint estimate per training run; model card entry; justification if material |

### Sub-domain 11.3: Quantization Energy Governance (L4)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| ES-03 | **Inference energy validation:** After INT8/INT4 quantization, measure actual inference energy consumption on target NXP hardware and compare against the power budget defined in ES-01. If the quantized model exceeds the power budget, it must not be deployed without either a revised budget (with hardware team approval) or further optimization. | 🟡 | L4 | Inference energy measurement results; comparison vs. power budget; approval for budget revision if needed |

### Sub-domain 11.4: Deployment Energy Governance (L6)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| ES-04 | **Fleet-scale energy impact assessment:** Before fleet-wide deployment, calculate the aggregate energy consumption across the deployment fleet (per-device inference energy × duty cycle × number of devices). Report this figure to the Governance Committee. For deployments exceeding defined fleet-scale thresholds, implement energy reporting as part of operational telemetry. | 🟢 | L6 | Fleet-scale energy calculation; GC notification; reporting plan for large fleets |

### Sub-domain 11.5: Operations (L7)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| ES-05 | **Power consumption monitoring:** Monitor actual NPU power consumption in production and compare against the design budget. Persistent overruns must trigger investigation: model optimization, duty cycle adjustment, or budget revision. | 🟡 | L7 | Power monitoring configuration; alert threshold; investigation procedure for overruns |
| ES-06 | **Sleep/wake cycle governance:** For battery-powered or energy-harvesting edge devices, govern the AI system's sleep/wake cycle to minimize unnecessary inference activations. The AI system must not prevent the device from entering low-power states when not needed. | 🟡 | L7 | Power management design documentation; sleep/wake cycle test results |

### Sub-domain 11.7: Hardware Retirement (L8)

| ID | Control | Priority | Stage | Evidence Required |
|----|---------|----------|-------|-------------------|
| ES-07 | **End-of-life hardware and data management:** When NXP edge devices reach end of life: (a) securely erase all stored data before hardware disposal (see D4 DP-10 and D9 ML-17); (b) document the hardware disposal method (recycling, certified e-waste, manufacturer take-back program); (c) retain records of disposal for environmental compliance purposes. | 🔴 | L8 | Data erasure certificates; hardware disposal records; e-waste compliance documentation |

---

## Standards Cross-Reference

| Control(s) | ISO 14001 | GHG Protocol | EU AI Act | IEEE 7010 |
|------------|-----------|-------------|-----------|-----------|
| ES-01, ES-03 | §8.1 (operational control) | Scope 3, Cat. 11 | Recital 27 | §5.3 (environmental impact) |
| ES-02 | §8.1 | Scope 3, Cat. 1 | Recital 27 | §5.3 |
| ES-04, ES-05, ES-06 | §9.1 (monitoring) | Scope 3 | Art. 9 | §5.3 |
| ES-07 | §8.2 (emergency preparedness) | Scope 3, Cat. 12 | Art. 18 | — |

---

*Domain D11 | Version 1.0 — 2026-05-29*
