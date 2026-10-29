---
name: bulk-rna-seq-activity-score-computation
description: Use when a cluster-specific gene signature (e.g., top 20 DEGs of a scRNA-seq
  tumor cluster such as Cluster 4 with PTMA and STMN1) must be translated into a bulk
  cohort metric to stratify patients for Kaplan-Meier/log-rank survival testing.
when_to_use_negative:
- The input is already per-cluster drug response predictions (IC50/AUC) rather than
  a gene signature needing bulk-cohort scoring
- No matched clinical survival data exist for the bulk cohort, so no high/low survival
  grouping can be tested
- The cluster signature itself showed no survival significance (as with the prostate
  cancer tumor clusters, where none reached p < 0.05)
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_3308
tools:
- name: scDrug
  role: Provides single_cell_analysis.py with --survival, --tcga, --id and --not_treated
    flags that download TCGA data and perform the activity-score-based survival analysis
  repo: https://github.com/ailabstw/scDrug
- name: Scanpy
  role: Produces the upstream clustering output (scanpyobj.h5ad, cluster_DEGs.csv)
    from which the top Cluster 4 DEGs including PTMA and STMN1 are selected as the
    signature
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.055
    title: 'scDrug: From single-cell RNA-seq to drug response prediction'
  source_content_hash: sha256:31f90d11c4106bf345414d051cef9cfee842fdba89a00e63cb3e893e31a147e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ailabstw/scDrug
  tools:
  - Python
  - Scanpy
  - scDrug
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# Bulk RNA-seq activity score computation

## Summary

Computes a per-patient activity score for a cluster-derived gene signature in bulk RNA-seq (e.g., TCGA) by binarizing each signature gene at its median expression (1 if above median, else 0) and summing across genes, enabling stratification of patients into high- vs low-expressing groups for survival analysis.

## When to use

Use when a cluster-specific gene signature (e.g., top 20 DEGs of a scRNA-seq tumor cluster such as Cluster 4 with PTMA and STMN1) must be translated into a bulk cohort metric to stratify patients for Kaplan-Meier/log-rank survival testing.

## When NOT to use

- The input is already per-cluster drug response predictions (IC50/AUC) rather than a gene signature needing bulk-cohort scoring
- No matched clinical survival data exist for the bulk cohort, so no high/low survival grouping can be tested
- The cluster signature itself showed no survival significance (as with the prostate cancer tumor clusters, where none reached p < 0.05)

## Inputs

- Cluster DEG list (cluster_DEGs.csv or top genes from scanpyobj.h5ad) defining the cluster-specific gene signature
- TCGA bulk RNA expression profiles (e.g., TCGA-LIHC matrix.mtx/genes.tsv/barcodes.tsv with --prefix)
- Matching TCGA clinical/survival information for the same patients

## Outputs

- Per-patient activity score column (sum of median-binarized signature gene values)
- Binary high/low-expressing group labels based on top/bottom activity-score quartiles
- Kaplan-Meier plot (e.g., cluster4_kaplan_meier.png)
- Survival results file with group labels, hazard ratio, and log-rank p-value (e.g., cluster4_survival_results.csv)

## How to apply

Take the top 20 differentially expressed genes of the cluster from the upstream Scanpy output (cluster_DEGs.csv / scanpyobj.h5ad) as the signature. For each gene in the bulk cohort (e.g., TCGA-LIHC expression profiles downloaded from TCGA), assign a binary value of 1 if the patient's expression exceeds the gene's median across all patients, else 0, then sum the column to get the Cluster 4 activity score per patient. Optionally restrict to untreated TCGA samples using the --not_treated flag of single_cell_analysis.py. Divide patients into 'high-expressing' and 'low-expressing' groups using the top and bottom quartiles of the activity score, then fit Kaplan-Meier curves with a log-rank test and compute the hazard ratio, saving the plot and results (e.g., cluster4_kaplan_meier.png, cluster4_survival_results.csv).

## Related tools

- **scDrug** (Provides single_cell_analysis.py with --survival, --tcga, --id and --not_treated flags that download TCGA data and perform the activity-score-based survival analysis) — https://github.com/ailabstw/scDrug
- **Scanpy** (Produces the upstream clustering output (scanpyobj.h5ad, cluster_DEGs.csv) from which the top Cluster 4 DEGs including PTMA and STMN1 are selected as the signature)

## Examples

```
python3 single_cell_analysis.py -f h5ad --input scanpyobj.h5ad --survival --tcga /path/TCGA --id TCGA-LIHC --not_treated
```

## Evaluation signals

- Reproduced TCGA-LIHC analysis of the Cluster 4 signature yields a hazard ratio of approximately 2.52 with -log2(p) ≈ 14 (log-rank p-value far below 0.05), matching the paper's Supplementary Fig. S1
- Each signature gene's binary value equals 1 only for patients with expression above that gene's median across all patients (check a few genes manually)
- High-expressing and low-expressing groups correspond exactly to the top and bottom quartiles of the summed activity score
- Output files cluster4_kaplan_meier.png and cluster4_survival_results.csv exist and the CSV contains group labels, hazard ratio, and log-rank p-value
- Log-rank p-value is reported far below the 0.05 significance threshold

## Limitations

- Survival association may not replicate for all cancer types: in the paper's prostate cancer dataset, none of the tumor clusters yielded statistical significance at p-value < 0.05, attributed to poor alignment of expression profiles and limited TCGA data
- Binary median-threshold scoring is coarse; results depend on the choice of top 20 DEGs and quartile cutoffs
- TCGA cohort can optionally be restricted to untreated samples (--not_treated), reducing sample size

## Evidence

- [methods] For each of the 20 signature genes, assign a binary value of 1 if the gene expression in a patient exceeds the gene's median expression across all patients, else 0: [evidence span withheld — anchor status: not_found]
- [supplementary] Reproduced analysis matches reported survival separation: [evidence span withheld — anchor status: not_found]
- [readme] README documents the survival-analysis flags used by this skill: "--survival            perform survival analysis
  --tcga TCGA           path to TCGA data"
- [readme]  untreated-sample restriction is a key option for this analysis: "--not_treated         only consider untreated samples from TCGA for survival analysis."
