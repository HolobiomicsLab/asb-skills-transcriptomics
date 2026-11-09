---
name: facs-label-mapping-for-clustering-benchmarking
description: Use when benchmarking scRNA-seq batch-correction or clustering methods
  on data with FACS-sorted cell-type labels, to quantify clustering consistency (ARI)
  and separation (silhouette) against true labels.
when_to_use_negative:
- The scRNA-seq data have no independently measured cell-type labels (e.g., no FACS
  sorting), so no ground truth exists to compute an ARI against
- The experimental design is completely confounded (each batch measures disjoint cell
  types), for which the article states 'no method is applicable'
- The goal is bulk-RNA-seq batch correction with known subtype labels, where ComBat/SVA-style
  methods are the intended design
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: BUSseq
  role: Fits the Bayesian hierarchical model via BUSseq_MCMC; cluster assignments
    (K selected by BIC, K=3..6) are scored against FACS labels
  repo: https://github.com/songfd2018/BUSseq-Rpackage
- name: Scanorama
  role: Competing MNN-based batch-correction method whose clusters are evaluated against
    FACS labels
- name: Seurat
  role: Competing shared-variation integration method whose clusters are evaluated
    against FACS labels
- name: LIGER
  role: Competing shared-variation integration method whose clusters are evaluated
    against FACS labels
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-16905-2
    title: Flexible experimental designs for valid single-cell RNA-sequencing experiments
      allowing batch effects correction
  source_content_hash: sha256:39338a52ae73aa429cbc11ea15a1c058b7ed9f7afdda4185465d965acb7d5b10
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/songfd2018/BUSseq-Rpackage
  tools:
  - BUSseq
  - ComBat
  - LIGER
  - RUV
  - SVA
  - Scanorama
  - Seurat
  - scMerge
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# facs-label-mapping-for-clustering-benchmarking

## Summary

Maps FACS-sorted ground-truth cell-type labels onto scRNA-seq cell barcodes so that clustering results from batch-correction methods can be scored by Adjusted Rand Index and silhouette, as done in the BUSseq hematopoietic benchmark (BUSseq ARI 0.582 vs six competitors).

## When to use

Use when benchmarking scRNA-seq batch-correction or clustering methods on data with FACS-sorted cell-type labels, to quantify clustering consistency (ARI) and separation (silhouette) against true labels.

## When NOT to use

- The scRNA-seq data have no independently measured cell-type labels (e.g., no FACS sorting), so no ground truth exists to compute an ARI against
- The experimental design is completely confounded (each batch measures disjoint cell types), for which the article states 'no method is applicable'
- The goal is bulk-RNA-seq batch correction with known subtype labels, where ComBat/SVA-style methods are the intended design

## Inputs

- Raw read count matrices from hematopoietic scRNA-seq studies (GEO accessions GSE72857 and GSE81682)
- FACS-sorted cell-type labels for each cell (ground-truth annotations)
- Preprocessed matrix restricted to 3,470 highly variable genes common across batches
- Cluster assignments and corrected datasets from each batch-correction method

## Outputs

- Per-method Adjusted Rand Index against FACS labels (e.g., BUSseq 0.582, MNN 0.575, ZINB-WaVE 0.348, LIGER 0.307, Seurat 0.266, Scanorama 0.518, scVI 0.197)
- Silhouette coefficients computed on each corrected dataset
- Combined comparison table saved as batch_correction_benchmark.csv

## How to apply

Restrict the analysis to the 3,470 highly variable genes common across batches and load the raw read count matrices (e.g., GEO accessions GSE72857 and GSE81682 hematopoietic data). Attach the FACS-sorted cell-type labels to each cell as the evaluation ground truth, keeping them strictly held out from the fitting stage (BUSseq itself clusters unknown subtypes via BUSseq_MCMC, selecting K by BIC enumeration from 3 to 6). Run each batch-correction method (BUSseq, ComBat, SVA, RUV, Scanorama, Seurat, LIGER/scMerge) on the same preprocessed matrices to obtain cluster assignments and corrected values, which can be used downstream as if all cells were sequenced in a single batch. Compute the Adjusted Rand Index of each method's clusters against the FACS labels and the silhouette coefficients of the corrected data, then assemble per-method values into a single comparison table (batch_correction_benchmark.csv). The skill is applied correctly when the label-matched cells are identical across methods so ARI values are directly comparable, and the best method (BUSseq, 0.582) tops the table.

## Related tools

- **BUSseq** (Fits the Bayesian hierarchical model via BUSseq_MCMC; cluster assignments (K selected by BIC, K=3..6) are scored against FACS labels) — https://github.com/songfd2018/BUSseq-Rpackage
- **Scanorama** (Competing MNN-based batch-correction method whose clusters are evaluated against FACS labels)
- **Seurat** (Competing shared-variation integration method whose clusters are evaluated against FACS labels)
- **LIGER** (Competing shared-variation integration method whose clusters are evaluated against FACS labels)

## Examples

```
install_github("songfd2018/BUSseq-Rpackage"); fit <- BUSseq_MCMC(ObservedData, n.celltypes = K, seed, n.cores = 4, n.iterations = 500, n.burnin, n.unchanged, n.output, working_dir)
```

## Evaluation signals

- The set of label-matched cells and the 3,470-gene feature space are identical across all methods before ARI computation
- BUSseq ARI of 0.582 is reproduced as the maximum among the compared methods, followed by MNN (0.575) and Scanorama (0.518)
- Silhouette coefficients are computed on each method's corrected values, not on raw counts
- batch_correction_benchmark.csv contains one ARI and one silhouette value per method with no missing entries
- Optimal K for BUSseq is chosen by minimum BIC over the enumerated K = 3 to 6, not by tuning against the FACS labels

## Limitations

- MCMC inference is computationally heavy (though most steps are parallelizable over cores, e.g., 500 iterations on four cores per K)
- Completely confounded designs admit no applicable method, so no label-based benchmark can rescue them
- MNN/Scanorama-style competitors assume batch effects are orthogonal to and much smaller than biological variation — assumptions that cannot be validated at the design stage and may bias the comparison

## Evidence

- [methods] Compute the Adjusted Rand Index of each method's clusters against the FACS labels and the silhouette coefficients of the corrected data.: [evidence span withheld — anchor status: not_found]
- [methods] BIC enumeration for K selection: "To select the optimal *K*, we need to compare the BIC values for different *K*s. In the vignettes, we enumerate *K* from 3 to 6."
- [readme] Corrected values usable downstream: "the corrected value can be used for downstream analysis as if all cells were sequenced in a single batch"
- [methods] Attaching FACS labels for evaluation: [evidence span withheld — anchor status: not_found]
