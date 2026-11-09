---
name: multi-replicates-mouse-cohort-design
description: Use when DA-testing from scRNA-Seq data involves biological replicates
  (e.g. several mice per age group), where inter-mouse variability must be separated
  from the condition-specific cell-state frequency shift.
when_to_use_negative:
- Data come from a single sample per condition with no replicate structure — there
  is no varREP covariate to model and inter-subject variance cannot be estimated
- The goal is plain clustering or cell-type annotation rather than condition-associated
  differential abundance of cell states
- Cell attribute labels (sample/phenotype annotation) carry many false positives,
  since ELVAR was reported to be more sensitive to FPs in the clustering attribute
  than in cell-type annotation
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ELVAR
  role: R-package implementing the full cell-attribute aware DA-testing pipeline (EVA
    clustering + DoDA negative-binomial regression with varREP)
  repo: aet21/ELVAR
- name: Seurat
  role: Creates the Seurat object and builds the cell-cell nearest-neighbor graph
    (FindNeighbors, k.param=20) used as state-manifold input
- name: R
  role: Language in which the entire DA-testing pipeline (clustering and NBR steps)
    is implemented
- name: dplyr
  role: Loaded for data manipulation within the ELVAR workflow
- name: Matrix
  role: Loaded for handling sparse count matrices in the Seurat/ELVAR workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-39017-z
    title: Cell-attribute aware community detection improves differential abundance
      testing from single-cell RNA-Seq data
  source_content_hash: sha256:2143e1d26d1c5ba7b6df5578b9acb272d027de6ad096c6893be33f17bc2e59cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aet21/ELVAR
  tools:
  - ELVAR
  - EVA
  - Matrix
  - R
  - Seurat
  - dplyr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# multi-replicates-mouse-cohort-design

## Summary

Design and analyze a multi-mouse scRNA-Seq cohort in which biological replicate structure (multiple mice per age group) is carried through the ELVAR differential-abundance pipeline, with mouse replicate modeled explicitly as a variance covariate (varREP) in the negative-binomial regression step.

## When to use

Use when DA-testing from scRNA-Seq data involves biological replicates (e.g. several mice per age group), where inter-mouse variability must be separated from the condition-specific cell-state frequency shift.

## When NOT to use

- Data come from a single sample per condition with no replicate structure — there is no varREP covariate to model and inter-subject variance cannot be estimated
- The goal is plain clustering or cell-type annotation rather than condition-associated differential abundance of cell states
- Cell attribute labels (sample/phenotype annotation) carry many false positives, since ELVAR was reported to be more sensitive to FPs in the clustering attribute than in cell-type annotation

## Inputs

- scRNA-Seq count matrix with per-cell condition attribute (e.g. age) and mouse-replicate labels
- Seurat object with phenotype/replicate metadata (seu.o@meta.data$age)
- Cell-cell nearest-neighbor state-manifold graph with vertex attributes
- Simulated cohort parameters (cell count, per-condition perturbation frequencies such as 0.25 vs 0.5)

## Outputs

- Negative-binomial-regression Wald/LRT DA statistics per cell-state per mouse replicate (DoDA output object)
- Bonferroni-corrected binomial attribute-enrichment calls per cluster (ProcessEVA output, sigcl.o)
- Per-method DA significance distributions across simulation replicates
- Wilcoxon rank-sum comparison P-values (e.g. saved to fig2e_wilcoxon_pvalues.csv)

## How to apply

Construct or simulate a multi-mouse dataset (as in the paper: 993 cells from young vs old mice, perturbed cell-state frequency rising from 0.25 in young to 0.5 in old mice) so that replicate mice are sampled within each condition. Build the cell-cell nearest-neighbor state-manifold graph with Seurat FindNeighbors (dims=1:8, k.param=20), attach the condition attribute (e.g. 'age') to graph vertices, and run EVA clustering via Eva_partitions (resolution=1, threshold=1e-7, alpha selected over 0.1–0.9 with SelOptAlpha) alongside Louvain baselines (LVdet, LVnonseq). Select attribute-enriched communities with ProcessEVA (binomial enrichment tests, Bonferroni-corrected), then run negative-binomial regression DA-testing with DoDA passing varDA = the cell-state/condition label and varREP = mouse replicate, so replicate-to-replicate variance does not inflate DA significance. Repeat across simulation replicates (e.g. 100 runs) and compare significance distributions across methods with one-sided Wilcoxon rank-sum tests.

## Related tools

- **ELVAR** (R-package implementing the full cell-attribute aware DA-testing pipeline (EVA clustering + DoDA negative-binomial regression with varREP)) — aet21/ELVAR
- **Seurat** (Creates the Seurat object and builds the cell-cell nearest-neighbor graph (FindNeighbors, k.param=20) used as state-manifold input)
- **R** (Language in which the entire DA-testing pipeline (clustering and NBR steps) is implemented)
- **dplyr** (Loaded for data manipulation within the ELVAR workflow)
- **Matrix** (Loaded for handling sparse count matrices in the Seurat/ELVAR workflow)

## Examples

```
da.o <- DoDA(seu.o,sigcl.o,varDA="age",varREP="mouse");
```

## Evaluation signals

- Validation succeeds if ELVAR attains significantly smaller one-sided Wilcoxon rank-sum P-values than both LVdet and LVnonseq DA-test P-value distributions (matching the Fig. 2e result)
- DoDA call must include both varDA (cell-state/condition label) and varREP='mouse' so replicate variance is modeled; omitting varREP is a design error
- EVA runs are repeated across simulation replicates (100 runs) with alpha optimized over seq(0.1, 0.9, 0.1) via SelOptAlpha
- ProcessEVA enrichment uses binomial tests with Bonferroni correction before any cluster is passed to DoDA
- Per-method DA statistics and the two Wilcoxon rank-sum P-values are written to fig2e_wilcoxon_pvalues.csv

## Limitations

- Cell-attribute aware clustering increases computational complexity and runtimes compared to Milo or the Louvain-analog; alpha optimization may require ~900 runs (100 runs x nine alpha values 0.1–0.9)
- ELVAR is sensitive to false positives in the clustering attribute (sample/phenotype label), which can degrade the quality of inferred communities
- No benchmark against miloDE was performed; the authors flag this as an open question
- Improvement over the Louvain-analog is more pronounced in studies with weaker clustering structure and less noticeable when clustering is strong

## Evidence

- [intro] Replicate variance must be modeled: "biological inter-subject variability, as well as technical batch effects, can potentially confound naïve DA-analyses"
- [methods] DoDA call models mouse replicate: "da.o <- DoDA(seu.o,sigcl.o,varDA="naive",varREP="mouse");"
- [discussion] Replicate-aware clustering can obviate batch correction: "ELVAR can circumvent the need for batch correction, as performing batch-correction prior to ELVAR rarely led to an improvement in sensitivity"
- [discussion] EVA alpha optimization is computationally costly: "to identify an optimal a parameter value may require a total of 100 runs for each of nine a parameter values, so a total of 900 runs"
