---
name: tabular-variant-data-processing
description: Use when a tabular variant annotation file (e.g., ClinVar variant summary
  listing) must be filtered by clinical significance category and in silico splicing-prediction
  scores to build a curated variant set for RNA-based assay-suitability analysis.
when_to_use_negative:
- The variant set is already restricted to pathogenic/likely-pathogenic ClinVar records
  — the filter targets 'Uncertain significance' and 'Conflicting interpretations of
  pathogenicity' records only.
- The analysis goal is assessing overall sequencing-depth suitability (MRSD scoring
  of biosamples) rather than curating a VUS input cohort — use MRSDexplorer directly
  on junction-count and seq-depths files instead.
- Variants lack SpliceAI v1.2.1 annotations and cannot be scored, since the >= 0.5
  threshold cannot be applied without delta scores.
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3320
tools:
- name: Python
  role: Parsing the ClinVar variant summary, filtering by clinical significance, annotating
    with SpliceAI scores, and applying the >= 0.5 threshold
- name: SpliceAI v1.2.1
  role: Source of delta scores (acceptor/donor gain/loss) used to classify VUSs as
    splice-impacting at the 0.5 threshold
- name: ClinVar
  role: Source database of annotated variant records (352,011 variants in the January
    2021 summary listing)
- name: MRSD
  role: Downstream statistical framework to which the curated splice-impacting VUS
    set is fed for tissue-suitability analysis
  repo: https://github.com/mcgm-mrsd/mrsd-explorer
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ajhg.2021.12.014
    title: 'MRSD: A quantitative approach for assessing suitability of RNA-seq in
      the investigation of mis-splicing in Mendelian disease'
  source_content_hash: sha256:36b2fcc134fc3a18eb8aeb2a508bf07d12a4392b1c3335b5184f06f0b58fd918
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgm-mrsd/mrsd-explorer
  tools:
  - ClinVar
  - MRSD
  - Python
  - SpliceAI v1.2.1
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# tabular-variant-data-processing

## Summary

A Python-based curation workflow that parses a large ClinVar variant summary table, filters records to variants of uncertain significance (VUS), annotates them with SpliceAI v1.2.1 delta scores, and thresholds at SpliceAI score >= 0.5 to yield a splice-impacting VUS set for downstream RNA-seq suitability analysis (MRSD).

## When to use

Use when a tabular variant annotation file (e.g., ClinVar variant summary listing) must be filtered by clinical significance category and in silico splicing-prediction scores to build a curated variant set for RNA-based assay-suitability analysis.

## When NOT to use

- The variant set is already restricted to pathogenic/likely-pathogenic ClinVar records — the filter targets 'Uncertain significance' and 'Conflicting interpretations of pathogenicity' records only.
- The analysis goal is assessing overall sequencing-depth suitability (MRSD scoring of biosamples) rather than curating a VUS input cohort — use MRSDexplorer directly on junction-count and seq-depths files instead.
- Variants lack SpliceAI v1.2.1 annotations and cannot be scored, since the >= 0.5 threshold cannot be applied without delta scores.

## Inputs

- ClinVar variant summary listing (tab-separated, January 2021 release; 352,011 annotated variant records)
- SpliceAI v1.2.1 delta scores for each retained variant (acceptor/donor gain/loss predictions)

## Outputs

- Tab-separated table of 7,507 splice-impacting VUSs (SpliceAI score >= 0.5) for downstream MRSD tissue-suitability analysis

## How to apply

Download the January 2021 ClinVar variant summary listing (352,011 annotated variants) and parse it in Python. Filter records to those with clinical significance 'Uncertain significance' or 'Conflicting interpretations of pathogenicity', retaining 185,119 VUS records (52.6% of the input). Annotate each retained variant with SpliceAI v1.2.1 delta scores for predicted splice-altering effects (acceptor gain/loss, donor gain/loss). Apply a SpliceAI score >= 0.5 threshold to retain splice-impacting VUSs, writing the resulting set (7,507 variants, 2.1% of the original table) to a tab-separated table for downstream MRSD tissue-suitability analysis. The rationale is that SpliceAI-predicted splice-impacting VUSs are the variant class for which RNA-seq re-inspection guided by MRSD scores can clarify pathogenicity.

