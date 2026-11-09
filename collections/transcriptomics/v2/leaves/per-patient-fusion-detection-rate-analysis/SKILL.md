---
name: per-patient-fusion-detection-rate-analysis
description: Use when a targeted fusion search has produced per-cell fusion calls
  across a multi-patient clinical cohort and you need per-patient recovery rates and
  cohort-level sensitivity to validate the calls.
when_to_use_negative:
- Input is single-patient or single-sample data with no cohort structure for per-patient
  rate aggregation
- Fusion calls come from de novo tools without a patient/translocation-status annotation
  to validate enrichment in carriers
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: Anchored-fusion
  role: Produces the per-cell NSD2-IGH fusion calls that are aggregated per patient
  repo: https://github.com/ShenLab-Genomics/Anchored-Fusion
- name: BLAT
  role: Resolves precise fusion breakpoints on candidate chimeric reads to confirm
    the aggregated calls
- name: BWA-MEM
  role: Maps reads to the anchored gene reference with unmapped mates aligned to the
    whole genome, feeding the per-cell calls
- name: scFusion
  role: scRNA-seq competing method whose calls can be compared against per-patient
    Anchored-fusion rates
  repo: https://github.com/XiDsLab/scFusion
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100733
    title: Anchored-fusion enables targeted fusion search in bulk and single-cell
      RNA sequencing data
  source_content_hash: sha256:49552ddd27c4f372f863d300d08b026b8c79fe22b3b56b7a6321b76548df3b9d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ShenLab-Genomics/Anchored-Fusion
  repo_url: https://github.com/ShenLab-Genomics/Anchored-Fusion
  tools:
  - Anchored-fusion
  - Arriba
  - BLAT
  - BWA-MEM
  - FusionCatcher
  - FusionInspector
  - STAR-Fusion
  - scFusion
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# per-patient-fusion-detection-rate-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aggregates per-cell fusion calls (e.g., NSD2-IGH from Anchored-fusion on scRNA-seq) across a clinical cohort to compute per-patient detection rates and overall sensitivity, as done for 597 cells from 15 multiple myeloma patients.

## When to use

Use when a targeted fusion search has produced per-cell fusion calls across a multi-patient clinical cohort and you need per-patient recovery rates and cohort-level sensitivity to validate the calls.

## When NOT to use

- Input is single-patient or single-sample data with no cohort structure for per-patient rate aggregation
- Fusion calls come from de novo tools without a patient/translocation-status annotation to validate enrichment in carriers

## Inputs

- Per-cell fusion calls from Anchored-fusion (NSD2-IGH-positive cells) across the cohort
- Cell-to-patient metadata mapping (e.g., 597 cells from 15 MM patients, SRA accession SRP158590)
- Patient translocation status annotations (e.g., t(4;14) carriers RRMM1, RRMM2, SMM0)
- Resolved fusion breakpoints with spanning/split read support (e.g., chr4:1,902,353 and chr4:1,905,943)

## Outputs

- Per-patient detection rates (fraction of fusion-positive cells recovered per patient)
- Total count of fusion-positive cells per patient and cohort-wide (e.g., 99 NSD2-IGH-positive cells)
- Cohort-level sensitivity (e.g., 92.5% of NSD2-IGH-positive cells; 60.3% of all cells detected)

## How to apply

Collect the per-cell fusion calls (cells positive for the anchored fusion, e.g., NSD2-IGH) across all cells and patients in the cohort, then group calls by patient to count positive cells per patient. Compute per-patient detection rate as the number of fusion-positive cells divided by the total number of positive cells (or all cells) per patient — in the MM cohort this yielded 32.6%, 59.5%, and 77.6% for patients RRMM1, RRMM2, and SMM0. Cross-check that positives map exclusively to patients carrying the cognate translocation (t(4;14) here): all 99 NSD2-IGH-positive cells derived from the three t(4;14) patients, and Anchored-fusion detected 92.5% of all NSD2-IGH-positive cells (60.3% of all cells) in the cohort of 597 cells and 15 patients. Validate breakpoints (chr4:1,902,353 and chr4:1,905,943) via BLAT-resolved spanning and split read support to confirm the aggregate calls represent a single genuine fusion event.

## Related tools

- **Anchored-fusion** (Produces the per-cell NSD2-IGH fusion calls that are aggregated per patient) — https://github.com/ShenLab-Genomics/Anchored-Fusion
- **BLAT** (Resolves precise fusion breakpoints on candidate chimeric reads to confirm the aggregated calls)
- **BWA-MEM** (Maps reads to the anchored gene reference with unmapped mates aligned to the whole genome, feeding the per-cell calls)
- **scFusion** (scRNA-seq competing method whose calls can be compared against per-patient Anchored-fusion rates) — https://github.com/XiDsLab/scFusion

## Examples

```
python Anchored_Fusion.py --file_anchored_cds=NSD2_transcript.fasta --fastq1=cell_1.fastq.gz --fastq2=cell_2.fastq.gz --out_folder=output_dir --file_ref_seq=hg38.fa --file_ref_ann=annotation.gtf
```

## Evaluation signals

- All fusion-positive cells derive exclusively from patients carrying the cognate translocation (t(4;14) patients RRMM1, RRMM2, SMM0; zero positives in the other 12 patients)
- Per-patient detection rates fall within expected reported ranges (32.6%, 59.5%, 77.6% for RRMM1, RRMM2, SMM0)
- Cohort totals reproduce 99 positive cells and 92.5% of all NSD2-IGH-positive cells across 597 cells / 15 patients
- Breakpoints resolve to the same loci (chr4:1,902,353 and chr4:1,905,943) across patients, consistent with a shared t(4;14) breakpoint

## Limitations

- Detection rate varies widely per patient (32.6%–77.6%), so cohort-level sensitivity should not be extrapolated to any single patient without sufficient cell counts
- Anchored-fusion only detects fusions that include the user-provided target gene; per-patient rates apply only to the anchored fusion
- Anchored-fusion requires paired-end RNA-seq and cannot be applied to single-end or whole-genome/exome data

## Evidence

- [results] Anchored-fusion identified 99 single cells with the NSD2-IGH fusion gene in the MM dataset: [evidence span withheld — anchor status: not_found]
- [discussion] Aggregate across 597 cells and 15 patients: [evidence span withheld — anchor status: not_found]
- [results] Two breakpoints on chromosome 4 resolve a shared event: [evidence span withheld — anchor status: not_found]
