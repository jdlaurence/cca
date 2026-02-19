### Project name
**CCA: Continuity Care Assistant**

### Team
J.D. Laurence-Chasen — Research scientist and software developer. Solo contributor responsible for clinical problem identification, system design, implementation, and evaluation.

### Problem statement

Primary care providers (PCPs) face an impossible information synthesis challenge. With an average of just 13.5 minutes per patient visit, they must make sense of 100+ pages of EHR data — clinical notes, imaging reports, lab results, and medication lists — often accumulated across years of care by multiple providers [ref]. Critical connections get missed: a hemoglobin that has been quietly declining over three visits, a lung nodule flagged for follow-up CT that was never ordered, or an NSAID prescribed by a specialist for a patient with documented chronic kidney disease.

These are not errors of competence. They are errors of information overload.

Direct quotes from interviewed primary care providers underscore this gap:

*"There is just so much data in [a patient's] chart. There's no way to review all of it before an appointment — important connections can be missed."* — M.L.W., M.D., University of Washington

Most existing clinical AI tools are either too narrowly scoped — a single-image classifier here, a lab flag there - or are not ready to integrate into existing clinical workflows. They don't take the patient-centered, longitudinal view that defines good primary care. What PCPs need is not another isolated alert, but a system that reads the whole chart the way they wish they had time to, connecting dots across visits, modalities, and time, in a way that seamlessly integrates into the EMR.

### Overall solution

CCA is a multimodal "Don't Miss" alert system that gives primary care physicians a trustworthy second look over their patients' charts before every visit. Powered by MedGemma 1.5 4B, it operates in two stages:

**Stage 1 — Extraction.** MedGemma processes each data source in the patient's record: clinical notes are parsed for symptoms, exam findings, and outstanding follow-ups; medical images (chest X-rays, CT, MRI) are analyzed for abnormalities; and lab values are trend-analyzed deterministically. Crucially, when CCA detects multiple images of the same modality — say, two chest X-rays taken a year apart — it automatically feeds them together into MedGemma for longitudinal comparison, surfacing new findings, progression, or regression that might otherwise be invisible to a physician who only sees the most recent report.

**Stage 2 — Synthesis.** The extracted findings are aggregated into a structured clinical context and passed back to MedGemma, which synthesizes cross-modal patterns into prioritized, evidence-backed alerts. A set of deterministic safety rules (e.g., NSAID use in CKD) runs in parallel to catch clear-cut contraindications with guaranteed precision.

The result is a concise set of actionable alerts — each with a severity level, a plain-language explanation, supporting evidence, and recommended next steps — designed to integrate directly into an EMR dashboard.

**What makes CCA novel** is not chart summarization per se, but the continuity-of-care framing. By leveraging MedGemma 1.5's new longitudinal imaging capability, CCA connects findings across time in a way that mirrors how a PCP *thinks* about their patients — not visit by visit, but as an evolving clinical story.

### Technical details

**Architecture.** CCA is built as a two-stage pipeline, both stages powered by MedGemma 1.5 4B (instruction-tuned):

| Stage | Input | MedGemma Role | Output |
|-------|-------|---------------|--------|
| 1a | Clinical notes | Structured extraction via prompted JSON | Symptoms, findings, follow-ups, red flags |
| 1b | Medical images | Single-image analysis + multi-image longitudinal comparison | Findings with anatomical localization; temporal change detection |
| 1c | Lab values | Deterministic (no model needed) | Trends, abnormality flags, clinical patterns (e.g., iron deficiency) |
| 2 | Aggregated context from Stage 1 | Cross-modal synthesis | Prioritized "Don't Miss" alerts with evidence chains |

**Why MedGemma 1.5 4B?** A single 4B-parameter model handles both extraction and synthesis — keeping the system lightweight enough for on-premise deployment while still delivering strong clinical reasoning. MedGemma 1.5's native support for longitudinal chest X-ray comparison is central to CCA's continuity-of-care approach and differentiates it from pipelines that treat each image in isolation.

**Hybrid AI + deterministic design.** For safety-critical rules with no clinical ambiguity (NSAID + CKD, for example), CCA uses deterministic checks that guarantee 100% recall. MedGemma handles the nuanced, cross-modal reasoning that requires contextual understanding. This layered approach reflects how clinical decision support should work in practice: hard rules for hard contraindications, AI for pattern recognition.

**Evaluation.** Three synthetic patient scenarios — designed in consultation with practicing PCPs — demonstrate CCA's core alert types:
- *Maria Klein (58F):* Progressive iron deficiency anemia with weight loss and GI symptoms accumulating subtly over 8 months — a slow-building pattern suggestive of GI malignancy.
- *James Morrison (64M):* A left hilar mass found incidentally on pre-op CXR with no follow-up CT ever completed. CCA's longitudinal comparison confirms the mass is new by comparing against a normal CXR from 15 months prior — exactly the kind of continuity gap it is designed to close.
- *Dorothy Williams (72F):* NSAID prescribed by orthopedics despite documented CKD Stage 3a, caught by deterministic safety rules with supporting context from MedGemma about accelerating renal decline.

Pipeline outputs are evaluated against clinician-verified ground truth using two complementary checks: (1) a **severity threshold** test confirming the model flags each concern at or above the expected severity level, and (2) an **LLM-as-judge** assessment in which MedGemma itself compares the generated alert against the reference, returning a binary YES/NO verdict on whether the core clinical concern was captured without critical omissions.

**Deployment considerations.** At 4B parameters, MedGemma 1.5 can run on a single GPU — making on-premise deployment in hospital data centers feasible without sending patient data to external APIs. The modular pipeline design also allows individual stages to be swapped or extended: future versions could incorporate agentic dispatch (e.g., routing suspicious imaging findings to a specialized model), PDF/chart ingestion, or medication reconciliation by cross-referencing note mentions against the active medication list.

**Code and reproducibility.** The full implementation is available as a Kaggle notebook with mock mode for CPU-only environments and live MedGemma inference on GPU. All synthetic patient data, ground truth annotations, and evaluation code are included.
