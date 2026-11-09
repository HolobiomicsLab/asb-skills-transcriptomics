---
name: cell-type-clustering-ari-benchmarking
description: Use when reproducing the LUAD/HCC827-H1975-H2228 scRNA-seq clustering
  benchmark across three platforms with the 2267 common HVGs, or when comparing ARI
  of unsupervised batch-correction clustering to published values.
when_to_use_negative:
- The data are bulk RNA-seq where batch correction methods like ComBat and SVA apply,
  not scRNA-seq counts with unknown subtypes
- The experimental design is completely confounded (e.g., batch 1 measures cell types
  1 and 2, batch 2 measures types 3 and 4), where no method is applicable
- Cell-type labels are fully known a priori and the goal is only correction, not simultaneous
  clustering of unknown subtypes
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: BUSseq
  role: Bayesian hierarchical model (BUSseq_MCMC) that simultaneously corrects batch
    effects and clusters cell types; optimal K chosen by BIC; highest ARI (0.993 at
    K=3)
  repo: github.com/songfd2018/BUSseq
- name: LIGER
  role: Competing integration method whose clustering ARI (0.825) is compared against
    BUSseq
- name: Scanorama
  role: MNN-based batch-correction competitor benchmarked by ARI (0.637)
- name: Seurat
  role: Shared-variation integration competitor benchmarked by ARI (0.324)
- name: sc_mixology
  role: Source of the LUAD three-cell-line scRNA-seq count data (HCC827, H1975, H2228;
    CEL-seq2, Drop-seq, 10x Chromium)
  repo: https://github.com/LuyiTian/sc_mixology
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-16905-2
    title: Flexible experimental designs for valid single-cell RNA-sequencing experiments
      allowing batch effects correction
  source_content_hash: sha256:ab4724dc8fb6ff975198962fe095c5837f03c47ac19b7e9345b07346b3147767
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BUSseq
  - LIGER
  - MNN
  - Scanorama
  - Seurat
  - sc_mixology
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cell-type-clustering-ari-benchmarking

## Summary

Benchmark cell-type clustering accuracy of BUSseq against batch-effect-correction methods (LIGER, MNN, Scanorama, scVI, Seurat, ZINB-WaVE, CellAssign) on the LUAD three-cell-line sc_mixology dataset (CEL-seq2, 10x Chromium, Drop-seq), scoring adjusted Rand index against known cell-line identities.

## When to use

Use when reproducing the LUAD/HCC827-H1975-H2228 scRNA-seq clustering benchmark across three platforms with the 2267 common HVGs, or when comparing ARI of unsupervised batch-correction clustering to published values.

## When NOT to use

- The data are bulk RNA-seq where batch correction methods like ComBat and SVA apply, not scRNA-seq counts with unknown subtypes
- The experimental design is completely confounded (e.g., batch 1 measures cell types 1 and 2, batch 2 measures types 3 and 4), where no method is applicable
- Cell-type labels are fully known a priori and the goal is only correction, not simultaneous clustering of unknown subtypes

## Inputs

- LUAD cancer-cell-line scRNA-seq read-count matrices from sc_mixology (GSE118767): HCC827, H1975, H2228 across CEL-seq2, 10x Chromium and Drop-seq batches
- Multi-batch read-count matrix restricted to the 2,267 common highly variable genes
- Known cell-line identity labels per cell

## Outputs

- Per-cell cluster/cell-type labels from BUSseq and each competing method
- BIC values for candidate K (3-6) used to select the optimal number of cell types
- ari_comparison.csv: table of adjusted Rand index per method vs known cell-line labels

## How to apply

