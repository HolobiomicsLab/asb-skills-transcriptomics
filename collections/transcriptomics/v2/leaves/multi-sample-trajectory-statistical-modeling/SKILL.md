---
name: multi-sample-trajectory-statistical-modeling
description: Use when scRNA-seq data from multiple biological samples per condition
  need pseudotemporal trajectory comparison across covariates (e.g. mild vs moderate
  COVID-19), and naive single-sample pseudotime methods would ignore sample-to-sample
  variation.
when_to_use_negative:
- Lamian modules 1 and 2 do not support Monocle2, Monocle3, or Slingshot trajectories
  — the pipeline assumes TSCAN pseudotime; tailoring to those methods is deferred
  to future work.
- Single-sample pseudotime analyses where cross-sample variability cannot be modeled
  — Lamian's key advantage (sample-level inference) does not apply.
- Data types outside gene expression, e.g. single-cell ATAC-seq — the statistical
  model extension is explicitly future research.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0128
tools:
- name: Lamian
  role: 'Core framework: lamian_test (TDE/XDE via permutation-based likelihood ratio
    test), cellPropTest (TCD/XCD), infer_tree_structure, evaluate_uncertainty, clusterGene'
  repo: https://github.com/Winnie09/Lamian
- name: TSCAN
  role: Default pseudotime construction via cluster-based minimum spanning tree; install
    from zji90/TSCAN fork
  repo: https://github.com/zji90/TSCAN
- name: Seurat
  role: Sample harmonization by embedding cells into a common low-dimensional space
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-42841-y
    title: A statistical framework for differential pseudotime analysis with multiple
      single-cell RNA-seq samples
  source_content_hash: sha256:58ec9d29ac845e8af3bdc69c851a7438a400c47ff7fd5478dfb5eb39e6dc4dda
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Winnie09/Lamian
  tools:
  - Lamian
  - Seurat
  - TSCAN
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0128
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# multi-sample-trajectory-statistical-modeling

## Summary

Apply the Lamian statistical framework for differential multi-sample pseudotime analysis of scRNA-seq data: after sample harmonization with Seurat and pseudotime construction with TSCAN's cluster-based minimum spanning tree, test for topology, cell abundance (TCD/XCD), and gene expression (TDE/XDE) changes across sample covariates such as disease severity while accounting for cross-sample variability.

## When to use

Use when scRNA-seq data from multiple biological samples per condition need pseudotemporal trajectory comparison across covariates (e.g. mild vs moderate COVID-19), and naive single-sample pseudotime methods would ignore sample-to-sample variation.

## When NOT to use

- Lamian modules 1 and 2 do not support Monocle2, Monocle3, or Slingshot trajectories — the pipeline assumes TSCAN pseudotime; tailoring to those methods is deferred to future work.
- Single-sample pseudotime analyses where cross-sample variability cannot be modeled — Lamian's key advantage (sample-level inference) does not apply.
- Data types outside gene expression, e.g. single-cell ATAC-seq — the statistical model extension is explicitly future research.

## Inputs

- Raw 10x Genomics scRNA-seq count matrices from multiple biological samples with covariates (e.g. E-MTAB-9357: 256 PBMC samples from 139 COVID-19 patients)
- Seurat-embedded common low-dimensional representation of cells across samples
- TSCAN-constructed pseudotime and trajectory tree (Lamian inputs: expdata, hca_bm_saver, hca_bm_pca, hca_bm_cellanno structures)
- Sample covariate table (e.g. disease severity, age, sex)

## Outputs

- TDE and XDE gene lists with FDR.overall < 0.05, partitioned into expression clusters (e.g. 2195 TDE genes in 5 clusters; 1315 XDE genes in 14 clusters for COVID CD8+ T cells)
- TCD/XCD cell-abundance test p-values along pseudotime (e.g. TCD p < 2.22e-308, XCD p = 1.38e-61)
- Differential topology test results per branch with branch uncertainty (detection rate over 1000 bootstrap samples)
- GO enrichment terms per gene cluster

## How to apply

