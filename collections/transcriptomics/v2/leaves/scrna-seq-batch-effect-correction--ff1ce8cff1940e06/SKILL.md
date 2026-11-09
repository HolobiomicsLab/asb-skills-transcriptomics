---
name: scrna-seq-batch-effect-correction
description: Use when a multi-batch scRNA-seq count matrix (e.g., three cell lines
  across CEL-seq2, 10x Chromium, Drop-seq) has unknown cell types and batch effects
  must be corrected while clustering cells and benchmarking ARI against alternative
  integration methods.
when_to_use_negative:
- The experimental design is completely confounded (e.g., batch 1 measures cell types
  1 and 2, batch 2 measures cell types 3 and 4) — no batch-correction method, including
  BUSseq, is applicable
- Input is already normalized or batch-corrected expression data; BUSseq requires
  raw read counts to model count nature, overdispersion, dropouts, and cell-specific
  size factors
- The goal is bulk-sample batch adjustment, where ComBat or SVA are the classic appropriate
  tools rather than a single-cell hierarchical model
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0203
tools:
- name: BUSseq
  role: Primary Bayesian hierarchical model that simultaneously corrects batch effects,
    clusters unknown cell types, imputes dropouts, and selects K via BIC
  repo: https://github.com/songfd2018/BUSseq
- name: LIGER
  role: Competing shared-variation integration method benchmarked by ARI against BUSseq
    clustering
- name: Scanorama
  role: Mutual-nearest-neighbor integration method benchmarked by ARI
- name: Seurat
  role: Competing integration/clustering method benchmarked by ARI
- name: CellAssign
  role: Semi-supervised comparator; BUSseq at K=3 (ARI 0.993) surpasses it (ARI 0.972)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-16905-2
    title: Flexible experimental designs for valid single-cell RNA-sequencing experiments
      allowing batch effects correction
  source_content_hash: sha256:8fe6ee2cd9dbcd091f1ab0b501960abcffd1bc9f3ec52aac485536c350a9d801
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/songfd2018/BUSseq
  tools:
  - BUSseq
  - CellAssign
  - LIGER
  - MNN
  - Scanorama
  - Seurat
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# scrna-seq-batch-effect-correction

## Summary

Fit BUSseq, an interpretable Bayesian hierarchical model, to simultaneously correct batch effects, cluster unknown cell types, impute dropout-caused missing data, and detect differentially expressed genes in multi-batch scRNA-seq count data, and benchmark the clustering against competing integration methods (LIGER, MNN, Scanorama, scVI, Seurat, ZINB-WaVE, CellAssign) via adjusted Rand index.

## When to use

Use when a multi-batch scRNA-seq count matrix (e.g., three cell lines across CEL-seq2, 10x Chromium, Drop-seq) has unknown cell types and batch effects must be corrected while clustering cells and benchmarking ARI against alternative integration methods.

## When NOT to use

- The experimental design is completely confounded (e.g., batch 1 measures cell types 1 and 2, batch 2 measures cell types 3 and 4) — no batch-correction method, including BUSseq, is applicable
- Input is already normalized or batch-corrected expression data; BUSseq requires raw read counts to model count nature, overdispersion, dropouts, and cell-specific size factors
- The goal is bulk-sample batch adjustment, where ComBat or SVA are the classic appropriate tools rather than a single-cell hierarchical model

## Inputs

- Multi-batch scRNA-seq raw read count matrix (genes x cells) spanning different protocols or platforms (e.g., CEL-seq2, 10x Chromium, Drop-seq from GSE118767)
- Vector of 2,267 common highly variable genes shared across batches
- Batch dimension information file (number of batches and cells per batch)
- Optional known cell-line/cell-type labels for ARI evaluation

## Outputs

- Inferred cell-type (subtype) labels per cell from BUSseq_MCMC posterior inference
- Corrected and imputed read count matrices from the fitted BUSseqfits object
- BIC values per candidate K used for optimal-K selection
- ARI comparison table (e.g., ari_comparison.csv) of BUSseq versus LIGER, MNN, Scanorama, scVI, Seurat, ZINB-WaVE, and CellAssign

## How to apply

