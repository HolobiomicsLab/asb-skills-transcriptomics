---
name: artificial-scrnaseq-dataset-generation
description: Use when benchmarking or reproducing accuracy/runtime comparisons of
  DEG prediction methods on artificial single-cell RNA-seq data where the set of true
  DEGs is known by construction, such as validating that singleCellHaystack outperforms
  cluster-based methods.
when_to_use_negative:
- The data are real experimental scRNA-seq data without a planted ground-truth DEG
  set — AUC against true labels cannot be computed.
- The goal is DEG prediction itself on a new biological dataset rather than benchmarking
  or comparing DEG methods on artificial data.
- Only continuous normalized expression is available without a detection threshold
  — the current singleCellHaystack implementation requires binary detection data.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: singleCellHaystack
  role: Method under benchmark; run haystack() with cell coordinates and binary expression/detection
    input on each artificial dataset
  repo: https://github.com/alexisvdb/singleCellHaystack
- name: Seurat
  role: Comparison method; run FindAllMarkers both with and without default filtering
    on the same artificial datasets
  repo: https://github.com/satijalab/seurat
- name: ROCR
  role: Compute AUC of the DEG detection ROC curve from method-specific significance
    scores with planted DEGs as true labels
- name: DEsingle
  role: Cluster-based comparison DEG method; runtime recorded per artificial dataset
- name: scDD
  role: Comparison method; known to have far longer runtimes and to fail on the largest
    datasets
- name: edgeR
  role: Bulk RNA-seq comparison DEG method applied to the artificial datasets
- name: monocle 2
  role: Comparison DEG method applied to the artificial datasets
- name: EMDomics
  role: Comparison DEG method applied to the artificial datasets
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-17900-3
    title: A clustering-independent method for finding differentially expressed genes
      in single-cell transcriptome data
  source_content_hash: sha256:698fb8e9fd478f9919923921667ebf98c43c5c7dba697986dfe847c9f173b8c7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexisvdb/singleCellHaystack
  tools:
  - DEsingle
  - EMDomics
  - R
  - ROCR
  - Seurat
  - edgeR
  - monocle 2
  - scDD
  - singleCellHaystack
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# artificial-scrnaseq-dataset-generation

## Summary

Generate artificial scRNA-seq datasets with known differentially expressed genes (DEGs) planted in non-randomly positioned cell subsets, across a range of cell numbers (e.g., 1,000–10,000 cells), to benchmark DEG prediction methods such as singleCellHaystack against existing approaches using AUC and runtime metrics.

## When to use

Use when benchmarking or reproducing accuracy/runtime comparisons of DEG prediction methods on artificial single-cell RNA-seq data where the set of true DEGs is known by construction, such as validating that singleCellHaystack outperforms cluster-based methods.

## When NOT to use

- The data are real experimental scRNA-seq data without a planted ground-truth DEG set — AUC against true labels cannot be computed.
- The goal is DEG prediction itself on a new biological dataset rather than benchmarking or comparing DEG methods on artificial data.
- Only continuous normalized expression is available without a detection threshold — the current singleCellHaystack implementation requires binary detection data.

## Inputs

- Artificial scRNA-seq datasets with known planted DEGs in non-randomly positioned cell subsets, at multiple cell sizes (1,000–10,000 cells)
- Cell coordinates in a multidimensional space (e.g., t-SNE/PCA embeddings)
- Binary gene detection matrix (gene detected or not per cell)

## Outputs

- benchmark_median_auc_runtime.csv with one row per method per dataset size (median AUC, median runtime)
- Per-method AUC values from ROCR ROC curves using planted DEGs as true labels
- Per-method runtime measurements (elapsed time per dataset)

## How to apply

Generate artificial scRNA-seq datasets containing known DEGs expressed in non-randomly positioned cell subsets, at multiple cell numbers (e.g., 1,000 up to 10,000 cells). On each dataset, run singleCellHaystack via its R interface (haystack() with cell coordinates and binary detection input, since the method requires binary detection data — a gene is either detected or not in each cell) and record elapsed runtime. Run comparison methods on the same datasets — DEsingle, EMDomics, scDD, edgeR, monocle 2, and Seurat FindAllMarkers with and without default filtering — noting failures such as scDD on the largest sets. Rank genes by each method's significance scores and compute the AUC of the DEG detection ROC curve using ROCR, treating planted DEGs as true labels. Aggregate median AUC and median runtime per method per dataset size and write the summary table (method, dataset size, median AUC, median runtime) to benchmark_median_auc_runtime.csv.

## Related tools

- **singleCellHaystack** (Method under benchmark; run haystack() with cell coordinates and binary expression/detection input on each artificial dataset) — https://github.com/alexisvdb/singleCellHaystack
- **Seurat** (Comparison method; run FindAllMarkers both with and without default filtering on the same artificial datasets) — https://github.com/satijalab/seurat
- **ROCR** (Compute AUC of the DEG detection ROC curve from method-specific significance scores with planted DEGs as true labels)
- **DEsingle** (Cluster-based comparison DEG method; runtime recorded per artificial dataset)
- **scDD** (Comparison method; known to have far longer runtimes and to fail on the largest datasets)
- **edgeR** (Bulk RNA-seq comparison DEG method applied to the artificial datasets)
- **monocle 2** (Comparison DEG method applied to the artificial datasets)
- **EMDomics** (Comparison DEG method applied to the artificial datasets)

## Examples

```
res <- haystack(dat.tsne, dat.expression)
```

## Evaluation signals

- benchmark_median_auc_runtime.csv contains one row per method per dataset size with finite median AUC in [0,1] and finite median runtime
- AUC ordering reproduces the finding that singleCellHaystack attains the highest accuracy on artificial datasets except at the smallest size (1,000 cells)
- singleCellHaystack median runtime is on the order of ~5 minutes on 10,000-cell datasets, faster than cluster-based methods like DEsingle and scDD
- scDD failures are recorded on the largest datasets rather than silently omitted
- Per-replicate aggregation uses median across replicates so outlier runtimes do not dominate the summary

## Limitations

- The current implementation of singleCellHaystack requires binary detection data as input; continuous normalized expression measures are planned but not supported.
- Use of a hard detection threshold is a weak point: two genes detected in the same subset of cells can get identical D_KL and p-value regardless of expression level.
- P-values are inflated because gene expression data are used twice (for coordinates/embeddings and for DEG prediction); singleCellHaystack suffers from this same issue.
- Zero counts may reflect dropout events from lack of sensitivity; imputation is not applied in this workflow.

## Evidence

- [abstract] Higher accuracy on artificial datasets: "Comparisons with existing DEG prediction approaches on artificial datasets show that singleCellHaystack has higher accuracy"
- [abstract] DKL finds genes in non-random subsets: "Our method uses Kullback-Leibler divergence to find genes that are expressed in subsets of cells that are non-randomly positioned in a multidimensional space"
- [methods] haystack invocation on coordinates and expression: "res <- haystack(dat.tsne, dat.expression)"
- [discussion] Binary detection requirement: "The current implementation of our method requires binary detection data as input (i.e., a gene is either detected or not in each cell)."
- [methods] Filtering lowly expressed genes reduces runtime: "We filter genes with less 10 cells with non-zero counts. This reduces the computational time by eliminating very lowly expressed genes."
- [readme] README: version note on original method: [evidence span withheld — anchor status: external_doc]
