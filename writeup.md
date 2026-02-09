### Project name [A concise name for your project.] 
CCA: Continuity Care Assistant

### Your team [Name your team members, their speciality and the role they played.] 
J.D. Laurence-Chasen. 

### Problem statement [Your answer to the “Problem domain” & “Impact potential” criteria] 
Primary care physicians have an average of 13.5 minutes per patient visit but need to synthesize information from imaging, tests, and average of XX+ EHR pages per patient. Critical connections between imaging findings, symptoms mentioned months ago, and long-term lab trends are frequently missed due to the sheer volume of data and the extreme time constraints place on doctors.

Direct quotes from interviewed primary care providers support this dire need:
1) ('There is just so much data in [a patient's] chart. There's no way to review all of it before an appointment.') [M.L.W, M.D., University of Washington]
2) (Quote) [A.J.R. M.D., Oregon Health and Sciences University]
3) (Quote) [A.S., D.O., University of Washington]

Impact potential sentence or two: xxx

### Overall solution: [Your answer to “Effective use of HAI-DEF models” criterion]
I propose CCA, the Continuity Care Assistant, a multimodal system that ultimately provides primary care physicians with a trustworthy “second look” over all of the patient charts, medical images, and labs. 

### Technical details [Your answer to “Product feasibility” criterion]

--example from ML, patient with elevated platlet count, with family history of a clotting disorder, buried deep in her chart
--in medical images, stuff identified all the time that isn't germane to diagnosis; summary statement has it, but is ignored. AI could read summary statement + look at iamges
--compare past images with summary reports, flag potential findings that correlate with new stuff


---
## Technical Architecture

### Why MedGemma 1.5 4B for Both Extraction AND Synthesis?

Additionally, MedGemma 1.5 4B adds support for **longitudinal medical imaging** (chest X-ray time series), making it ideal for tracking patterns across visits.

### Deployment Considerations

| Consideration | Approach |
|---------------|----------|
| **Privacy** | All HAI-DEF models run on-premise; no PHI leaves the network |
| **Latency** | Stage 1 parallel execution; MedGemma 1.5 4B runs on single GPU |
| **Cost** | 4B model is compute-efficient—can run offline on modest hardware |
| **Reliability** | Safety rules provide deterministic fallback for critical contraindications |
| **Auditability** | Each stage produces structured outputs for review |

### Problems Addressed

| Problem | How CCA Helps |
|---------|---------------|
| **Missed diagnoses** | Surfaces slow-building patterns that span multiple visits |
| **Lost follow-ups** | Tracks recommended imaging/testing with automatic overdue alerts |
| **Fragmented care** | Catches drug-disease interactions when multiple providers prescribe |
| **Information overload** | Synthesizes 100+ pages into prioritized, actionable alerts |

### Deployment Considerations

- **EHR Integration**: Designed as a sidebar widget, not a replacement for existing workflows
- **Alert fatigue**: High-precision rules + severity levels prevent over-alerting
- **Privacy**: Open-weight models run on-premise; no PHI leaves the network
- **Physician trust**: Evidence chains let clinicians verify the reasoning
---

! Note that other models could be used in this workflow--it's the overall flow that's important. MedGemma 1.5 is the synthesizer first and foremost. The workflow could be strengthened if MedGemma dispatched other model (i.e., agentically) if there was something suspcious on an X-ray time serioues. 