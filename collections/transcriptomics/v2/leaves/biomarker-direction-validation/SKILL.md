---
name: biomarker-direction-validation
description: Use when reproducing a gene-expression signature where the expected direction
  of change is known (e.g. MCEMP1 up, HLA-DRA down), and you must verify that multiple
  perturbation conditions (viral proteins vs GFP control) recapitulate that pattern
  on a majority basis.
when_to_use_negative:
- When the goal is statistical significance testing (p-values, BH adjustment) rather
  than sign-of-fold-change concordance — use the severity-vs-mild differential expression
  analysis from the COVID19-severity app instead
- When only one condition is being tested (no multiple perturbations to count a majority
  over), the fraction-based acceptance rule is not meaningful
- When the expected direction of the biomarker signature has not been established
  from prior cohorts
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Streamlit
  role: Webtool used by the authors to query and visualize the gene expression differences
    (e.g. MCEMP1, HLA-DRA) between severe and mild COVID-19 patients across the 7
    datasets being validated against
  repo: https://share.streamlit.io/kuanrongchan/covid19-severity/main/app.py
- name: pip
  role: Installs dependencies (requirements.txt) for running the COVID19-severity
    repository locally to reproduce the reference signature directions
  repo: https://github.com/kuanrongchan/COVID19-severity
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.ebiom.2023.104472
    title: Early peripheral blood MCEMP1 and HLA-DRA expression predicts COVID-19
      prognosis
  source_content_hash: sha256:e34e132c7b4c7c7c6ff35a93e6f22fa205a4d5941061b09242e27451b55e94d6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Streamlit
  - pip
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# biomarker-direction-validation

## Summary

A directional concordance check that validates whether a perturbation experiment reproduces a reported biomarker signature by counting the fraction of conditions showing the expected fold-change direction. Here, MCEMP1 log2FC must be positive and HLA-DRA log2FC negative relative to GFP control for a majority (>50%) of SARS-CoV-2 viral-protein transfection conditions.

## When to use

Use when reproducing a gene-expression signature where the expected direction of change is known (e.g. MCEMP1 up, HLA-DRA down), and you must verify that multiple perturbation conditions (viral proteins vs GFP control) recapitulate that pattern on a majority basis.

## When NOT to use

- When the goal is statistical significance testing (p-values, BH adjustment) rather than sign-of-fold-change concordance — use the severity-vs-mild differential expression analysis from the COVID19-severity app instead
- When only one condition is being tested (no multiple perturbations to count a majority over), the fraction-based acceptance rule is not meaningful
- When the expected direction of the biomarker signature has not been established from prior cohorts

## Inputs

- GSE186650 series matrix and sample metadata (GSM annotations mapping samples to transfected SARS-CoV-2 protein or GFP control)
- Expression value table with MCEMP1 and HLA-DRA rows and per-sample columns
- Condition mapping collapsing duplicate transfection replicates per protein

## Outputs

- viral_protein_log2fc.csv with one row per SARS-CoV-2 protein and MCEMP1/HLA-DRA log2 fold-change columns
- Fraction of viral proteins showing concordant MCEMP1-up/HLA-DRA-down direction (accept if >50%)

## How to apply

First compute per-gene log2 fold changes as mean expression under each viral-protein transfection condition minus mean GFP-control expression, using pandas/numpy groupby-mean on the parsed GSE186650 series matrix, collapsing duplicated transfection conditions by averaging. Assemble one row per SARS-CoV-2 protein with columns for MCEMP1 and HLA-DRA log2FC, and write to viral_protein_log2fc.csv. The validation criterion is directional, not significance-based: count the fraction of viral proteins with positive MCEMP1 log2FC AND negative HLA-DRA log2FC. Accept the reproduction only if the majority (>50%) of proteins satisfy both directions, consistent with the reported CD14+ cell dysregulation pattern. Rationale: the article established MCEMP1 and HLA-DRA as the most consistently differentially regulated genes in peripheral blood of severe COVID-19 patients across 7 transcriptomics datasets, so direction (not magnitude or p-value) is the robust reproducibility signal.

## Related tools

- **Streamlit** (Webtool used by the authors to query and visualize the gene expression differences (e.g. MCEMP1, HLA-DRA) between severe and mild COVID-19 patients across the 7 datasets being validated against) — https://share.streamlit.io/kuanrongchan/covid19-severity/main/app.py
- **pip** (Installs dependencies (requirements.txt) for running the COVID19-severity repository locally to reproduce the reference signature directions) — https://github.com/kuanrongchan/COVID19-severity

## Examples

```
fc = df.groupby('protein')[['MCEMP1','HLA-DRA']].mean(); log2fc = fc - fc.loc['GFP']; frac = ((log2fc['MCEMP1']>0)&(log2fc['HLA-DRA']<0)).mean(); accept = frac>0.5
```

## Evaluation signals

- viral_protein_log2fc.csv exists with exactly one row per unique SARS-CoV-2 protein transfection condition
- Every row has numeric MCEMP1 and HLA-DRA log2FC values computed as condition mean minus GFP-control mean
- The counted fraction of proteins with MCEMP1 log2FC > 0 and HLA-DRA log2FC < 0 exceeds 0.5
- GFP control mean expression values match the average over all GFP-transfected samples after collapsing duplicates
- Directional result is consistent with the reported pattern that MCEMP1 is heightened and HLA-DRA reduced predominantly in CD14+ cells in severe COVID-19

## Limitations

- Directional concordance ignores effect size and statistical significance; a protein with a tiny positive MCEMP1 log2FC still counts toward the majority
- The reference signature (MCEMP1 up, HLA-DRA down in CD14+ cells) was established from severe vs mild COVID-19 blood transcriptomics; sensitivity and specificity of these biomarkers have not been prospectively validated
- Heterogeneity across datasets and patients was noted by the authors, so a majority-direction acceptance rule is a coarse reproduction criterion

## Evidence

- [abstract] The most consistent differentially regulated genes in peripheral blood of severe COVID-19 patients were MCEMP1, HLA-DRA and ETS1 across the 7 transcriptomics datasets.: "The most consistent differentially regulated genes in peripheral blood of severe COVID-19 patients were MCEMP1, HLA-DRA and ETS1 across the 7 transcriptomics datasets."
- [abstract] we found significantly heightened MCEMP1 and reduced HLA-DRA expression as early as four days before the nadir of respiratory function: "we found significantly heightened MCEMP1 and reduced HLA-DRA expression as early as four days before the nadir of respiratory function, and the differential expression of MCEMP1 and HLA-DRA occurred"
- [intro] we ascertained that the dysregulation of MCEMP1 and HLA-DRA transcripts occurs primarily in the CD14+ immune cell subsets: "we ascertained that the dysregulation of MCEMP1 and HLA-DRA transcripts occurs primarily in the CD14+ immune cell subsets"
- [readme] The processed files contain p-value (t-test), adjusted p-value (BH step-up procedure), ratio, and fold-change data between severe COVID-19 and mild COVID-19: "The processed files contain p-value (t-test), adjusted p-value (BH step-up procedure), ratio, and fold-change data between severe COVID-19 and mild COVID-19, or severe COVID-19 and healthy subjects."
- [discussion] Future prospective studies will also be needed to validate the sensitivity and specificity of using MCEMP1/HLA-DRA expression levels in predicting severe COVID-19.: [evidence span withheld — anchor status: not_found]
