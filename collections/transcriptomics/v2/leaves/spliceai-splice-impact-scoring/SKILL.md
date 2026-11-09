---
name: spliceai-splice-impact-scoring
description: Use when building a prioritized set of ClinVar VUSs predicted to alter
  splicing (acceptor/donor gain/loss) as input for tissue-suitability or RNA-seq re-inspection
  analyses such as MRSD scoring.
when_to_use_negative:
- The goal is differential expression or allele-specific expression assessment rather
  than predicted splice-altering variants — MRSD-based downstream analysis is validated
  only for splice junction coverage
- Variants already have established pathogenic/benign clinical significance — the
  filter targets 'Uncertain significance' and 'Conflicting interpretations of pathogenicity'
  records only
- A different SpliceAI version or threshold is required by the downstream protocol
  — the 7,507-variant count is specific to SpliceAI v1.2.1 and the 0.5 cutoff
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_3320
- http://edamontology.org/topic_0625
tools:
- name: SpliceAI
  role: Deep-learning delta-score annotation of acceptor/donor gain/loss used to identify
    splice-impacting VUSs at a 0.5 threshold
- name: ClinVar
  role: Source database of annotated variants; January 2021 variant summary provides
    clinical significance labels for VUS filtering
- name: Python
  role: Parsing the ClinVar variant summary, applying clinical-significance and SpliceAI-score
    filters, and writing the tab-separated output table
- name: MRSD
  role: Downstream statistical framework consuming the filtered splice-impacting VUS
    set to assess RNA-seq depth/tissue suitability
  repo: github.com/mcgm-mrsd/mrsd-explorer
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ajhg.2021.12.014
    title: 'MRSD: A quantitative approach for assessing suitability of RNA-seq in
      the investigation of mis-splicing in Mendelian disease'
  source_content_hash: sha256:8c46ab844cb18d480c292dc27996b88971dead134a4c1de58edbded808ecc187
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ClinVar
  - MRSD
  - Python
  - SpliceAI
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_3320
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# spliceai-splice-impact-scoring

## Summary

Curation of a set of splice-impacting variants of uncertain significance (VUSs) by annotating ClinVar records with SpliceAI v1.2.1 delta scores and filtering at a score >= 0.5 threshold, yielding a defined variant table for downstream RNA-seq suitability (MRSD) analysis.

## When to use

Use when building a prioritized set of ClinVar VUSs predicted to alter splicing (acceptor/donor gain/loss) as input for tissue-suitability or RNA-seq re-inspection analyses such as MRSD scoring.

## When NOT to use

- The goal is differential expression or allele-specific expression assessment rather than predicted splice-altering variants — MRSD-based downstream analysis is validated only for splice junction coverage
- Variants already have established pathogenic/benign clinical significance — the filter targets 'Uncertain significance' and 'Conflicting interpretations of pathogenicity' records only
- A different SpliceAI version or threshold is required by the downstream protocol — the 7,507-variant count is specific to SpliceAI v1.2.1 and the 0.5 cutoff

## Inputs

- ClinVar variant summary listing (January 2021 release; 352,011 annotated variants)
- SpliceAI v1.2.1 delta score annotations (acceptor/donor gain/loss) per variant

## Outputs

- Tab-separated table of 7,507 splice-impacting VUSs (SpliceAI score >= 0.5) for downstream MRSD tissue-suitability analysis
- Intermediate set of 185,119 VUS records filtered on clinical significance annotations

## How to apply

Download the January 2021 ClinVar variant summary listing (352,011 annotated variants) and, using Python, parse the summary to retain records with clinical significance 'Uncertain significance' or 'Conflicting interpretations of pathogenicity', yielding 185,119 VUS records (52.6% of the total). Annotate each retained variant with SpliceAI v1.2.1 delta scores for predicted splice-altering effects (acceptor gain/loss and donor gain/loss). Filter the annotated variants at a SpliceAI score >= 0.5 threshold, which retains 7,507 (2.1% of the original variants) splice-impacting VUSs. Write the filtered set to a tab-separated table so it can serve as the variant input for downstream MRSD tissue-suitability analysis.

## Related tools

- **SpliceAI** (Deep-learning delta-score annotation of acceptor/donor gain/loss used to identify splice-impacting VUSs at a 0.5 threshold)
- **ClinVar** (Source database of annotated variants; January 2021 variant summary provides clinical significance labels for VUS filtering)
- **Python** (Parsing the ClinVar variant summary, applying clinical-significance and SpliceAI-score filters, and writing the tab-separated output table)
- **MRSD** (Downstream statistical framework consuming the filtered splice-impacting VUS set to assess RNA-seq depth/tissue suitability) — github.com/mcgm-mrsd/mrsd-explorer

## Examples

```
python process_clinvar_spliceai.py --clinvar variant_summary_2021-01.txt --spliceai_threshold 0.5 --output splice_impacting_vus.tsv  # must yield 7,507 rows
```

## Evaluation signals

- The intermediate VUS filter must retain exactly 185,119 records (52.6% of 352,011 ClinVar variants)
- The final tab-separated table must contain exactly 7,507 variant rows, matching the paper's reported count of splice-impacting VUSs
- Every row in the output table must carry a SpliceAI v1.2.1 delta score >= 0.5
- Every row must originate from a ClinVar record annotated 'Uncertain significance' or 'Conflicting interpretations of pathogenicity'
- The output must be a valid tab-separated table compatible with downstream MRSD tissue-suitability analysis

## Limitations

- The 7,507-variant count is contingent on the January 2021 ClinVar release; other releases will yield different VUS and splice-impacting counts
- SpliceAI is an in silico prioritization tool and prediction discordance between splice tools is a known limitation for identifying pathogenic splice-impacting variants
- SpliceAI scoring only flags predicted acceptor/donor gain/loss; deep intronic or other splicing mechanisms may not be captured at the 0.5 threshold

## Evidence

- [other] From 352,011 ClinVar variants, 185,119 (52.6%) were identified as VUS, and 7,507 (2.1%) were retained as splice-impacting VUSs after SpliceAI-score-based filtering for downstream MRSD analysis.: [evidence span withheld — anchor status: not_found]
- [other] SpliceAI v1.2.1 delta scores annotation step: [evidence span withheld — anchor status: not_found]
- [other] Filter threshold definition: [evidence span withheld — anchor status: not_found]
- [other] Clinical significance filter step: [evidence span withheld — anchor status: not_found]
- [abstract] Article context on ClinVar VUS splice-impacting variants: "up to 67.8% of the variants of uncertain significance in ClinVar that are predicted to impact splicing could be assayed by RNA sequencing"
