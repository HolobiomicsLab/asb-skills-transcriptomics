---
name: single-cell-module-scoring
description: Use when you have Hotspot-identified heritable gene modules (e.g., from
  phylogenetically autocorrelated genes in a lineage-traced tumor clone) and need
  a per-cell score for each module to overlay on phylogeny or expression projections.
when_to_use_negative:
- Modules are user-defined signatures rather than de novo Hotspot modules — then plain
  VISION signature scoring without the Hotspot de novo module step is appropriate
- No cell similarity structure (phylogeny, spatial metric, or latent space) is available
  — Hotspot module identification underlying the scores cannot be computed
- Input expression is raw counts without library-size normalization for the scoring
  step
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3299
tools:
- name: VISION
  role: Provides the signature-scoring procedure used to compute module scores for
    every cell, and the front-end for the interactive web-based report
  repo: https://github.com/YosefLab/VISION
- name: Hotspot
  role: Identifies the de novo heritable gene modules (autocorrelated genes grouped
    by local correlation clustering) that are scored per cell
  repo: https://github.com/YosefLab/Hotspot
- name: PhyloVision
  role: Pipeline that integrates Hotspot modules, VISION scoring, and phylogenetic
    autocorrelation analysis into an interactive report
  repo: https://github.com/YosefLab/VISION
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100200
    title: Interactive, integrated analysis of single-cell transcriptomic and phylogenetic
      data with PhyloVision
  source_content_hash: sha256:d9d0c964dd9c573ade5264acca75f450055d354534cc1aebb7382b482ecccb44
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/VISION
  tools:
  - Hotspot
  - PhyloVision
  - R
  - VISION
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3299
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# single-cell-module-scoring

## Summary

Computes per-cell gene module scores for de novo Hotspot gene modules (learned from the intersection of phylogeny and expression data) using the signature-scoring procedure in VISION, so heritable expression programs can be visualized across cells in the PhyloVision report.

## When to use

Use when you have Hotspot-identified heritable gene modules (e.g., from phylogenetically autocorrelated genes in a lineage-traced tumor clone) and need a per-cell score for each module to overlay on phylogeny or expression projections.

## When NOT to use

- Modules are user-defined signatures rather than de novo Hotspot modules — then plain VISION signature scoring without the Hotspot de novo module step is appropriate
- No cell similarity structure (phylogeny, spatial metric, or latent space) is available — Hotspot module identification underlying the scores cannot be computed
- Input expression is raw counts without library-size normalization for the scoring step

## Inputs

- Library-size-normalized gene-by-cell expression matrix for the clone (e.g., 1,127 cells)
- Hotspot module definitions: module identities and member genes from the phylogenetically autocorrelated gene set (FDR < 0.05)
- Optional: user-provided gene signatures (e.g., MSigDB) for enrichment interpretation

## Outputs

- Per-cell module scores (one score per cell per Hotspot module)
- Module table (module identities, member genes, and per-cell module scores), e.g. hotspot_gene_modules.csv

## How to apply

After Hotspot has grouped phylogenetically autocorrelated genes into non-overlapping modules (via clustering the pairwise autocorrelation matrix from compute_local_correlations with min_gene_threshold = 70), compute module scores for every cell using the signature-scoring procedure in VISION, treating each Hotspot module's member genes as a signature. In the PhyloVision case study this was applied to a clone of 1,127 metastatic lung adenocarcinoma cells on a Cassiopeia-reconstructed phylogeny, using library-size-normalized expression. The resulting per-cell module scores enable interpretation of heritable expression programs, e.g. via the interactive VISION-based report and enrichment against user-provided signatures such as those from MSigDB. Write the module table (module identities, member genes, and per-cell module scores) to a file such as hotspot_gene_modules.csv for downstream validation and visualization.

## Related tools

- **VISION** (Provides the signature-scoring procedure used to compute module scores for every cell, and the front-end for the interactive web-based report) — https://github.com/YosefLab/VISION
- **Hotspot** (Identifies the de novo heritable gene modules (autocorrelated genes grouped by local correlation clustering) that are scored per cell) — https://github.com/YosefLab/Hotspot
- **PhyloVision** (Pipeline that integrates Hotspot modules, VISION scoring, and phylogenetic autocorrelation analysis into an interactive report) — https://github.com/YosefLab/VISION

## Examples

```
import hotspot; hs.create_knn_graph(weighted_graph=False, n_neighbors=30); hs_results = hs.compute_autocorrelations(); hs_modules = hs.compute_modules(min_gene_threshold=70); module_scores = hs.calculate_module_scores()
```

## Evaluation signals

- Output table (e.g., hotspot_gene_modules.csv) exists and contains module identities plus member genes for every module
- Each module contains at least the minimum gene threshold of member genes (min_gene_threshold = 70) drawn from genes passing the 0.05 FDR autocorrelation threshold
- A per-cell module score is present for every cell in the clone (e.g., all 1,127 cells)
- Module scores vary across cells consistent with phylogenetic structure (assessable with Geary's C phylogenetic autocorrelation in PhyloVision)
- Modules are non-overlapping, as expected from Hotspot's module grouping of the autocorrelated genes

## Limitations

- PhyloVision's implementation in R may provide challenges for utilizing separate software available in Python, like Cassiopeia
- Analyses of this kind (e.g., the metastasis dataset) lack experimental validation to support the generated hypotheses
- The VISION README is truncated in context, so exact scoring function arguments cannot be verified from the provided material

## Evidence

- [results] VISION signature scoring of Hotspot modules: "we computed module scores for every cell using the signature-scoring procedure in VISION"
- [results] Hotspot de novo modules from phylogeny-expression intersection: "gene modules (i.e., not guided by pre-defined signatures) that are learned from the intersection of phylogeny and expression data using our Hotspot algorithm (DeTomaso and Yosef, 2021)"
- [methods] Autocorrelation and module grouping in methods: "clustering the pairwise autocorrelation matrix computed with Hotspot’s “compute_local_correlations” function"
- [readme] Hotspot workflow includes per-cell module scores: "5. Group genes into modules
6. Compute summary per-cell module scores"
- [readme] VISION interprets scRNA-seq via coordinated gene signatures: "VISION aids in the interpretation of single-cell RNA-seq (scRNA-seq) data by selecting for gene signatures which describe coordinated variation between cells."
