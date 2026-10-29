---
name: cross-method-integration-evaluation
description: Use when a multi-batch scRNA-seq dataset with known ground-truth cell
  labels (e.g., GSE118767 sc_mixology) must be integrated and clustered by several
  competing methods and their accuracies compared via ARI.
when_to_use_negative:
- The experimental design is completely confounded (each batch measures disjoint cell
  types) — 'no method is applicable'
- The data are bulk RNA-seq rather than single-cell counts, where ComBat/SVA are the
  appropriate correction tools
- Cell-type labels are known a priori and the goal is pure batch correction or differential
  expression rather than unsupervised clustering benchmarking
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: BUSseq
  role: Bayesian hierarchical model fit via BUSseq_MCMC; benchmark method whose clustering
    (K selected by BIC) is compared against all alternatives by ARI
  repo: github.com/songfd2018/BUSseq
- name: LIGER
  role: Competing integration method identifying shared variations across batches;
    ARI 0.825 on the LUAD dataset
- name: MNN
  role: Mutual-nearest-neighbor batch-correction competitor; ARI 0.650 on the LUAD
    dataset
- name: Scanorama
  role: MNN-based integration competitor; ARI 0.637 on the LUAD dataset
- name: Seurat
  role: Shared-variation integration competitor; ARI 0.324 on the LUAD dataset
- name: CellAssign
  role: Semi-supervised comparison method; ARI 0.972 vs BUSseq K=3 ARI 0.993
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-16905-2
    title: Flexible experimental designs for valid single-cell RNA-sequencing experiments
      allowing batch effects correction
  source_content_hash: sha256:c2df040a4f057d570af112c7718b426ed2f7473b58f80171d73e07471ff4dcfd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BUSseq
  - CellAssign
  - LIGER
  - MNN
  - Scanorama
  - Seurat
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# cross-method-integration-evaluation

## Summary

Benchmark multiple scRNA-seq batch-effect-correction and clustering methods (BUSseq, LIGER, MNN, Scanorama, scVI, Seurat, ZINB-WaVE, semi-supervised CellAssign) on the same multi-batch dataset and rank them by adjusted Rand index against known cell identities, as done on the LUAD three-cell-line dataset.

## When to use

Use when a multi-batch scRNA-seq dataset with known ground-truth cell labels (e.g., GSE118767 sc_mixology) must be integrated and clustered by several competing methods and their accuracies compared via ARI.

## When NOT to use

- The experimental design is completely confounded (each batch measures disjoint cell types) — 'no method is applicable'
- The data are bulk RNA-seq rather than single-cell counts, where ComBat/SVA are the appropriate correction tools
- Cell-type labels are known a priori and the goal is pure batch correction or differential expression rather than unsupervised clustering benchmarking

## Inputs

- Multi-batch scRNA-seq read-count matrix from GSE118767 sc_mixology (HCC827, H1975, H2228 via CEL-seq2, 10x Chromium, Drop-seq)
- Vector of 2,267 highly variable genes common across batches
- Known cell-line identity labels for ARI computation

## Outputs

- BUSseq cell-type cluster assignments at K=3 (ARI 0.993) and K=4 selected by BIC (ARI 0.841)
- Cluster labels and ARIs for LIGER, MNN, Scanorama, scVI, Seurat, ZINB-WaVE, and CellAssign
- ari_comparison.csv containing all ARI values in a single table

## How to apply

Download the LUAD cancer-cell-line count matrices (HCC827, H1975, H2228 assayed by CEL-seq2, 10x Chromium, Drop-seq from GSE118767) and restrict to the 2,267 common highly variable genes. Fit BUSseq with BUSseq_MCMC, enumerating K from 3 to 6 (~2 minutes per K for 500 iterations), select the optimal K by minimum BIC (expect K=4), then refit with K fixed at 3 and compute ARI against known cell-line identities (target 0.993). Run each competing method — LIGER (0.825), MNN (0.650), Scanorama (0.637), scVI (0.429), Seurat (0.324), ZINB-WaVE (0.398), and semi-supervised CellAssign (0.972) — on the identical HVG count matrix and compute ARIs against the same labels. Collate all ARIs into a single table (ari_comparison.csv) and check that BUSseq ranks highest among the unsupervised methods. The rationale is that BUSseq is a Bayesian hierarchical model that jointly corrects batch effects and clusters unknown cell types, so it can be fairly compared to each alternative on the same input and metric.

## Related tools

- **BUSseq** (Bayesian hierarchical model fit via BUSseq_MCMC; benchmark method whose clustering (K selected by BIC) is compared against all alternatives by ARI) — github.com/songfd2018/BUSseq
- **LIGER** (Competing integration method identifying shared variations across batches; ARI 0.825 on the LUAD dataset)
- **MNN** (Mutual-nearest-neighbor batch-correction competitor; ARI 0.650 on the LUAD dataset)
- **Scanorama** (MNN-based integration competitor; ARI 0.637 on the LUAD dataset)
- **Seurat** (Shared-variation integration competitor; ARI 0.324 on the LUAD dataset)
- **CellAssign** (Semi-supervised comparison method; ARI 0.972 vs BUSseq K=3 ARI 0.993)

## Examples

```
BUSseq_MCMC(ObservedData, n.celltypes=3, seed=123, n.cores=4, n.iterations=500, n.burnin=100, working_dir="./")
```

## Evaluation signals

- BUSseq at K=3 reproduces ARI 0.993 against known cell-line identities (vs CellAssign 0.972)
- All reproduced ARIs match published values within tolerance: LIGER 0.825, MNN 0.650, Scanorama 0.637, scVI 0.429, Seurat 0.324, ZINB-WaVE 0.398
- BIC-based K selection over K=3..6 yields the expected four clusters covering the three cell lines
- BUSseq ranks highest among the unsupervised methods in the collated ari_comparison.csv
- Every method is evaluated on the same 2,267-HVG count matrix, so the comparison table is complete with one ARI per method

## Limitations

- MNN/Scanorama require batch effects to be almost orthogonal to biological subspaces and much smaller than biological variation — assumptions that cannot be validated at the design stage
- MetaSparseKmeans-style methods require all subtypes present in each batch, too restrictive for many scRNA-seq experiments
- MCMC inference is computationally heavy (though most steps are parallelizable); C++ source supports Linux only (tested on Ubuntu 18.04 and CentOS 7, R 3.6.0)

## Evidence

- [intro] The mutual-nearest-neighbor (MNN) based approaches, including MNN14 and Scanorama15, allow each batch to contain some but not all cell types: "The mutual-nearest-neighbor (MNN) based approaches, including MNN14 and Scanorama15, allow each batch to contain some but not all cell types"
- [methods] To select the optimal *K*, we need to compare the BIC values for different *K*s. In the vignettes, we enumerate *K* from 3 to 6.: "To select the optimal *K*, we need to compare the BIC values for different *K*s. In the vignettes, we enumerate *K* from 3 to 6."
- [methods] BUSseq can integrate read count matrices obtained from different scRNA-seq platforms: "BUSseq can integrate read count matrices obtained from different scRNA-seq platforms"
- [readme] the corrected value can be used for downstream analysis as if all cells were sequenced in a single batch: "the corrected value can be used for downstream analysis as if all cells were sequenced in a single batch"