Download the raw count matrices (e.g., LUAD HCC827/H1975/H2228 from sc_mixology GSE118767) and restrict to the common highly variable genes (2,267 genes in the benchmark). Install BUSseq from GitHub (songfd2018/BUSseq-Rpackage; the C++ BUSseq-1.0 build requires RngStreams and R 3.6.0 on Linux) and fit with BUSseq_MCMC, enumerating K from 3 to 6 in parallel (~2 minutes per K for 500 iterations on 4 cores) and selecting the optimal K by minimum BIC. Refit with the chosen K (or K fixed at 3 for the supervised-like comparison) and compute the adjusted Rand index of inferred cell-type labels against known identities; expect ARI 0.993 for BUSseq K=3 versus CellAssign 0.972, LIGER 0.825, MNN 0.650, Scanorama 0.637, scVI 0.429, ZINB-WaVE 0.398, and Seurat 0.324 on the LUAD data. Then use corrected_read_counts/imputed_read_counts from the fitted object for downstream single-batch-style analysis, and write the ARI table (e.g., ari_comparison.csv) for validation. The rationale is that BUSseq models count nature, overdispersion, dropout events, and cell-specific size factors simultaneously, avoiding the multi-stage pipelines whose errors propagate into batch correction.

## Related tools

- **BUSseq** (Primary Bayesian hierarchical model that simultaneously corrects batch effects, clusters unknown cell types, imputes dropouts, and selects K via BIC) — https://github.com/songfd2018/BUSseq
- **LIGER** (Competing shared-variation integration method benchmarked by ARI against BUSseq clustering)
- **Scanorama** (Mutual-nearest-neighbor integration method benchmarked by ARI)
- **Seurat** (Competing integration/clustering method benchmarked by ARI)
- **CellAssign** (Semi-supervised comparator; BUSseq at K=3 (ARI 0.993) surpasses it (ARI 0.972))

## Examples

```
install_github("songfd2018/BUSseq-Rpackage")  # then: BUSseq_MCMC(ObservedData, n.celltypes=3, seed, n.cores, n.iterations, n.burnin, n.output, working_dir)
```

## Evaluation signals

- BUSseq K=3 clustering on the LUAD data achieves ARI 0.993 against known cell-line identities, and ranks highest among unsupervised methods (LIGER 0.825, MNN 0.650, Scanorama 0.637, scVI 0.429, Seurat 0.324, ZINB-WaVE 0.398)
- BIC values across enumerated K (3 to 6) identify the correct number of clusters (four clusters covering the three cell lines)
- Reproduced ARIs of all competing methods match the published values within tolerance in ari_comparison.csv
- Corrected read counts support downstream analysis as if all cells were sequenced in a single batch
- Optimal K selection expects four clusters (three cell lines plus an extra subtype) on the LUAD benchmark

## Limitations

- MCMC algorithms carry heavy computation load, though most BUSseq MCMC steps are parallelizable across cores
- Existing MNN/Scanorama-style approaches require batch effects to be almost orthogonal to biological subspaces and much smaller than biological variations — assumptions that cannot be validated at the design stage
- Completely confounded designs admit no applicable batch-correction method
- The C++ BUSseq-1.0 source supports Linux only (tested on Ubuntu 18.04 and CentOS 7, R 3.6.0)

## Evidence

- [abstract] BUSseq can simultaneously correct batch effects, cluster cell types, impute dropout-caused missing data, and detect differentially expressed genes without preliminary normalization: [evidence span withheld — anchor status: not_found]
- [methods] To select the optimal K, we need to compare the BIC values for different Ks. In the vignettes, we enumerate K from 3 to 6.: "To select the optimal *K*, we need to compare the BIC values for different *K*s. In the vignettes, we enumerate *K* from 3 to 6."
- [intro] The mutual-nearest-neighbor (MNN) based approaches, including MNN14 and Scanorama15, allow each batch to contain some but not all cell types: "The mutual-nearest-neighbor (MNN) based approaches, including MNN14 and Scanorama15, allow each batch to contain some but not all cell types"
- [readme] BUSseq can integrate read count matrices obtained from different scRNA-seq platforms and allow cell types to be measured in some but not all of the batches as long as the experimental design fulfills the conditions.: "BUSseq can integrate read count matrices obtained from different scRNA-seq platforms and allow cell types to be measured in some but not all of the batches"
- [discussion] most steps of the MCMC algorithm for BUSseq are parallelizable (Supplementary Note 12).: "most steps of the MCMC algorithm for BUSseq are parallelizable (Supplementary Note 12)."
