---
name: covid-pbmc-t-cell-subset-annotation
description: Use when analyzing 10x Genomics PBMC scRNA-seq from COVID-19 patients
  (e.g. accession E-MTAB-9357) to subset naive/CD8+ T cells, construct pseudotime,
  and detect TDE/XDE gene expression or TCD/XCD cell-proportion changes between mild
  and moderate severity groups.
when_to_use_negative:
- Data is a single biological sample per condition — Lamian's cross-sample variability
  modeling requires multiple samples per condition, unlike condiments which assumes
  one sample per condition
- Trajectories were inferred with Monocle2, Monocle3, or Slingshot — Lamian modules
  1 and 2 do not support these trajectory types
- The analysis goal is a systematic comparison of sample-harmonization methods, which
  the authors explicitly place beyond the scope of the study
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: Lamian
  role: 'Statistical framework for differential multi-sample pseudotime analysis:
    lamian_test for TDE/XDE, cellPropTest for TCD/XCD, clusterGene for clustering
    significant genes'
  repo: https://github.com/Winnie09/Lamian
- name: Seurat
  role: Sample harmonization by embedding cells from all COVID-19 samples into a common
    low-dimensional space
- name: TSCAN
  role: Default method to construct pseudotime via cluster-based minimum spanning
    tree; install the updated GitHub version (zji90/TSCAN)
  repo: https://github.com/zji90/TSCAN
- name: scRNA-seq
  role: 'Input data modality: 10x Genomics single-cell RNA-sequencing of PBMCs'
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-42841-y
    title: A statistical framework for differential pseudotime analysis with multiple
      single-cell RNA-seq samples
  source_content_hash: sha256:af824dd9da1448cb8f65835a32c988ed8859f411fe44012a4f13c79075bd0bfa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Winnie09/Lamian
  tools:
  - Lamian
  - Seurat
  - TSCAN
  - scRNA-seq
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# covid-pbmc-t-cell-subset-annotation

## Summary

A workflow for isolating and characterizing CD8+ T cell subsets from COVID-19 PBMC scRNA-seq samples (E-MTAB-9357) and testing for differential pseudotime dynamics between disease severity levels using Lamian. It combines cell/sample quality filtering, Seurat-based sample harmonization, TSCAN pseudotime, and Lamian TDE/XDE/TCD/XCD tests.

## When to use

Use when analyzing 10x Genomics PBMC scRNA-seq from COVID-19 patients (e.g. accession E-MTAB-9357) to subset naive/CD8+ T cells, construct pseudotime, and detect TDE/XDE gene expression or TCD/XCD cell-proportion changes between mild and moderate severity groups.

## When NOT to use

- Data is a single biological sample per condition — Lamian's cross-sample variability modeling requires multiple samples per condition, unlike condiments which assumes one sample per condition
- Trajectories were inferred with Monocle2, Monocle3, or Slingshot — Lamian modules 1 and 2 do not support these trajectory types
- The analysis goal is a systematic comparison of sample-harmonization methods, which the authors explicitly place beyond the scope of the study

## Inputs

- Raw 10x Genomics PBMC scRNA-seq count matrices from accession E-MTAB-9357 (256 samples, 139 COVID-19 patients)
- Sample-level metadata assigning disease severity (mild vs moderate) and other covariates
- Cell-level annotation identifying naive/CD8+ T cells for subsetting

## Outputs

- Filtered CD8+ T cell expression matrix with harmonized low-dimensional embeddings
- TSCAN pseudotime trajectories per sample with branch uncertainty (bootstrap detection rates)
- TDE and XDE gene rankings with FDR.overall values (e.g. 2195 TDE and 1315 XDE genes for the COVID severity analysis)
- TCD and XCD test p-values (e.g. TCD p < 2.22e-308, XCD p = 1.38e-61)
- Gene clusters with GO enrichment per cluster (e.g. TBX21/ZEB2/interferon-gene patterns)

## How to apply