## Related tools

- **Python** (Parsing the ClinVar variant summary, filtering by clinical significance, annotating with SpliceAI scores, and applying the >= 0.5 threshold)
- **SpliceAI v1.2.1** (Source of delta scores (acceptor/donor gain/loss) used to classify VUSs as splice-impacting at the 0.5 threshold)
- **ClinVar** (Source database of annotated variant records (352,011 variants in the January 2021 summary listing))
- **MRSD** (Downstream statistical framework to which the curated splice-impacting VUS set is fed for tissue-suitability analysis) — https://github.com/mcgm-mrsd/mrsd-explorer

## Examples

```
python process_cummings_output.py output_list output_prefix  # cf. Python tabular parsing; VUS filter: df[df['ClinicalSignificance'].isin(['Uncertain significance','Conflicting interpretations of pathogenicity'])] then SpliceAI >= 0.5
```

## Evaluation signals

- Row-count check at each stage: 352,011 input records → 185,119 VUS (52.6%) → 7,507 splice-impacting VUSs (2.1%).
- The final output table must contain exactly 7,507 variant rows, matching the paper's reported count of splice-impacting VUSs.
- Every retained VUS record must carry clinical significance 'Uncertain significance' or 'Conflicting interpretations of pathogenicity'.
- Every output row must have a SpliceAI v1.2.1 delta score >= 0.5 for at least one acceptor/donor gain/loss class.
- Output file must be a tab-separated table compatible with the downstream MRSD tissue-suitability analysis pipeline.

## Limitations

- The curated set depends on the January 2021 ClinVar release; counts (185,119 VUS; 7,507 splice-impacting) will shift with newer database versions.
- SpliceAI score >= 0.5 is an in silico prioritization threshold; the article notes there is 'little consensus regarding the best approach to filter true positive and pathogenic events from neutral or artifactual findings'.
- Up to 67.8% of these splice-impacting VUSs can be assayed by RNA-seq in at least one of the four biosamples (whole blood, LCLs, skeletal muscle, fibroblasts) — the curation does not guarantee RNA-seq suitability for all 7,507 variants.
- Pre-computed MRSD scores are most accurate for Illumina 75 bp paired-end poly-A enrichment RNA-seq workflows matching GTEx; the variant set's downstream usability is bounded by that pipeline scope.

## Evidence

- [other] From 352,011 ClinVar variants, 185,119 (52.6%) were identified as VUS, and 7,507 (2.1%) were retained as splice-impacting VUSs after SpliceAI-score-based filtering for downstream MRSD analysis.: [evidence span withheld — anchor status: not_found]
- [abstract] up to 67.8% of the variants of uncertain significance in ClinVar that are predicted to impact splicing could be assayed by RNA sequencing in at least one of the biosamples: "up to 67.8% of the variants of uncertain significance in ClinVar that are predicted to impact splicing could be assayed by RNA sequencing"
- [intro] there is little consensus regarding the best approach to filter true positive and pathogenic events from neutral or artifactual findings: "there is little consensus regarding the best approach to filter true positive and pathogenic events from neutral or artifactual findings"
- [discussion] we demonstrate through MRSD-guided re-inspection of VUSs in ClinVar that it may be possible to use RNA-seq to clarify the effect of >5,000 variants of uncertain significance: "we demonstrate through MRSD-guided re-inspection of VUSs in ClinVar that it may be possible to use RNA-seq to clarify the effect of >5,000 variants of uncertain significance"
- [readme] MRSD (Minimum Required Sequencing Depth) is a statistical framework to predict the depth of RNA sequencing required to achieve a user-specified level of coverage: "MRSD (Minimum Required Sequencing Depth) is a statistical framework to predict the depth of RNA sequencing required to achieve a user-specified level of coverage"
