---
name: benchmark-score-aggregation-and-ranking
description: Use when benchmark results from multiple cross-species integration strategies
  must be reduced to a single ranked score per task, e.g. after running scIB metrics
  on integrated embeddings from tasks like Pancreas_hs_mm or Hippocampus_hs_mu_ss.
when_to_use_negative:
- 'Cell type mappings across species are non-one-to-one: the paper''s scoring is validated
  only for one-to-one cell type mappings, and the assessment metrics for non-one-to-one
  mappings are not yet available'
- Ground-truth cell type annotations are untrustworthy or homology between cell types
  is uncertain, since the quantitative scoring depends on accurate annotations and
  literature-curated one-to-one homology
- The goal is to compare algorithm implementations on a single dataset rather than
  rank cross-species integration strategies across the BENGAL reference benchmarking
  tasks
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: BENGAL
  role: Pipeline that runs the 28 strategies and produces the integrated scores table
    and rankings
- name: scIB
  role: Computes the 4 species-mixing and 5 biology-conservation metrics (including
    the new cell type distinguishability metric) that are aggregated into the integrated
    score
- name: fastMNN
  role: Integration algorithm whose strategy outputs are scored and ranked
  repo: https://github.com/LTLA/batchelor
- name: Harmony
  role: Integration algorithm whose strategy outputs are scored and ranked
- name: LIGER / LIGER UINMF
  role: iNMF-based integration algorithms (including the unshared-features variant)
    whose outputs are scored and ranked
  repo: https://github.com/welch-lab/liger
- name: SeuratV4
  role: CCA and RPCA variants are among the successfully integrating strategies in
    the ranking
  repo: https://github.com/satijalab/seurat-wrappers
- name: SAMap
  role: Heuristic whole-body atlas integrator included in the scoring and ranking
  repo: https://github.com/atarashansky/SAMap
- name: scOntoMatch
  role: Unifies cell ontology annotations across datasets before metric computation,
    supporting the annotation-accuracy assumption of the scoring
  repo: https://github.com/Papatheodorou-Group/scOntoMatch
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-41855-w
    title: Benchmarking strategies for cross-species integration of single-cell RNA
      sequencing data
  source_content_hash: sha256:ffbc32d68c5d8d10ae15fbb886391d01cf5c26b3371922264a8d72c7d0ff4a44
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LTLA/batchelor
  tools:
  - BENGAL
  - Harmony
  - LIGER
  - LIGER / LIGER UINMF
  - LIGER UINMF
  - SAMap
  - Scanorama
  - SeuratV4
  - fastMNN
  - scANVI
  - scIB
  - scOntoMatch
  - scVI
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.95
n_claims: 20
---

# benchmark-score-aggregation-and-ranking

## Summary

Aggregates scIB species-mixing and biology-conservation metrics for cross-species integration strategies into a single integrated score (weighted 40/60) per benchmark task, then ranks the 28 gene-homology/integration-algorithm combinations within each task. This is how the BENGAL pipeline determines which strategies (e.g. scANVI, scVI, SeuratV4, SAMap) perform most consistently across the 7 reference tasks.

## When to use

Use when benchmark results from multiple cross-species integration strategies must be reduced to a single ranked score per task, e.g. after running scIB metrics on integrated embeddings from tasks like Pancreas_hs_mm or Hippocampus_hs_mu_ss.

## When NOT to use

- Cell type mappings across species are non-one-to-one: the paper's scoring is validated only for one-to-one cell type mappings, and the assessment metrics for non-one-to-one mappings are not yet available
- Ground-truth cell type annotations are untrustworthy or homology between cell types is uncertain, since the quantitative scoring depends on accurate annotations and literature-curated one-to-one homology
- The goal is to compare algorithm implementations on a single dataset rather than rank cross-species integration strategies across the BENGAL reference benchmarking tasks

## Inputs

- Integrated low-dimensional embeddings from each of the 28 strategy combinations (gene homology variant x integration algorithm, plus SAMap for whole-body atlas tasks)
- Cell type annotations used for biology-conservation metric computation (e.g. after scOntoMatch ontology matching)
- scIB metric outputs: 4 species-mixing metrics and 5 biology-conservation metrics per strategy per task

## Outputs

