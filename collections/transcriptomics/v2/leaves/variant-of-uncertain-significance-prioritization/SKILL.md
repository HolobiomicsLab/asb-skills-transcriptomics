---
name: variant-of-uncertain-significance-prioritization
description: Use when preparing a ClinVar-derived VUS input set for RNA-seq or MRSD-based
  analysis of splice-altering variants, or when reproducing the paper's 352,011 ->
  185,119 -> 7,507 variant curation workflow.
when_to_use_negative:
- The variant set is already classified as pathogenic or likely pathogenic in ClinVar
  — this skill intentionally restricts to 'Uncertain significance' and 'Conflicting
  interpretations of pathogenicity' records.
- The downstream question concerns differential gene expression or allele-specific
  expression rather than splice junction coverage — the paper states the MRSD model
  cannot directly be extended to these use cases.
- The RNA-seq data are long-read, single-cell, or spatially resolved — the paper notes
  no appropriate control datasets exist yet to develop corresponding MRSD models for
  these modalities.
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: SpliceAI
  role: Annotates retained ClinVar VUS records with v1.2.1 delta scores used to filter
    for splice-altering effects at a >= 0.5 threshold
- name: MRSDexplorer
  role: Consumes the curated splice-impacting VUS set in the downstream MRSD tissue-suitability
    scoring of genes/transcripts
  repo: https://github.com/mcgm-mrsd/mrsd-explorer
- name: ClinVar
  role: Source database whose January 2021 variant summary listing provides the 352,011
    annotated variants from which VUSs are extracted
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ajhg.2021.12.014
    title: 'MRSD: A quantitative approach for assessing suitability of RNA-seq in
      the investigation of mis-splicing in Mendelian disease'
  source_content_hash: sha256:8dfffd2eb167a3f0bcb71ca084ce699c76f1bcb3975641c28c208ef16cbb19ce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgm-mrsd/mrsd-explorer
  tools:
  - ClinVar
  - MRSDexplorer
  - Python
  - SpliceAI
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# variant-of-uncertain-significance-prioritization

## Summary

A curation skill that extracts ClinVar variants of uncertain significance (VUS) predicted to impact splicing via SpliceAI delta-score filtering (score >= 0.5), producing a defined splice-impacting VUS set for downstream MRSD tissue-suitability analysis.

## When to use

Use when preparing a ClinVar-derived VUS input set for RNA-seq or MRSD-based analysis of splice-altering variants, or when reproducing the paper's 352,011 -> 185,119 -> 7,507 variant curation workflow.

## When NOT to use

- The variant set is already classified as pathogenic or likely pathogenic in ClinVar — this skill intentionally restricts to 'Uncertain significance' and 'Conflicting interpretations of pathogenicity' records.
- The downstream question concerns differential gene expression or allele-specific expression rather than splice junction coverage — the paper states the MRSD model cannot directly be extended to these use cases.
- The RNA-seq data are long-read, single-cell, or spatially resolved — the paper notes no appropriate control datasets exist yet to develop corresponding MRSD models for these modalities.

## Inputs

- January 2021 ClinVar variant summary listing (tab-separated, 352,011 annotated variants)
- SpliceAI v1.2.1 per-variant delta scores (acceptor/donor gain/loss)

## Outputs

- Tab-separated table of 7,507 splice-impacting VUSs for downstream MRSD tissue-suitability analysis

## How to apply

Download the January 2021 ClinVar variant summary listing (352,011 annotated variants) and parse it with Python, retaining only records whose clinical significance is 'Uncertain significance' or 'Conflicting interpretations of pathogenicity', which yields 185,119 VUS records (52.6% of the input). Annotate each retained VUS with SpliceAI v1.2.1 delta scores for predicted splice-altering effects (acceptor/donor gain/loss). Filter at a SpliceAI score >= 0.5 threshold to retain splice-impacting VUSs, then write the resulting 7,507 variants (2.1% of the original ClinVar set) to a tab-separated table that feeds the MRSD tissue-suitability analysis. This curation directly enables the paper's finding that up to 67.8% of splice-impacting ClinVar VUSs could be assayed by RNA sequencing in at least one biosample.

## Related tools

- **SpliceAI** (Annotates retained ClinVar VUS records with v1.2.1 delta scores used to filter for splice-altering effects at a >= 0.5 threshold)
- **MRSDexplorer** (Consumes the curated splice-impacting VUS set in the downstream MRSD tissue-suitability scoring of genes/transcripts) — https://github.com/mcgm-mrsd/mrsd-explorer
- **ClinVar** (Source database whose January 2021 variant summary listing provides the 352,011 annotated variants from which VUSs are extracted)

## Examples

```
vus = df[df.clinical_significance.isin(['Uncertain significance','Conflicting interpretations of pathogenicity'])]; splice_vus = vus[vus.SpliceAI_delta >= 0.5]
```

## Evaluation signals

- The filtered output table contains exactly 7,507 variant rows, matching the paper's reported count of splice-impacting VUSs.
- The intermediate VUS set contains 185,119 records (52.6% of the 352,011 ClinVar variants), and every record's clinical significance string is 'Uncertain significance' or 'Conflicting interpretations of pathogenicity'.
- Every retained variant has an associated SpliceAI delta score >= 0.5; no variant below the threshold appears in the output table.
- The output is a well-formed tab-separated table with per-variant identifiers compatible with the MRSD gene/transcript cross-referencing step.

## Limitations

- Counts are tied to the January 2021 ClinVar snapshot and SpliceAI v1.2.1; newer database versions or annotation tools will not reproduce the exact 185,119/7,507 figures.
- The SpliceAI >= 0.5 cutoff is a hard in-silico prioritization threshold; the paper notes there is little consensus on the best approach to filter true positive and pathogenic splicing events from neutral or artifactual findings.
- The curated set prioritizes splice-impacting effects only; the paper notes MRSD predictions may not accurately reflect coverage for transcripts in disorders with widespread transcriptomic changes (e.g., interferonopathies, spliceosome disruption).

## Evidence

- [other] SpliceAI >= 0.5 retention count: [evidence span withheld — anchor status: not_found]
- [other] VUS clinical significance filter: [evidence span withheld — anchor status: not_found]
- [abstract] Up to 67.8% ClinVar VUS assayable: "up to 67.8% of the variants of uncertain significance in ClinVar that are predicted to impact splicing could be assayed by RNA sequencing in at least one of the biosamples"
- [intro] Filtering consensus caveat: "there is little consensus regarding the best approach to filter true positive and pathogenic events from neutral or artifactual findings"
- [discussion] MRSD-guided VUS re-inspection: "we demonstrate through MRSD-guided re-inspection of VUSs in ClinVar that it may be possible to use RNA-seq to clarify the effect of >5,000 variants of uncertain significance"
- [readme] Transcript model assignment basis: "A single transcript model for each gene in the GENCODE human v19 annotation has been assigned according to a hierarchical approach"
