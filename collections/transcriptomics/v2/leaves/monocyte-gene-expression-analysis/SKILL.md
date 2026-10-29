---
name: monocyte-gene-expression-analysis
description: Use when analyzing peripheral-blood or monocyte transcriptomics (GEO/ArrayExpress
  raw counts or series matrices, e.g. GSE186650, E-MTAB-9721) to quantify MCEMP1,
  HLA-DRA or ETS1 dysregulation in CD14+ cells across COVID-19 severity or viral-protein
  perturbation conditions.
when_to_use_negative:
- When the question is the effect of different SARS-CoV-2 variants on transcriptomic
  responses — the included datasets did not directly evaluate variant effects
- When precise timing of MCEMP1/HLA-DRA dysregulation earlier than 4 days pre-nadir
  is needed — no public gene expression profiles exist more than 4 days before respiratory
  nadir
- When validating the sensitivity/specificity of MCEMP1/HLA-DRA as a prognostic test
  — this requires prospective studies not covered by the public datasets
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Streamlit
  role: Web app for querying gene-expression differences between severe and mild COVID-19
    patients and plotting box/strip plots of a selected gene per dataset
  repo: https://github.com/kuanrongchan/COVID19-severity
- name: pip
  role: Installs the app dependencies locally (pip install -r requirements.txt) before
    launching the query tool with streamlit run app.py
provenance:
  source_task_ids:
  - task_008
  - task_003
  source_papers:
  - doi: 10.1016/j.ebiom.2023.104472
    title: Early peripheral blood MCEMP1 and HLA-DRA expression predicts COVID-19
      prognosis
  source_content_hash: sha256:3f32230a3b97d34c2af019f35dcbe7db40257db5fad80cc8a00a7e0b8d35563b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kuanrongchan/COVID19-severity
  tools:
  - Streamlit
  - pip
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# monocyte-gene-expression-analysis

## Summary

Reproduce and interpret differential expression of the monocyte/CD14+ severity markers MCEMP1, HLA-DRA and ETS1 from public COVID-19 transcriptomic datasets — computing severe-vs-mild and viral-protein-vs-GFP log2 fold changes and aligning longitudinal expression to respiratory nadir — to support the paper's finding that early MCEMP1 upregulation and HLA-DRA downregulation in CD14+ cells prognosticate severe COVID-19.

## When to use

Use when analyzing peripheral-blood or monocyte transcriptomics (GEO/ArrayExpress raw counts or series matrices, e.g. GSE186650, E-MTAB-9721) to quantify MCEMP1, HLA-DRA or ETS1 dysregulation in CD14+ cells across COVID-19 severity or viral-protein perturbation conditions.

## When NOT to use

- When the question is the effect of different SARS-CoV-2 variants on transcriptomic responses — the included datasets did not directly evaluate variant effects
- When precise timing of MCEMP1/HLA-DRA dysregulation earlier than 4 days pre-nadir is needed — no public gene expression profiles exist more than 4 days before respiratory nadir
- When validating the sensitivity/specificity of MCEMP1/HLA-DRA as a prognostic test — this requires prospective studies not covered by the public datasets

## Inputs

- GEO series matrices / raw counts (e.g. GSE186650 monocyte coculture samples; GSE155454, GSE161731, GSE157103, GSE152418, GSE172114, GSE164805)
- ArrayExpress longitudinal whole-blood microarray dataset E-MTAB-9721 with clinical metadata mapping samples to days relative to respiratory nadir
- Bibert 2021 processed dataset (10.17632/8wxhhykfnh.2)
- Sample annotation tables mapping each sample to transfected protein condition (or GFP control) or disease severity / nadir day

## Outputs

- viral_protein_log2fc.csv: one row per SARS-CoV-2 protein with MCEMP1 and HLA-DRA log2 fold change vs GFP control
- nadir_aligned_expression.csv: per-day expression of MCEMP1, HLA-DRA and ETS1 relative to respiratory nadir
- nadir_trajectories.png: gene-expression trajectory plot around nadir (matplotlib/plotly)
- Box and strip plots of gene-of-interest expression per dataset with t-test p-values, BH-adjusted p-values, ratio and fold-change statistics

