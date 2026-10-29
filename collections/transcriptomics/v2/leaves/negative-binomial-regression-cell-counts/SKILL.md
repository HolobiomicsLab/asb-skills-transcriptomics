---
name: negative-binomial-regression-cell-counts
description: Use when testing whether counts of attribute-enriched cell communities
  (from ProcessEVA) shift with a biological condition across replicates, e.g. naive-vs-mature
  Cd4+ T-cells with age or stem/T-reg fractions with cancer stage.
when_to_use_negative:
- Input is aggregate/bulk cell-type counts with no cell-cell neighborhood graph or
  replicate structure, so no attribute-enriched communities can be defined
- No replicate/sample variable exists to model inter-subject variability, violating
  the NBR design (varREP is required)
- The clustering attribute (sample/phenotype label) is suspected of containing false
  positives, which ELVAR is known to be sensitive to
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: ELVAR
  role: Provides the DoDA function that fits negative binomial regressions of enriched-community
    cell counts on the DA variable with replicate covariate
  repo: https://github.com/aet21/ELVAR
- name: MASS
  role: Supplies the negative binomial regression machinery underlying DoDA's Wald
    and LRT tests
- name: Seurat
  role: Holds the scRNA-Seq count matrix and metadata (age/phenotype, replicate) passed
    to DoDA
- name: R
  role: Language in which the DA-testing pipeline including the NBR step is implemented
  repo: https://github.com/aet21/ELVAR
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-39017-z
    title: Cell-attribute aware community detection improves differential abundance
      testing from single-cell RNA-Seq data
  source_content_hash: sha256:1ff69812ada5294a15478dcd2eeb7edb8ffda7a7e3fc0338daec729ed5bb3a18
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
  - ggplot2
  - igraph
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# negative-binomial-regression-cell-counts

## Summary

The DoDA step of the ELVAR pipeline: negative binomial regressions (NBR) of attribute-enriched community cell counts on a condition variable (e.g. age, disease stage, smell-loss phenotype), with mouse/sample replicate as covariate, yielding Wald and LRT statistics for differential abundance of cell states.

## When to use

Use when testing whether counts of attribute-enriched cell communities (from ProcessEVA) shift with a biological condition across replicates, e.g. naive-vs-mature Cd4+ T-cells with age or stem/T-reg fractions with cancer stage.

## When NOT to use

- Input is aggregate/bulk cell-type counts with no cell-cell neighborhood graph or replicate structure, so no attribute-enriched communities can be defined
- No replicate/sample variable exists to model inter-subject variability, violating the NBR design (varREP is required)
- The clustering attribute (sample/phenotype label) is suspected of containing false positives, which ELVAR is known to be sensitive to

## Inputs

- Seurat object with cell metadata containing the DA variable (age, disease stage, smell-loss phenotype) and a replicate variable (mouse ID, patient/sample ID)
- Significant-community object from ProcessEVA (attribute-enriched, Bonferroni-significant merged cell sets per condition group)
- Undirected igraph cell-cell nearest-neighbor graph with condition attached as vertex attribute

## Outputs

- Wald and LRT statistics with P-values per cell state (e.g. naive CS1 and mature/activated CS0 fractions), exportable as CSV (e.g. elvar_da_statistics.csv, olfactory_epithelium_da_results.csv)
- Direction-of-change calls for each cell state versus the condition variable

## How to apply

First obtain attribute-enriched communities by running Eva_partitions on the cell-cell nearest-neighbor graph (FindNeighbors, dims 1:8, k.param=20) with the condition as vertex attribute, then ProcessEVA with binomial tests and Bonferroni significance to merge significant cells per condition group. Feed the Seurat object and the significant-community object into DoDA with varDA set to the condition/phenotype variable (e.g. varDA='naive' for age) and varREP set to the replicate variable (e.g. varREP='mouse' or patient/sample ID), so MASS negative-binomial regressions model cell-state counts on the condition while accounting for inter-subject variability. Record both Wald and LRT statistics and P-values, then benchmark against the Louvain non-sequential (alpha=0) analog to confirm the cell-attribute-aware clustering yields stronger significance.

## Related tools

- **ELVAR** (Provides the DoDA function that fits negative binomial regressions of enriched-community cell counts on the DA variable with replicate covariate) — https://github.com/aet21/ELVAR
- **MASS** (Supplies the negative binomial regression machinery underlying DoDA's Wald and LRT tests)
- **Seurat** (Holds the scRNA-Seq count matrix and metadata (age/phenotype, replicate) passed to DoDA)
- **R** (Language in which the DA-testing pipeline including the NBR step is implemented) — https://github.com/aet21/ELVAR

## Examples

```
da.o <- DoDA(seu.o,sigcl.o,varDA="naive",varREP="mouse");
```

## Evaluation signals

- Wald and LRT P-values for the expected cell states are highly significant, e.g. significant decrease of the naive (CS1) fraction and increase of the mature/activated (CS0) fraction with age
- Each enriched condition group is supported by at least two mouse/sample replicates before fitting the regression
- Significance from the ELVAR (EVA-based) communities is stronger than from the Louvain non-sequential (LVnonseq, alpha=0) benchmark
- Statistics table is complete and saved (e.g. elvar_da_statistics.csv) with direction and significance matching the published figures (Fig. 3g, Supplementary Fig. S6/S7)

## Limitations

- ELVAR is more sensitive to false positives in the clustering attribute (sample/phenotype label) than to false positives in cell-type annotation, which can affect the quality of inferred communities feeding the NBR
- Cell-attribute aware DA inference comes at increased computational complexity and runtimes compared to Milo or the Louvain-analog, though deemed not practically prohibitive
- No benchmark of ELVAR against miloDE was performed; the authors flag this as an open question

## Evidence

- [methods] DoDA call in tutorial: "da.o <- DoDA(seu.o,sigcl.o,varDA="naive",varREP="mouse");"
- [intro] pipeline including NBR step: "We implement the whole DA-testing pipeline (including the cell-attribute-aware clustering and NBR steps) in the programming language R"
- [intro] confounding rationale: "biological inter-subject variability, as well as technical batch effects, can potentially confound naïve DA-analyses"
- [discussion] ELVAR sensitivity finding: "ELVAR displayed higher sensitivity than Milo and DA-seq to detect biologically important and plausible DA-shifts"
- [readme] ELVAR package purpose: "ELVAR is an R-package for differential abundance (DA) testing of cell-types in single-cell RNA-Seq data."
