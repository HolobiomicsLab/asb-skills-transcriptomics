---
name: violin-plot-benchmark-visualization
description: Use when benchmarking multiple DA-testing methods on scRNA-Seq data and
  you need to display the full distribution of DA z-statistics per method/dataset
  to compare sensitivity.
when_to_use_negative:
- When only point estimates or single p-values per method are available rather than
  distributions of z-statistics — a violin plot would convey false spread
- When comparing methods on different datasets or differently preprocessed inputs,
  since the comparison assumes identical Seurat processing (vst variable features,
  ScaleData, RunPCA, FindNeighbors k.param=20) per dataset
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0091
tools:
- name: ELVAR
  role: Source of the method-under-test DA z-statistics via its DoDA negative-binomial-regression
    step
  repo: https://github.com/aet21/ELVAR
- name: Seurat
  role: Standardizes preprocessing (FindVariableFeatures vst, ScaleData, RunPCA, FindNeighbors
    k.param=20) so all three methods consume identically prepared inputs
- name: R
  role: Implementation language for the pipeline and for rendering the comparative
    violin plots
- name: dplyr
  role: Tidying and aggregating z-statistics into long format for plotting
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-39017-z
    title: Cell-attribute aware community detection improves differential abundance
      testing from single-cell RNA-Seq data
  source_content_hash: sha256:427d894d2d0635282c1a6e67fbc1b6f400a22f36cfc0e64cd9bb0ab616643e74
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aet21/ELVAR
  tools:
  - ELVAR
  - EVA
  - MASS
  - Matrix
  - R
  - Seurat
  - dplyr
  - igraph
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# Violin plot benchmark visualization

## Summary

Generates violin plots of differential-abundance (DA) z-statistics to visually and quantitatively compare DA-testing methods (ELVAR, DA-seq, Milo) across benchmark datasets, reproducing the paper's Fig. 5-style comparison of statistical significance and fraction of condition-enriched cells captured.

## When to use

Use when benchmarking multiple DA-testing methods on scRNA-Seq data and you need to display the full distribution of DA z-statistics per method/dataset to compare sensitivity.

## When NOT to use

- When only point estimates or single p-values per method are available rather than distributions of z-statistics — a violin plot would convey false spread
- When comparing methods on different datasets or differently preprocessed inputs, since the comparison assumes identical Seurat processing (vst variable features, ScaleData, RunPCA, FindNeighbors k.param=20) per dataset

## Inputs

- DA z-statistics from ELVAR DoDA negative-binomial-regression testing
- DA z-statistics from adapted DA-seq runs on the same datasets
- DA z-statistics from adapted Milo runs on the same datasets
- Per-method/per-dataset fraction of condition-enriched cells captured
- Long-format tidy dataframe with columns for method, dataset, and z-statistic

## Outputs

- Comparative z-statistic violin plot PNG (e.g. fig5_elvar_benchmark_violins.png)
- Table of captured-cell fractions per method and dataset

## How to apply

Collect the per-community or per-cell-type DA z-statistics from each method run — in this benchmark, ELVAR's negative-binomial-regression output from DoDA, plus the adapted DA-seq and Milo z-statistics computed on the same Seurat-processed datasets (Cd4+ T-cell Tabula Muris Senis lung, Covid-19, colon cancer). Assemble a tidy long-format table keyed by method, dataset, and z-statistic, then render violin plots (typically faceted or filled by dataset) so the distribution shift between methods is visible. Pair the violins with a companion table computing, for each method and dataset, the fraction of condition-enriched cells captured, since the paper's claim is that ELVAR z-statistics and captured-cell fractions exceed those of Milo and DA-seq (e.g. for Cd4+ T-cells, alveolar macrophages, Covid-19 and colon enterocyte populations). Save the figure (e.g. fig5_elvar_benchmark_violins.png) and the fractions table as validation artifacts against the paper's reported higher sensitivity.

## Related tools

- **ELVAR** (Source of the method-under-test DA z-statistics via its DoDA negative-binomial-regression step) — https://github.com/aet21/ELVAR
- **Seurat** (Standardizes preprocessing (FindVariableFeatures vst, ScaleData, RunPCA, FindNeighbors k.param=20) so all three methods consume identically prepared inputs)
- **R** (Implementation language for the pipeline and for rendering the comparative violin plots)
- **dplyr** (Tidying and aggregating z-statistics into long format for plotting)

## Examples

```
ggplot(bench_df, aes(x=method, y=zstat, fill=dataset)) + geom_violin(trim=FALSE) + facet_wrap(~dataset) + ggsave("fig5_elvar_benchmark_violins.png")
```

## Evaluation signals

- ELVAR z-statistic distributions in the violins visibly exceed those of Milo and DA-seq across the aging (Cd4+ T-cell, alveolar macrophage), Covid-19, and colon-enterocyte settings
- Captured-cell fractions in the companion table are higher for ELVAR than for Milo and DA-seq in every benchmarked dataset, consistent with the paper's reported higher sensitivity
- All methods were run on identically preprocessed Seurat objects (same PCA dims 1:8, k.param=20 neighbor graph), so violin differences reflect method rather than preprocessing
- Figure renders successfully and is saved as the expected artifact (fig5_elvar_benchmark_violins.png) with a matched fractions table

## Limitations

- ELVAR's improved inference comes at increased computational complexity and runtimes compared to Milo or the Louvain-analog, which affects how quickly the full benchmark (including ~100 runs per alpha value, nine alpha values 0.1-0.9) can be completed before plotting
- No benchmark comparison of ELVAR against miloDE was performed; the violin comparison cannot speak to that newer method
- A stronger-significance violins result depends on reliable phenotype/sample-label attributes, since ELVAR is more sensitive to false positives in the clustering attribute than in cell-type annotation

## Evidence

- [discussion] ELVAR displayed higher sensitivity than Milo and DA-seq to detect biologically important and plausible DA-shifts: "ELVAR displayed higher sensitivity than Milo and DA-seq to detect biologically important and plausible DA-shifts, such as the age-related shift from naïve to mature Cd4+ T-cells in lung tissue"
- [methods] DoDA produces the DA test statistics visualized: "da.o <- DoDA(seu.o,sigcl.o,varDA="naive",varREP="mouse");"
- [methods] Standardized Seurat preprocessing that all compared methods share: "seu.o <- FindNeighbors(seu.o, dims = 1:8, k.param=20,verbose=FALSE);"
- [readme] README confirming ELVAR R-package provenance: "ELVAR is an R-package for differential abundance (DA) testing of cell-types in single-cell RNA-Seq data."
- [abstract] Benchmarking scope against local neighborhood-based methods: "we benchmark ELVAR against an analogous algorithm that uses Louvain for clustering, as well as local neighborhood-based"
