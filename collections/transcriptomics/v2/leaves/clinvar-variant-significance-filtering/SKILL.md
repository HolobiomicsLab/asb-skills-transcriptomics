---
name: clinvar-variant-significance-filtering
description: Use when the goal is to extract a bounded, clinically interpretable set
  of predicted splice-altering variants from ClinVar as the input cohort for RNA-seq
  splicing analyses such as MRSD scoring.
when_to_use_negative:
- If the aim is to survey known pathogenic/likely-pathogenic variants rather than
  VUSs whose interpretation RNA-seq could clarify — this skill deliberately excludes
  those significance classes
- If SpliceAI (v1.2.1) splice-impact annotations are unavailable or a different splice-prediction
  tool is used, the 7,507-variant count and the MRSD-derived VUS-resolving-power results
  will not be reproducible
- If the target analysis is differential expression or allele-specific expression
  rather than splice-junction coverage, since MRSD (and hence this filtered VUS cohort)
  is validated only for bulk short-read poly-A splice-junction coverage
edam_operation: http://edamontology.org/operation_3678
edam_topics:
- http://edamontology.org/topic_3577
tools:
- name: Python
  role: Parses the ClinVar variant summary, filters records on clinical significance
    terms, annotates/merges SpliceAI v1.2.1 delta scores, applies the 0.5 threshold,
    and writes the tab-separated VUS table
- name: MRSD
  role: Downstream consumer of the filtered 7,507-variant set, used to compute gene-
    and tissue-specific minimum required sequencing depth for each VUS-containing
    gene
  repo: https://github.com/mcgm-mrsd/mrsd-explorer
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ajhg.2021.12.014
    title: 'MRSD: A quantitative approach for assessing suitability of RNA-seq in
      the investigation of mis-splicing in Mendelian disease'
  source_content_hash: sha256:c80725ae57feed0f4365d28aa60a1b40a4aaa69f0bae9c2143c08129b2d9bec6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgm-mrsd/mrsd-explorer
  tools:
  - MRSD
  - Python
  edam_operation: http://edamontology.org/operation_3678
  edam_topics:
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# clinvar-variant-significance-filtering

## Summary

Curation of a set of splice-impacting variants of uncertain significance (VUS) from the ClinVar variant summary by filtering on clinical significance terms and SpliceAI v1.2.1 delta scores, producing the 7,507-variant input used for downstream MRSD tissue-suitability analysis.

## When to use

Use when the goal is to extract a bounded, clinically interpretable set of predicted splice-altering variants from ClinVar as the input cohort for RNA-seq splicing analyses such as MRSD scoring.

## When NOT to use

- If the aim is to survey known pathogenic/likely-pathogenic variants rather than VUSs whose interpretation RNA-seq could clarify — this skill deliberately excludes those significance classes
- If SpliceAI (v1.2.1) splice-impact annotations are unavailable or a different splice-prediction tool is used, the 7,507-variant count and the MRSD-derived VUS-resolving-power results will not be reproducible
- If the target analysis is differential expression or allele-specific expression rather than splice-junction coverage, since MRSD (and hence this filtered VUS cohort) is validated only for bulk short-read poly-A splice-junction coverage

## Inputs

- ClinVar variant summary listing (January 2021; 352,011 annotated variants) with clinical significance field
- SpliceAI v1.2.1 delta score annotations for acceptor/donor gain/loss per variant

## Outputs

- Tab-separated table of 7,507 splice-impacting VUSs (one row per variant) for downstream MRSD tissue-suitability analysis
- Intermediate VUS set of 185,119 records after clinical-significance filtering

## How to apply

Download the January 2021 ClinVar variant summary listing (352,011 annotated variants) and, using Python, parse the records and retain those with clinical significance 'Uncertain significance' or 'Conflicting interpretations of pathogenicity', yielding 185,119 VUS records (52.6% of the source set). Annotate each retained variant with SpliceAI v1.2.1 delta scores for predicted splice-altering effects (acceptor/donor gain/loss), then apply a hard threshold of SpliceAI score >= 0.5 to keep splice-impacting VUSs (7,507 variants, 2.1% of the source set). Write the filtered set to a tab-separated table that feeds the MRSD tissue-suitability analysis, which in this study determined that up to 67.8% of splice-impacting ClinVar VUSs could be assayed by RNA-seq in at least one biosample (whole blood, LCLs, skeletal muscle, cultured fibroblasts). The two-stage design — significance-term filter followed by in silico splice-impact filter — restricts downstream analysis to variants where RNA-seq evidence could plausibly clarify the clinical interpretation.

## Related tools

- **Python** (Parses the ClinVar variant summary, filters records on clinical significance terms, annotates/merges SpliceAI v1.2.1 delta scores, applies the 0.5 threshold, and writes the tab-separated VUS table)
- **MRSD** (Downstream consumer of the filtered 7,507-variant set, used to compute gene- and tissue-specific minimum required sequencing depth for each VUS-containing gene) — https://github.com/mcgm-mrsd/mrsd-explorer

## Evaluation signals

- Final output table contains exactly 7,507 variant rows, matching the paper's reported splice-impacting VUS count
- Intermediate VUS filter retains exactly 185,119 records (52.6%) from the 352,011 ClinVar variants
- Retained records all carry clinical significance 'Uncertain significance' or 'Conflicting interpretations of pathogenicity' — no pathogenic/likely-pathogenic/benign classes present
- Every retained variant has a SpliceAI v1.2.1 delta score >= 0.5 (acceptor/donor gain/loss)
- Output is a tab-separated table with one row per variant, compatible with the .junc-counts/.introns.bed workflow used by MRSDexplorer

## Limitations

- The SpliceAI >= 0.5 threshold and the January 2021 ClinVar snapshot are version-locked: newer ClinVar releases or SpliceAI versions will change the 185,119 VUS and 7,507 splice-impacting counts
- Filtering relies on in silico splice prediction, and the paper notes 'discordance between in silico variant prioritization tools' as a general limitation of splice-impacting variant identification
- The filtered VUS set is only useful for biosamples/protocols matched to the MRSD validation scope (bulk short-read poly-A enrichment RNA-seq); MRSD predictions are not characterized for long-read, single-cell, or spatially resolved RNA-seq

## Evidence

- [other] From 352,011 ClinVar variants, 185,119 (52.6%) were identified as VUS, and 7,507 (2.1%) were retained as splice-impacting VUSs after SpliceAI-score-based filtering for downstream MRSD analysis.: [evidence span withheld — anchor status: not_found]
- [abstract] Abstract claim on assayable VUSs: "up to 67.8% of the variants of uncertain significance in ClinVar that are predicted to impact splicing could be assayed by RNA sequencing"
- [other] SpliceAI annotation and threshold step: [evidence span withheld — anchor status: not_found]
- [other] Significance-term filter definition: [evidence span withheld — anchor status: not_found]