- Integrated score per strategy per task (e.g. range 0.33–0.94 for Pancreas_hs_mm, 0.31–0.89 for Hippocampus_hs_mu_ss)
- Within-task ranking of the 28 integration strategies
- bengal_integrated_scores.csv table of scores and rankings

## How to apply

For each strategy–task combination, compute the 4 batch-correction (species-mixing) metrics and the 5 biology-conservation metrics with scIB, including the dedicated cell-type distinguishability metric developed in the paper. Aggregate the metrics into an integrated score per strategy per task, weighting species mixing 40% and biology conservation 60%, so that strategies balancing the two (scANVI, scVI, SeuratV4) are favored over pure species-mixing or pure biology-conservation behavior. Rank strategies within each task (Pancreas_hs_mm, Hippocampus_hs_mu_ss, Embryo_xt_dr, 4 heart references) and persist both the scores and rankings as bengal_integrated_scores.csv. Interpret the ranking under the paper's two stated assumptions: cell type annotations are accurate, and cell type homology from literature is known one-to-one.

## Related tools

- **BENGAL** (Pipeline that runs the 28 strategies and produces the integrated scores table and rankings)
- **scIB** (Computes the 4 species-mixing and 5 biology-conservation metrics (including the new cell type distinguishability metric) that are aggregated into the integrated score)
- **fastMNN** (Integration algorithm whose strategy outputs are scored and ranked) — https://github.com/LTLA/batchelor
- **Harmony** (Integration algorithm whose strategy outputs are scored and ranked)
- **LIGER / LIGER UINMF** (iNMF-based integration algorithms (including the unshared-features variant) whose outputs are scored and ranked) — https://github.com/welch-lab/liger
- **SeuratV4** (CCA and RPCA variants are among the successfully integrating strategies in the ranking) — https://github.com/satijalab/seurat-wrappers
- **SAMap** (Heuristic whole-body atlas integrator included in the scoring and ranking) — https://github.com/atarashansky/SAMap
- **scOntoMatch** (Unifies cell ontology annotations across datasets before metric computation, supporting the annotation-accuracy assumption of the scoring) — https://github.com/Papatheodorou-Group/scOntoMatch

## Evaluation signals

- All 28 strategy combinations are present with a finite integrated score for each of the 7 benchmark tasks
- Scores fall in a comparable range per task; the paper observed 0.33–0.94 for Pancreas_hs_mm and 0.31–0.89 for Hippocampus_hs_mu_ss
- The species-mixing vs biology-conservation weighting is applied as 40/60 in the aggregation
- Successfully integrating strategies include harmony, fastMNN, scANVI, scVI, SeuratV4 CCA, SeuratV4 RPCA and SAMap; scANVI, scVI and SeuratV4 achieve the best balance in the top ranks
- The output table bengal_integrated_scores.csv contains both scores and within-task rankings

## Limitations

- The quantitative scoring system rests on two assumptions: cell type annotations are accurate and known cell type homology from literature is reliable
- The benchmark uses one-to-one cell type mappings; valid non-one-to-one mappings across species exist and are not assessed
- SAMap is heuristic and does not explain observed cell type expression similarity; expression similarities may arise from convergence or concerted evolution rather than homology, and no algorithm disentangles these sources

## Evidence

- [methods] Aggregate metrics into an integrated score weighted 40/60 between species-mixing and biology conservation per strategy per task.: [evidence span withheld — anchor status: not_found]
- [abstract] using 9 established metrics. We also develop a new biology conservation metric to address the maintenance of cell type distinguishability.: "using 9 established metrics. We also develop a new biology conservation metric to address the maintenance of cell type distinguishability."
- [results] In the Pancreas_hs_mm task, integrated scores ranged between 0.33 and 0.94, and in the Hippocampus_hs_mu_ss task between 0.31 and 0.89.: [evidence span withheld — anchor status: not_found]
- [discussion] Our quantitative scoring system of species mixing and biology conservation is supported by two assumptions: (1) cell type annotations are accurate; (2) known cell type homology from literature is: "Our quantitative scoring system of species mixing and biology conservation is supported by two assumptions: (1) cell type annotations are accurate; (2) known cell type homology from literature is"
- [readme] It relies on integrative non-negative matrix factorization to identify shared and dataset-specific factors.: [evidence span withheld — anchor status: external_doc]
