---
name: batch-integration-workflow-comparison
description: Use when you have one or more scRNA-seq perturbation datasets (e.g. drug-treatment
  vs control) with ground-truth labels and need to compare ICAT against conventional
  integration workflows quantitatively by ARI, LISI, and DB.
when_to_use_negative:
- The goal is only to run ICAT on a new dataset without comparing it to integration
  baselines — use the standalone ICAT clustering workflow instead.
- The data contain no perturbation or treatment contrast (single homogeneous condition),
  so integration-vs-ICAT comparison on treatment mixing is not meaningful.
- No ground-truth labels exist, making ARI (the primary cluster-quality metric in
  this benchmark) uncomputable.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: ICAT
  role: Primary method under benchmarking; performs control-guided NCFS gene weighting
    and semi-supervised Louvain clustering
  repo: https://github.com/BradhamLab/icat
- name: icat-sc
  role: pip-installable ICAT package used to run models.icat with treatment labels
    on each dataset
  repo: https://github.com/BradhamLab/icat
- name: scanpy
  role: Preprocessing (log1p normalization, PCA, neighbor graph), AnnData data structure,
    and metric computation environment for all workflows
- name: AnnData
  role: Data container holding expression matrix, obs treatment labels, and the ICAT
    upweighted matrix in obsm["X_icat"]
- name: Snakemake
  role: Pipeline executor for reproducing the full manuscript benchmark from the ICAT_manuscript
    repository
  repo: https://github.com/BradhamLab/ICAT_manuscript
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad278
    title: 'ICAT: a novel algorithm to robustly identify cell states following perturbations
      in single-cell transcriptomes'
  source_content_hash: sha256:d22d2ac34fc2f6b29219faeefeb9532081584f1968053703c44a7014897163bb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BradhamLab/icat
  tools:
  - AnnData
  - ICAT
  - Snakemake
  - icat-sc
  - numpy
  - scanpy
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# batch-integration-workflow-comparison

## Summary

Benchmark ICAT's control-guided semi-supervised clustering against standard integration-plus-Louvain workflows (Seurat, Scanorama, Harmony) on perturbed scRNA-seq datasets using ARI, LISI, and Davies-Bouldin metrics to test whether ICAT resolves perturbation-unique cell states better than batch-correction approaches.

## When to use

Use when you have one or more scRNA-seq perturbation datasets (e.g. drug-treatment vs control) with ground-truth labels and need to compare ICAT against conventional integration workflows quantitatively by ARI, LISI, and DB.

## When NOT to use

- The goal is only to run ICAT on a new dataset without comparing it to integration baselines — use the standalone ICAT clustering workflow instead.
- The data contain no perturbation or treatment contrast (single homogeneous condition), so integration-vs-ICAT comparison on treatment mixing is not meaningful.
- No ground-truth labels exist, making ARI (the primary cluster-quality metric in this benchmark) uncomputable.

## Inputs

- Perturbed scRNA-seq count matrices with treatment labels: Tian CellMix (H2228/HCC827/H1975/Mixed), Kagohara (cetuximab vs PBS), Kang (control vs IFN-beta) from the archived_data.tar.gz of BradhamLab/ICAT_manuscript
- Ground-truth cell-type/cell-state label annotations per dataset for ARI computation
- Scanpy AnnData objects preprocessed with log1p normalization, PCA, and a neighbor graph

## Outputs

- fig4_benchmark_metrics.csv containing ARI, LISI, and DB values for every method-dataset pair
- fig4_benchmark.png method-comparison plot reproducing Figure 4
- Per-method AnnData objects with sslouvain cluster assignments and the ICAT upweighted matrix in adata.obsm["X_icat"]

## How to apply