Download raw 10x count matrices from E-MTAB-9357 (256 PBMC samples from 139 COVID-19 patients) and filter cells with fewer than 2,000 reads, fewer than 500 expressed genes, or more than 10% mitochondrial reads; then drop samples with fewer than 500 cells and samples with fewer than 100 CD8+ T cells before subsetting the naive/CD8+ T cells. Harmonize samples by embedding cells into a common low-dimensional space with Seurat, and construct pseudotime with TSCAN's cluster-based minimum spanning tree. Run Lamian lamian_test with test.type='time' (TDE) and test.type='variable', testvar=2 (XDE) using the permutation-based likelihood ratio test (permuiter=100 or higher), and cellPropTest with test.type='Time' (TCD) and 'Variable' (XCD) on pseudotime divided into 100 equal-length intervals. Cluster significant genes with clusterGene, compute GO enrichment per cluster, and call TDE/XDE genes at FDR.overall < 0.05. Correct application is judged by reproducing the reported severity signals (e.g. 2195 TDE genes in 5 clusters, 1315 XDE genes in 14 clusters, TCD p < 2.22e-308, XCD p = 1.38e-61) and the TBX21/ZEB2/interferon gene patterns.

## Related tools

- **Lamian** (Statistical framework for differential multi-sample pseudotime analysis: lamian_test for TDE/XDE, cellPropTest for TCD/XCD, clusterGene for clustering significant genes) — https://github.com/Winnie09/Lamian
- **Seurat** (Sample harmonization by embedding cells from all COVID-19 samples into a common low-dimensional space)
- **TSCAN** (Default method to construct pseudotime via cluster-based minimum spanning tree; install the updated GitHub version (zji90/TSCAN)) — https://github.com/zji90/TSCAN
- **scRNA-seq** (Input data modality: 10x Genomics single-cell RNA-sequencing of PBMCs)

## Examples

```
res <- lamian_test(pca = pca, cellanno = cellanno, design = design, pseudotime = pt, type = 2, test.type = 'variable', testvar = 2, permuiter = 100)
```

## Evaluation signals

- Reproduce 2195 TDE genes in 5 clusters and 1315 XDE genes in 14 clusters at FDR.overall < 0.05 for the COVID-19 severity analysis
- TCD p-value < 2.22e-308 and XCD p-value = 1.38e-61 as reported for the COVID severity analysis
- Recovered gene clusters show the reported TBX21/ZEB2/interferon-gene expression patterns
- Lamian XDE gene rankings show the highest overlap proportion between two random partitions of samples versus condiments, monocle2TrajTestCorr, limma, and tradeSeq
- Post-filter dataset contains no cells with <2,000 reads, <500 expressed genes, or >10% mitochondrial reads, and no samples with <500 cells or <100 CD8+ T cells

## Limitations

- A systematic comparison of harmonization methods is beyond the scope of the study
- Lamian modules 1 and 2 do not support Monocle2, Monocle3, or Slingshot trajectories; interoperability improvements are deferred to future work
- Simultaneously accounting for pseudotime inference uncertainty, gene expression and cell abundance uncertainty, and cross-sample/cross-cell variability remains an open research topic
- Extension of the Lamian statistical model to other data types such as single-cell ATAC-seq is deferred to future research

## Evidence

- [discussion] Filter cells with fewer than 2,000 reads or 500 expressed genes or more than 10% mitochondrial reads (COVID): "We filtered out cells with fewer than 2,000 reads or 500 expressed genes or more than 10% mitochondrial reads."
- [discussion] Filter samples with fewer than 100 CD8+ T cells (COVID): "Samples with fewer than 100 CD8+ T cells were filtered out."
- [discussion] Sample harmonization and TSCAN pseudotime construction: "For sample harmonization, we used Seurat32 to embed cells into a common low-dimensional space."
- [discussion] TSCAN is Lamian's default pseudotime method: "In Lamian, TSCAN is used as the default method to construct pseudotime due to its flexibility and scalability."
- [discussion] Permutation-based likelihood ratio test for model comparison: "To conduct a hypothesis test comparing two models, we use a permutation-based likelihood ratio test."
- [discussion] Cell density modeling in 100 pseudotime intervals: "we divide the pseudotime from 0 to its maximum into 100 consecutive intervals of equal lengths."
- [discussion] Dataset provenance from E-MTAB-9357: "The raw count matrices of 256 PBMC 10x Genomics scRNA-seq samples from 139 COVID-19 patients were downloaded from E-MTAB-935743."
- [abstract] Lamian controls FDR by accounting for cross-sample variability: "Unlike existing methods that ignore sample variability, Lamian draws statistical inference after accounting for cross-sample variability and hence substantially reduces sample-specific false"
- [readme] Installation requires updated TSCAN from GitHub: "## download the most updated TSCAN from Github
devtools::install_github("zji90/TSCAN")"