Filter low-quality cells (COVID data: <2,000 reads, <500 expressed genes, or >10% mitochondrial reads; HCA-BM: <5,000 reads, <1,000 expressed genes, >10% mito) and drop samples with fewer than 500 cells or fewer than 100 CD8+ T cells. Harmonize samples by embedding cells into a common low-dimensional space with Seurat, then construct pseudotime with TSCAN's cluster-based minimum spanning tree (Lamian's default trajectory method). Fit Lamian models via permutation-based likelihood ratio tests: lamian_test with test.type='time' for TDE and test.type='variable', testvar=2 for XDE (permuiter >= 100), and cellPropTest with test.type='Time' (TCD) or 'Variable' (XCD) on pseudotime divided into 100 equal-length intervals; call significance at FDR.overall < 0.05. Characterize branch uncertainty via detection rate in 1000 bootstrap samples and cluster significant genes with clusterGene followed by GO enrichment. The rationale is that Lamian draws inference after accounting for cross-sample variability, reducing sample-specific false discoveries that existing methods (condiments, Phenopath, tradeSeq) do not control.

## Related tools

- **Lamian** (Core framework: lamian_test (TDE/XDE via permutation-based likelihood ratio test), cellPropTest (TCD/XCD), infer_tree_structure, evaluate_uncertainty, clusterGene) — https://github.com/Winnie09/Lamian
- **TSCAN** (Default pseudotime construction via cluster-based minimum spanning tree; install from zji90/TSCAN fork) — https://github.com/zji90/TSCAN
- **Seurat** (Sample harmonization by embedding cells into a common low-dimensional space)

## Examples

```
devtools::install_github("zji90/TSCAN"); devtools::install_github("Winnie09/Lamian"); res <- lamian_test(pca, cellanno, design, pseudotime, test.type='variable', testvar=2, permuiter=100)
```

## Evaluation signals

- Reproduce the COVID-19 CD8+ T cell benchmark: 2195 TDE genes in 5 clusters and 1315 XDE genes in 14 clusters at FDR.overall < 0.05, with TBX21/ZEB2/interferon-gene patterns
- Reproduce reported cell-abundance statistics: TCD p-value < 2.22e-308 and XCD p-value = 1.38e-61
- Lamian's XDE gene overlap between two random partitions of COVID samples should exceed competing methods (condiments, monocle2TrajTestCorr, limma, tradeSeq), confirming reproducible rankings
- FDR control check: on multi-sample data, false discovery rates should be appropriately controlled because inference accounts for cross-sample variability
- Installation check: updated TSCAN must be installed from Github (zji90/TSCAN) after removing any old version, per the README

## Limitations

- A systematic comparison of harmonization methods is beyond the scope of the study.
- Lamian modules 1 and 2 do not support Monocle2, Monocle3, or slingshot trajectories; tailoring for scalability/interoperability is future work.
- Simultaneously accounting for pseudotime inference uncertainty, gene expression and cell abundance uncertainty, and cross-sample/cross-cell variability remains an open research topic.
- Extension to other data types such as single-cell ATAC-seq is deferred to future research.
- EM algorithm details for fitting the model are in Supplementary Notes; simulations for TDE, TCD, and XCD detection are likewise deferred to Supplementary Notes.

## Evidence

- [discussion] permutation-based likelihood ratio test: "To conduct a hypothesis test comparing two models, we use a permutation-based likelihood ratio test."
- [discussion] TSCAN default pseudotime: "In Lamian, TSCAN is used as the default method to construct pseudotime due to its flexibility and scalability."
- [discussion] 100 equal-length intervals: "we divide the pseudotime from 0 to its maximum into 100 consecutive intervals of equal lengths."
- [abstract] cross-sample variability: "Lamian draws statistical inference after accounting for cross-sample variability and hence substantially reduces sample-specific false"
- [discussion] Seurat harmonization: "For sample harmonization, we used Seurat32 to embed cells into a common low-dimensional space."
- [discussion] branch uncertainty via bootstrap: "For each of the branch, we characterize its uncertainty using its detection rate in 1000 bootstrap samples."
- [discussion] COVID cell filters: "We filtered out cells with fewer than 2,000 reads or 500 expressed genes or more than 10% mitochondrial reads."
- [readme] GitHub install: [evidence span withheld — anchor status: external_doc]
- [discussion] E-MTAB-9357 dataset: "The raw count matrices of 256 PBMC 10x Genomics scRNA-seq samples from 139 COVID-19 patients were downloaded from E-MTAB-935743."