Download the LUAD cancer-cell-line read-count matrices from the sc_mixology repository (GSE118767) covering the HCC827, H1975 and H2228 cell lines assayed by CEL-seq2, Drop-seq and 10x Chromium, and assemble the multi-batch count matrix. Restrict analysis to the 2,267 highly variable genes common across batches. Run BUSseq_MCMC enumerating K from 3 to 6 in parallel (~2 minutes per K for 500 iterations) and choose the optimal K by minimum BIC, expecting four clusters. Refit with K fixed at 3 and compute ARI of inferred labels against known cell-line identities (target 0.993). Run LIGER, MNN, Scanorama, scVI, Seurat, ZINB-WaVE and semi-supervised CellAssign on the same HVG matrix, compute their ARIs (0.825, 0.650, 0.637, 0.429, 0.324, 0.398, 0.972 respectively), and collate into ari_comparison.csv, verifying BUSseq ranks highest among unsupervised methods.

## Related tools

- **BUSseq** (Bayesian hierarchical model (BUSseq_MCMC) that simultaneously corrects batch effects and clusters cell types; optimal K chosen by BIC; highest ARI (0.993 at K=3)) — github.com/songfd2018/BUSseq
- **LIGER** (Competing integration method whose clustering ARI (0.825) is compared against BUSseq)
- **Scanorama** (MNN-based batch-correction competitor benchmarked by ARI (0.637))
- **Seurat** (Shared-variation integration competitor benchmarked by ARI (0.324))
- **sc_mixology** (Source of the LUAD three-cell-line scRNA-seq count data (HCC827, H1975, H2228; CEL-seq2, Drop-seq, 10x Chromium)) — https://github.com/LuyiTian/sc_mixology

## Examples

```
library(devtools); install_github("songfd2018/BUSseq-Rpackage"); fit <- BUSseq_MCMC(ObservedData, n.celltypes=3, seed=123, n.iterations=500, n.cores=4)
```

## Evaluation signals

- BUSseq at K=3 yields ARI 0.993 against known cell-line identities, matching the published value within tolerance
- Competitor ARIs reproduce published values: LIGER 0.825, MNN 0.650, Scanorama 0.637, scVI 0.429, Seurat 0.324, ZINB-WaVE 0.398, CellAssign 0.972
- BIC comparison over K=3..6 selects four clusters for the full unsupervised fit
- BUSseq ranks highest among unsupervised methods in ari_comparison.csv
- Count matrix is restricted to exactly 2,267 common HVGs before fitting any method

## Limitations

- MCMC algorithms carry heavy computation load, though most steps of the BUSseq MCMC are parallelizable (~2 minutes per K for 500 iterations on multiple cores)
- MNN/Scanorama-based competitors require batch effects to be almost orthogonal to biological subspaces and much smaller than biological variations, assumptions that cannot be validated at the design stage
- A GPU/CUDA implementation (BUSseq_gpu) or C++ build requiring RngStreams may be needed for scale; C++ sources are tested only on Linux (Ubuntu 18.04, CentOS 7)

## Evidence

- [results] BUSseq achieves the highest clustering accuracy on the LUAD dataset with an ARI of 0.841, outperforming LIGER (0.825), MNN (0.650), Scanorama (0.637), scVI (0.429), Seurat (0.324), and ZINB-WaVE (0.398): [evidence span withheld — anchor status: not_found]
- [methods] To select the optimal K, we need to compare the BIC values for different Ks. In the vignettes, we enumerate K from 3 to 6.: "To select the optimal *K*, we need to compare the BIC values for different *K*s. In the vignettes, we enumerate *K* from 3 to 6."
- [readme] sc_mixology uses three human lung adenocarcinoma cell lines HCC827, H1975 and H2228, which were cultured separately, and then processed in three different ways.: [evidence span withheld — anchor status: external_doc]
- [methods] BUSseq can integrate read count matrices obtained from different scRNA-seq platforms: "BUSseq can integrate read count matrices obtained from different scRNA-seq platforms"
- [methods] install_github("songfd2018/BUSseq-Rpackage") # install BUSseq: "install_github("songfd2018/BUSseq-Rpackage") # install BUSseq"
