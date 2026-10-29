---
name: synthetic-differential-expression-perturbation
description: Use when benchmarking single-cell drug repurposing pipelines (e.g., ASGARD
  vs. Alakwaa's vs. Guo's) for AUC robustness across simulation grids where ground
  truth must be known by construction rather than inferred.
when_to_use_negative:
- When the goal is drug recommendation on real patient data rather than method benchmarking
  — use the standard ASGARD GetDrug/DrugScore workflow instead.
- When ground-truth drugs cannot be encoded by construction (e.g., purely observational
  case-control data without an injected DE program) — ROC/AUC robustness curves are
  not interpretable.
- When comparing pipelines that require bulk-RNA signatures only, since this simulation
  design is specific to single-cell case-control inputs.
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: ASGARD
  role: Pipeline under test; run with GetDrugRef and GetDrug using the multi-cluster
    drug score on each simulated dataset
  repo: https://github.com/lanagarmire/ASGARD
- name: Seurat
  role: Loads and preprocesses the resampled GSE123926/GSE113197 expression matrices
    used to build synthetic datasets
- name: CLUE
  role: Platform used to run Alakwaa's pipeline on the identical simulation replicates
- name: pROC
  role: Computes ROC curves and AUCs for every pipeline at every parameter-grid setting
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-36637-3
    title: ASGARD is A Single-cell Guided Pipeline to Aid Repurposing of Drugs
  source_content_hash: sha256:41e99ff8d37430a66576f72bd5e341a6d988e57941da9fb7c01906a074dca2b4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/ASGARD
  tools:
  - ASGARD
  - CLUE
  - Seurat
  - pROC
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# synthetic-differential-expression-perturbation

## Summary

A simulation-based robustness assessment that bootstraps synthetic case-control scRNA-seq datasets from real cells (GSE123926 TNBC PDX and GSE113197 normal breast epithelium) while varying population sizes, differential-expression levels, and diseased-cell proportions, so the ground-truth drug is known by construction and pipeline AUCs can be benchmarked with pROC.

## When to use

Use when benchmarking single-cell drug repurposing pipelines (e.g., ASGARD vs. Alakwaa's vs. Guo's) for AUC robustness across simulation grids where ground truth must be known by construction rather than inferred.

## When NOT to use

- When the goal is drug recommendation on real patient data rather than method benchmarking — use the standard ASGARD GetDrug/DrugScore workflow instead.
- When ground-truth drugs cannot be encoded by construction (e.g., purely observational case-control data without an injected DE program) — ROC/AUC robustness curves are not interpretable.
- When comparing pipelines that require bulk-RNA signatures only, since this simulation design is specific to single-cell case-control inputs.

## Inputs

- GSE123926 TNBC PDX epithelial scRNA-seq expression matrices from GEO
- GSE113197 normal breast epithelial scRNA-seq expression matrices from GEO
- Breast tissue-specific drug reference profile (e.g., DrugReference/breast_rankMatrix.txt) built from LINCS L1000 GSE70138/GSE92742 via PrepareReference

## Outputs

- ROC curves and AUC values per pipeline per simulation-grid setting (computed with pROC)
- Robust-AUC comparison curves saved as supplementary_fig3_robust_auc_curves.pdf

## How to apply

Download GSE123926 (TNBC PDX epithelial cells) and GSE113197 (normal breast epithelial cells) from GEO and load the expression matrices with Seurat. Generate bootstrapped synthetic case-control datasets by resampling real cells while sweeping cell-population sizes, DE levels from 20% to 90%, and diseased-cell proportions from 20% to 90%; because the injected DE program encodes a known drug, the ground truth is known by construction. Run each pipeline on every replicate: ASGARD via GetDrugRef (breast_rankMatrix.txt reference) and GetDrug with the multi-cluster drug score, Alakwaa's pipeline via the CLUE platform, and Guo's cluster-level repurposing pipeline on identical replicates. Compute ROC curves and AUCs for every pipeline at every grid setting with the pROC package, then plot robust-AUC curves to compare parameter sensitivity. Judge success by whether AUCs remain stable (robust) across the whole parameter grid, with ASGARD expected to stay above the competitors.

## Related tools

- **ASGARD** (Pipeline under test; run with GetDrugRef and GetDrug using the multi-cluster drug score on each simulated dataset) — https://github.com/lanagarmire/ASGARD
- **Seurat** (Loads and preprocesses the resampled GSE123926/GSE113197 expression matrices used to build synthetic datasets)
- **CLUE** (Platform used to run Alakwaa's pipeline on the identical simulation replicates)
- **pROC** (Computes ROC curves and AUCs for every pipeline at every parameter-grid setting)

## Examples

```
Drug.ident.res = GetDrug(gene.data = Gene.list, drug.ref.profiles = drug.ref.profiles)
```

## Evaluation signals

- AUCs remain stable across the full simulation grid (population sizes, DE levels 20%–90%, diseased-cell proportions 20%–90%), matching Supplementary Fig. 3A–C robustness pattern.
- ASGARD's AUC stays above Alakwaa's and Guo's pipelines across the whole range of simulation parameters in the plotted curves.
- Every synthetic replicate has a verifiable ground-truth drug encoded by the injected DE program (known by construction).
- ROC/AUC values are reproducible via the pROC-based computation for all three pipelines on identical replicates.
- The robust-AUC figure is regenerated as supplementary_fig3_robust_auc_curves.pdf from the /Figure scripts in the Single-cell-drug-repositioning repo.

## Limitations

- AUC performance varies with single-cell DE method choice; sparsity and heterogeneity of single-cell data can cause false discoveries of differentially expressed genes, affecting simulation outcomes.
- Quantitative AUC results are only referenced via figures (Supplementary Fig. 3A–C), not reported numerically in the text.
- The drug reference is limited to 98 cell lines from LINCS L1000, so tissue-specific reference availability constrains which disease contexts can be simulated.

## Evidence

- [results] Bootstrapping design and grid parameters: [evidence span withheld — anchor status: not_found]
- [discussion] Benchmark metric computation: "ROCs and AUCs are calculated for each pipeline using the pROC package79"
- [methods] Multi-cluster drug score applied to simulated data: [evidence span withheld — anchor status: not_found]
- [results] Ground truth by construction: [evidence span withheld — anchor status: not_found]
- [readme] pROC package from README context: [evidence span withheld — anchor status: external_doc]