Retrieve the Tian (H2228/HCC827/H1975/Mixed CellMix), Kagohara (cetuximab vs PBS), and Kang (control vs IFN-beta) datasets from the BradhamLab/ICAT_manuscript archive and preprocess each in Scanpy with log1p normalization, PCA, and a neighbor graph. Run ICAT (pip install icat-sc) via models.icat with treatment labels so control cells define Louvain clusters and semi-supervised Louvain clusters all cells, reading the upweighted matrix from adata.obsm["X_icat"]. Run the three baseline integration + Louvain workflows (Seurat, Scanorama, Harmony) and the extended ICATSeurat, ICATScan, and ICATHarm workflows on the same preprocessed datasets so the comparison is paired method-by-method. For every method-dataset pair, compute ARI against ground-truth cell labels, LISI for treatment-batch mixing, and the Davies-Bouldin index for cluster separation. Assemble the results into fig4_benchmark_metrics.csv and render the method-comparison plot fig4_benchmark.png, reproducing Figure 4 of the paper. The rationale is that integration workflows correct away perturbation-specific signal, so ICAT should preserve perturbation-unique clusters that integration erases.

## Related tools

- **ICAT** (Primary method under benchmarking; performs control-guided NCFS gene weighting and semi-supervised Louvain clustering) — https://github.com/BradhamLab/icat
- **icat-sc** (pip-installable ICAT package used to run models.icat with treatment labels on each dataset) — https://github.com/BradhamLab/icat
- **scanpy** (Preprocessing (log1p normalization, PCA, neighbor graph), AnnData data structure, and metric computation environment for all workflows)
- **AnnData** (Data container holding expression matrix, obs treatment labels, and the ICAT upweighted matrix in obsm["X_icat"])
- **Snakemake** (Pipeline executor for reproducing the full manuscript benchmark from the ICAT_manuscript repository) — https://github.com/BradhamLab/ICAT_manuscript

## Examples

```
model = models.icat(ctrl_value="control", ncfs_kws={'reg': 1, 'sigma': 3}); out = model.cluster(adata, adata.obs['treatment'])
```

## Evaluation signals

- ARI values fall within [0, 1] for every method-dataset pair.
- The relative ICAT-vs-integration ordering matches the trends reported in Figure 4: ICAT outperforms Seurat, Scanorama, and Harmony on the Kagohara and Kang datasets by ARI, LISI, and DB.
- fig4_benchmark_metrics.csv contains one row per method-dataset pair covering all nine workflow variants (3 baselines, 3 extended ICAT workflows, ICAT) across the three datasets.
- ICAT output AnnData contains non-empty obsm["X_icat"] upweighted matrix and obs["sslouvain"] cluster labels before metrics are computed.

## Limitations

- The ICAT_manuscript analysis was run on a CentOS high-performance cluster and 'is not guaranteed to run on your system out-of-the-box'; the conda environment from environment.yaml or spec-file.txt must be activated (conda activate icat).
- ICAT hyperparameters (Louvain n_neighbors/resolution, NCFS sigma/reg) must be optimized for new data via optimize.optimize_louvain and optimize.optimize_ncfs prior to running the full workflow; original values are in the paper's supplemental tables.
- Integration-based baselines may legitimately win on datasets where treatment mixing rather than perturbation-state resolution is the goal; ICAT's advantage is specifically in resolving perturbation-unique cell states.

## Evidence

- [other] ICAT outperformed all three integration methods (Seurat, Scanorama, Harmony) in the Kagohara and Kang real datasets, producing higher-quality clusters by ARI, LISI, and DB metrics.: [evidence span withheld — anchor status: not_found]
- [readme] ICAT install via pip: "ICAT can be installed on linux machines using `pip` with the following command: `pip install icat-sc`"
- [readme] Manuscript reproduction instructions: [evidence span withheld — anchor status: external_doc]
- [abstract] ICAT rationale vs integration workflows: "We empirically validate ICAT in a developmental model and find that only ICAT identifies a perturbation-unique cellular response."
- [intro] Semi-supervised clustering description: "perform semi-supervised clustering such that the originally identified control labels remain constant, but cells from experimental conditions are free to cluster with any other cells regardless of"