## How to apply

For cross-sectional severity data, download raw counts and processed files from the curated 7-dataset repository (GSE155454, Bibert 2021, GSE161731, GSE157103, GSE152418, GSE172114, GSE164805), compare severe vs mild COVID-19 (or severe vs healthy), and evaluate t-test p-values with BH step-up adjusted p-values, ratios and fold changes per gene. For the SARS-CoV-2 protein mechanism (GSE186650), parse sample annotations with pandas to map each monocyte/epithelial-coculture sample to its transfected protein, average duplicated transfection conditions, and compute per-gene log2FC as mean expression under each viral protein minus the mean GFP-control expression; accept the reproduction if the majority (>50%) of proteins show MCEMP1 upregulation and HLA-DRA downregulation. For longitudinal kinetics (E-MTAB-9721), align each whole-blood microarray sample to days relative to the patient's respiratory nadir, bin by day-relative-to-nadir, and verify that MCEMP1 is heightened and HLA-DRA reduced as early as 4 days pre-nadir, ETS1 drops at nadir, and all three transcripts return to baseline 7-8 days post-nadir. Interpret dysregulation as localized to CD14+ immune cell subsets, checking single-cell PBMC/BALF data where available.

## Related tools

- **Streamlit** (Web app for querying gene-expression differences between severe and mild COVID-19 patients and plotting box/strip plots of a selected gene per dataset) — https://github.com/kuanrongchan/COVID19-severity
- **pip** (Installs the app dependencies locally (pip install -r requirements.txt) before launching the query tool with streamlit run app.py)

## Examples

```
mean_expr = df.groupby('condition')[['MCEMP1','HLA-DRA']].mean(); log2fc = mean_expr.loc[protein] - mean_expr.loc['GFP']
```

## Evaluation signals

- In the GSE186650 reproduction, the fraction of SARS-CoV-2 proteins with positive MCEMP1 log2FC and negative HLA-DRA log2FC exceeds 50% relative to GFP
- In the E-MTAB-9721 longitudinal analysis, MCEMP1 is significantly heightened and HLA-DRA reduced already in the earliest bin, 4 days before nadir, with ETS1 reduced only at nadir
- All three transcripts (MCEMP1, HLA-DRA, ETS1) return to baseline in bins 7-8 days post-nadir
- Per-dataset processed outputs carry t-test p-values and BH step-up adjusted p-values alongside ratio and fold-change columns between severe and mild (or severe and healthy) groups
- MCEMP1/HLA-DRA differential expression localizes to CD14+ immune cell subsets when cross-checked against PBMC/BALF single-cell data

## Limitations

- Heterogeneity across the study datasets (Supplementary Fig. S9) means patient factors influencing MCEMP1 and HLA-DRA expression are not fully understood
- Sensitivity and specificity of MCEMP1/HLA-DRA expression for predicting severe COVID-19 have not been validated; prospective studies are needed
- No gene expression profiles exist more than 4 days before respiratory nadir, so the precise onset of dysregulation is undetermined
- The role of MCEMP1 in monocytes — gene regulation, function and protein expression after SARS-CoV-2 infection — remains uncharacterised

## Evidence

- [abstract] Elevated MCEMP1 and reduced HLA-DRA gene expression in CD14+ cells during the early phase of disease are prognostic of severe COVID-19: "Elevated MCEMP1 and reduced HLA-DRA gene expression in CD14+ cells during the early phase of disease are prognostic of severe COVID-19."
- [intro] MCEMP1 heightened and HLA-DRA reduced four days pre-nadir: "MCEMP1 and HLA-DRA were found to be differentially expressed in severe COVID-19 patients as early as four days from nadir of respiratory function"
- [readme] Processed files contain t-test p-values, BH adjusted p-values, ratio and fold-change: "The processed files contain p-value (t-test), adjusted p-value (BH step-up procedure), ratio, and fold-change data between severe COVID-19 and mild COVID-19"
- [intro] Dysregulation occurs primarily in CD14+ immune cell subsets: "the dysregulation of MCEMP1 and HLA-DRA transcripts occurs primarily in the CD14+ immune cell subsets"
