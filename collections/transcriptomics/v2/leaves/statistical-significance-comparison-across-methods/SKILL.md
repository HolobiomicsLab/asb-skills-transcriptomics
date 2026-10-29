---
name: statistical-significance-comparison-across-methods
description: Use when a new differential abundance method must be validated against
  established competitors on shared single-cell datasets, requiring comparable per-cell-type
  DA statistics and captured-cell-fraction metrics.
when_to_use_negative:
- When the goal is a single method's DA calls on one dataset without a head-to-head
  benchmark — no comparison baseline exists
- When datasets are not identically preprocessed across methods, since preprocessing
  differences would confound the significance comparison
- When the competitor of interest is miloDE — the authors explicitly did not benchmark
  against it and flag this as an open question
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: ELVAR
  role: Attribute-aware EVA clustering + negative-binomial-regression DA testing being
    benchmarked (Eva_partitions, SelOptAlpha, ProcessEVA, DoDA)
  repo: https://github.com/aet21/ELVAR
- name: DA-seq
  role: Non-clustering-based DA-testing competitor run on the same datasets to obtain
    comparable DA z-statistics
- name: Milo
  role: Local-neighborhood-based DA-testing competitor run on the same datasets for
    sensitivity comparison
- name: Seurat
  role: 'Identical preprocessing across methods: FindVariableFeatures (vst), ScaleData,
    RunPCA, FindNeighbors (k.param=20, dims=1:8)'
- name: R
  role: Language in which the whole benchmarking pipeline is implemented (ELVAR is
    an open-source R-package)
  repo: https://github.com/aet21/ELVAR
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-39017-z
    title: Cell-attribute aware community detection improves differential abundance
      testing from single-cell RNA-Seq data
  source_content_hash: sha256:113c4e8a331fd8767dd63db8ba87f7d299e368071109938c040360379d6196c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aet21/ELVAR
  tools:
  - DA-seq
  - ELVAR
  - EVA
  - MASS
  - Matrix
  - Milo
  - R
  - Seurat
  - dplyr
  - igraph
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# statistical-significance-comparison-across-methods

## Summary

Benchmark a new DA-testing method (e.g. ELVAR) head-to-head against clustering-free competitors (DA-seq, Milo) by running all methods on the same scRNA-Seq datasets and comparing DA z-statistic distributions and the fraction of condition-enriched cells captured, as done for aging, Covid-19 and colon cancer benchmarks.

## When to use

Use when a new differential abundance method must be validated against established competitors on shared single-cell datasets, requiring comparable per-cell-type DA statistics and captured-cell-fraction metrics.

## When NOT to use

- When the goal is a single method's DA calls on one dataset without a head-to-head benchmark — no comparison baseline exists
- When datasets are not identically preprocessed across methods, since preprocessing differences would confound the significance comparison
- When the competitor of interest is miloDE — the authors explicitly did not benchmark against it and flag this as an open question

## Inputs

- scRNA-Seq benchmark datasets as Seurat objects (e.g. Tabula Muris Senis lung CD4+ T-cells, Covid-19, colon cancer) from the aet21/ELVAR repository
- Cell attribute/phenotype metadata (age, disease stage, sample/batch labels) in the Seurat meta.data slot
- Cell-cell nearest-neighbour state-manifold graph (FindNeighbors, k.param=20, dims=1:8)
- Per-method DA test results (z-statistics per cell type/community)

## Outputs

- Comparative DA z-statistic violin plots per dataset and method (Fig. 5-style, e.g. fig5_elvar_benchmark_violins.png)
- Table of fraction of condition-enriched cells captured per method and dataset
- Per-cell-type DA significance comparisons (ELVAR vs DA-seq vs Milo)

## How to apply

