---
name: kaplan-meier-survival-analysis
description: Use when a tumor cell subcluster (e.g., Louvain Cluster 4 from scanpyobj.h5ad)
  has cluster-specific DEGs and you need to test whether its gene signature stratifies
  TCGA patient cohorts (e.g., TCGA-LIHC) into high- vs low-survival groups.
when_to_use_negative:
- When the goal is drug response prediction (IC50, PRISM AUC) rather than patient
  survival — use drug_response_prediction.py instead of the --survival flag.
- When no matched TCGA bulk expression and clinical survival data are available for
  the cohort of interest.
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0203
tools:
- name: scDrug
  role: Provides single_cell_analysis.py, whose --survival, --tcga, --id, and --not_treated
    flags perform the cluster-signature Kaplan-Meier analysis against TCGA cohorts.
  repo: https://github.com/ailabstw/scDrug
- name: Scanpy
  role: Produces the Anndata object (scanpyobj.h5ad) and DEG table that supply the
    cluster-specific gene signature used to score patient expression.
  repo: https://github.com/ailabstw/scDrug
- name: Python
  role: Execution environment for the scDrug pipeline and the survival scoring/plotting
    code.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.055
    title: 'scDrug: From single-cell RNA-seq to drug response prediction'
  source_content_hash: sha256:1c69978993c7d91dbd3cb6376e9de061e31527f9bc4b6664c436679167e743e7
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
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# kaplan-meier-survival-analysis

## Summary

Survival analysis step within scDrug's Single-Cell Data Analysis that links a cluster-specific gene signature from scRNA-seq data (e.g., GSE156625 hepatocellular carcinoma Cluster 4 with top DEGs PTMA and STMN1) to patient survival in TCGA bulk RNA-seq, via activity-score stratification, Kaplan-Meier curves, log-rank testing, and hazard ratio estimation.

## When to use

Use when a tumor cell subcluster (e.g., Louvain Cluster 4 from scanpyobj.h5ad) has cluster-specific DEGs and you need to test whether its gene signature stratifies TCGA patient cohorts (e.g., TCGA-LIHC) into high- vs low-survival groups.

## When NOT to use

- When the goal is drug response prediction (IC50, PRISM AUC) rather than patient survival — use drug_response_prediction.py instead of the --survival flag.
- When no matched TCGA bulk expression and clinical survival data are available for the cohort of interest.

## Inputs

- Scanpy Anndata clustering output (scanpyobj.h5ad) from single_cell_analysis.py
- Cluster differentially expressed genes (cluster_DEGs.csv), top 20 DEGs of the target cluster
- TCGA bulk RNA-seq expression profiles downloaded from the TCGA database (e.g., TCGA-LIHC)
- Matching TCGA clinical/survival information

## Outputs

- Kaplan-Meier plot (e.g., cluster4_kaplan_meier.png)
- CSV of group labels, hazard ratio, and log-rank p-value (e.g., cluster4_survival_results.csv)

## How to apply

Take the clustering output (scanpyobj.h5ad and cluster_DEGs.csv) from single_cell_analysis.py and select the top 20 differentially expressed genes of the target cluster as the signature (for Cluster 4 these include PTMA and STMN1). Download TCGA bulk RNA expression and matching clinical data from the TCGA database for the relevant project id (e.g., TCGA-LIHC), optionally restricting to untreated samples with --not_treated. For each signature gene, assign a binary value of 1 when a patient's expression exceeds the gene's median across all patients (else 0) and sum these to obtain the cluster activity score per patient. Divide patients into 'high-expressing' and 'low-expressing' groups if their activity score is in the top or bottom quartile. Fit Kaplan-Meier curves for the two groups with the log-rank test, compute the hazard ratio, and save the plot (cluster4_kaplan_meier.png) and group labels, hazard ratio, and log-rank p-value (cluster4_survival_results.csv).

## Related tools

- **scDrug** (Provides single_cell_analysis.py, whose --survival, --tcga, --id, and --not_treated flags perform the cluster-signature Kaplan-Meier analysis against TCGA cohorts.) — https://github.com/ailabstw/scDrug
- **Scanpy** (Produces the Anndata object (scanpyobj.h5ad) and DEG table that supply the cluster-specific gene signature used to score patient expression.) — https://github.com/ailabstw/scDrug
- **Python** (Execution environment for the scDrug pipeline and the survival scoring/plotting code.)

## Examples

```
python3 single_cell_analysis.py -f h5ad --input scanpyobj.h5ad --survival --tcga /path/to/TCGA --id TCGA-LIHC --not_treated
```

## Evaluation signals

- Reproducing the Cluster 4 / TCGA-LIHC analysis should yield a hazard ratio of approximately 2.52 with -log2(p) ≈ 14 (log-rank p-value far below 0.05), matching Supplementary Fig. S1.
- High- and low-expressing groups must correspond to the top and bottom quartiles of the summed binary activity score.
- The per-gene binarization must use each gene's median expression across all patients as the 0/1 cutoff.
- With --not_treated, only untreated TCGA samples should enter the survival cohort.
- Outputs must include both the Kaplan-Meier plot (cluster4_kaplan_meier.png) and the CSV with group labels, hazard ratio, and log-rank p-value.

## Limitations

- In the prostate cancer dataset analyzed in the paper, none of the tumor clusters yielded statistical significance at p-value < 0.05; the authors attribute this to poor alignment of expression between scRNA-seq profiles and limited TCGA data.
- Significance of one cluster's signature (e.g., HCC Cluster 4) does not generalize — cluster signatures may fail to reach p < 0.05 in other cancer types or smaller TCGA cohorts.

## Evidence

- [intro] Single-Cell Data Analysis performed data preprocessing, clustering, cell type annotation, Gene Set Enrichment Analysis (GSEA), and survival analysis.: "Single-Cell Data Analysis performed data preprocessing, clustering, cell type annotation, Gene Set Enrichment Analysis (GSEA), and survival analysis."
- [methods] Quartile grouping of activity scores: [evidence span withheld — anchor status: not_found]
- [methods] TCGA data source: "downloaded from the TCGA database [29]"
- [readme] Survival CLI flags: "--survival            perform survival analysis
  --tcga TCGA           path to TCGA data
  --id ID               Specify TCGA project id in the format "TCGA-xxxx", e.g., "TCGA-LIHC""
- [readme] Untreated-sample restriction flag: "--not_treated         only consider untreated samples from TCGA for survival analysis."
- [results] Known failure mode in prostate cohort: "none of tumor clusters in this dataset yielded a statistical significance at p-value <0.05 for the survival analysis."