Run every method on identically preprocessed Seurat objects so comparisons reflect the method, not preprocessing: for ELVAR, perform EVA clustering via Eva_partitions over alpha 0.1–0.9 with ~100 runs per alpha, select the optimal alpha with SelOptAlpha, extract attribute-enriched clusters with ProcessEVA, and run negative-binomial-regression DA testing with DoDA (e.g. varDA='naive', varREP='mouse'); run adapted DA-seq and Milo on the same cell-cell nearest-neighbour state-manifold (FindNeighbors, k.param=20, dims=1:8) to obtain comparable DA z-statistics. For each method and dataset (e.g. Tabula Muris Senis lung CD4+ T-cells, Covid-19 PBMC, colon cancer), compute the fraction of condition-enriched cells captured and the associated statistical significance of DA shifts such as the age-related naive-to-mature CD4+ T-cell shift. Summarise per-method z-statistics as violin plots and tabulate captured-cell fractions, judging the better method by stronger significance and higher captured fraction, as ELVAR showed over Milo and DA-seq.

## Related tools

- **ELVAR** (Attribute-aware EVA clustering + negative-binomial-regression DA testing being benchmarked (Eva_partitions, SelOptAlpha, ProcessEVA, DoDA)) — https://github.com/aet21/ELVAR
- **DA-seq** (Non-clustering-based DA-testing competitor run on the same datasets to obtain comparable DA z-statistics)
- **Milo** (Local-neighborhood-based DA-testing competitor run on the same datasets for sensitivity comparison)
- **Seurat** (Identical preprocessing across methods: FindVariableFeatures (vst), ScaleData, RunPCA, FindNeighbors (k.param=20, dims=1:8))
- **R** (Language in which the whole benchmarking pipeline is implemented (ELVAR is an open-source R-package)) — https://github.com/aet21/ELVAR

## Examples

```
library(ELVAR); eva.o <- Eva_partitions(gr.o, resolution=1, alpha=0.8, Vattr.name="age"); selAlpha.o <- SelOptAlpha(nC.m,nc0,pur.m,mod.m,alpha.v=seq(0.1,0.9,0.1)); sigcl.o <- ProcessEVA(eva.o,seu.o,attrName="age"); da.o <- DoDA(seu.o,sigcl.o,varDA="naive",varREP="mouse")
```

## Evaluation signals

- ELVAR z-statistics exceed those of DA-seq and Milo for the benchmarked shifts (CD4+ T-cell, alveolar macrophage, Covid-19, colon enterocyte), reproducing Fig. 5
- ELVAR's fraction of condition-enriched cells captured is higher than Milo's and DA-seq's across the aging, Covid-19 and colon settings
- All methods were run on the same Seurat objects with identical FindNeighbors parameters (k.param=20, dims=1:8), so differences are attributable to the DA method
- ELVAR's DA calls come from its full pipeline: optimal alpha selected via SelOptAlpha over alpha.v = seq(0.1, 0.9, 0.1) with ~100 runs per alpha
- Violin plots and captured-cell-fraction tables are produced for every method-dataset combination

## Limitations

- ELVAR has increased computational complexity and runtimes compared to Milo or the Louvain-analog (deemed not practically prohibitive); selecting an optimal alpha may require ~900 runs (9 alpha values x 100 runs)
- ELVAR is more sensitive to false positives in the clustering attribute (sample/phenotype label), which can affect the quality of inferred communities
- No benchmark against miloDE was performed; the authors flag this as an open question
- ELVAR's improvement over its Louvain-analog is more pronounced in studies with weaker clustering structure and less noticeable with stronger clustering

## Evidence

- [results] ELVAR attained stronger levels of statistical significance than both DA-seq and Milo: [evidence span withheld — anchor status: not_found]
- [discussion] Higher sensitivity than Milo and DA-seq: "ELVAR displayed higher sensitivity than Milo and DA-seq to detect biologically important and plausible DA-shifts, such as the age-related shift from naïve to mature Cd4+ T-cells in lung tissue"
- [methods] DoDA negative-binomial regression call: "da.o <- DoDA(seu.o,sigcl.o,varDA="naive",varREP="mouse");"
- [discussion] Alpha selection across nine values: "to identify an optimal a parameter value may require a total of 100 runs for each of nine a parameter values, so a total of 900 runs"
- [abstract] Benchmark scope statement: "Using simulated and real single-cell and single-nucleus RNA-Seq datasets, we benchmark ELVAR against an analogous algorithm that uses Louvain for clustering, as well as local neighborhood-based"
- [readme] ELVAR open-source R-package: "ELVAR is an R-package for differential abundance (DA) testing of cell-types in single-cell RNA-Seq data."
