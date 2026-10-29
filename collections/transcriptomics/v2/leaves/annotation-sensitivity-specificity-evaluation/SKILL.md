---
name: annotation-sensitivity-specificity-evaluation
description: Use when you must quantify how robust an scRNA-seq cell-type annotation
  method is to marker-gene-set noise, by systematically degrading marker sets and
  comparing resulting annotations against ground-truth cell-type labels.
when_to_use_negative:
- You only need a point annotation of clusters with a known, intact marker database
  (use the standard Miko scoring or sc-type workflow instead of benchmarking)
- Your goal is evaluating unsupervised clustering resolution rather than marker-set-driven
  annotation accuracy
- No author-curated ground-truth cell-type labels are available to compute sensitivity/specificity
  against
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scMiko
  role: Provides nullScore(), mikoScore(), sigScore(), coherentFraction(), and benchmarkScores()
    used to score permuted gene sets and compute sensitivity/specificity
  repo: https://github.com/NMikolajewicz/scMiko
- name: scPipeline
  role: Modular Rmarkdown workflow scripts that orchestrate the preprocessing, scoring,
    and benchmarking steps
  repo: https://github.com/NMikolajewicz/scPipeline
- name: Seurat
  role: Framework for loading the gastrulation data as a Seurat object and per-sample
    normalization/merging
- name: sctransform
  role: SCTransform normalization (glmGamPoi, v2 flavor) of each sample before merging
  repo: https://github.com/satijalab/sctransform
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42003-022-04093-2
    title: Multi-level cellular and functional annotation of single-cell transcriptomes
      using scPipeline
  source_content_hash: sha256:fbda65216eef2e0e97d43e0f60ec4aabc78df076b0afa5c435cdb88bcd3ce63a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NMikolajewicz/scMiko
  tools:
  - Seurat
  - scMiko
  - scPipeline
  - sctransform
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# annotation-sensitivity-specificity-evaluation

## Summary

Benchmark the sensitivity and specificity of Miko score-based cell-type annotation by permuting cell-type-specific gene sets with random genes (0%, 25%, 50% replacement) and measuring how annotation accuracy degrades against author-curated labels.

## When to use

Use when you must quantify how robust an scRNA-seq cell-type annotation method is to marker-gene-set noise, by systematically degrading marker sets and comparing resulting annotations against ground-truth cell-type labels.

## When NOT to use

- You only need a point annotation of clusters with a known, intact marker database (use the standard Miko scoring or sc-type workflow instead of benchmarking)
- Your goal is evaluating unsupervised clustering resolution rather than marker-set-driven annotation accuracy
- No author-curated ground-truth cell-type labels are available to compute sensitivity/specificity against

## Inputs

- ps_count_matrix.rds (murine gastrulation scRNA-seq count matrix, Pijuan-Sala et al.)
- ps_meta_data.rds (author-curated cell-type labels for the gastrulation dataset)
- Cell-type-specific marker gene sets derived by differential expression from author-annotated cell types
- Random gene pools for permutation at 0%, 25%, and 50% replacement rates

## Outputs

- benchmark_permutation_results.csv (sensitivity and specificity per permutation level)
- Significant cluster enrichment calls per permutation level (sigScore output, FDR < 0.05, miko_score > 0)
- Permutation-level accuracy values expected per Fig. 4d-e: 100%/94% at 0%, 93%/96% at 25%, 54%/98% at 50%

## How to apply

Load the murine gastrulation count matrix (ps_count_matrix.rds) and metadata (ps_meta_data.rds) into a Seurat object, split by sample, and normalize each sample with SCTransform (glmGamPoi, v2 flavor) before merging. Derive cell-type-specific marker gene sets from author-annotated cell types via differential expression (benchmarkScores deg.logFC.threshold and deg.fdr.threshold settings), then permute set membership at 0%, 25% and 50% replacement with random genes to generate degraded sets of matched size. Fit the gene set-size-dependent null model with nullScore() (nbin = 24, min.gs.size = 2, max.gs.size = 200, step.size = 10), score the permuted sets with mikoScore(), and call significant cluster enrichments with sigScore() (FDR < 0.05, miko_score > 0). Apply the coherence filter (coherence_fraction >= 0.8) to accept or reject candidate annotations, then compute sensitivity and specificity with benchmarkScores() against the author-curated labels at each permutation level, writing results to benchmark_permutation_results.csv.

## Related tools

- **scMiko** (Provides nullScore(), mikoScore(), sigScore(), coherentFraction(), and benchmarkScores() used to score permuted gene sets and compute sensitivity/specificity) — https://github.com/NMikolajewicz/scMiko
- **scPipeline** (Modular Rmarkdown workflow scripts that orchestrate the preprocessing, scoring, and benchmarking steps) — https://github.com/NMikolajewicz/scPipeline
- **Seurat** (Framework for loading the gastrulation data as a Seurat object and per-sample normalization/merging)
- **sctransform** (SCTransform normalization (glmGamPoi, v2 flavor) of each sample before merging) — https://github.com/satijalab/sctransform

## Examples

```
nullScore(object = so.query, nbin = 24, min.gs.size = 2, max.gs.size = 200, step.size = 10); so.query <- mikoScore(object = so.query); bm <- benchmarkScores(object = sigScore(object = so.query, fdr.threshold = 0.05, miko.score.threshold = 0), coherence_fraction = 0.8)
```

## Evaluation signals

- Non-permuted gene sets (0% permutation) yield ~100% sensitivity and ~94% specificity for cluster-specific gene sets
- 25% permutation yields ~93% sensitivity and ~96% specificity; 50% permutation yields ~54% sensitivity with specificity remaining high (~98%)
- Sensitivity declines monotonically with permutation rate while specificity stays in the 94-98% range, matching the Fig. 4d-e benchmark trend
- Output table benchmark_permutation_results.csv contains sensitivity/specificity rows for all three permutation levels
- Significant enrichment calls satisfy FDR < 0.05 and miko_score > 0, with accepted annotations passing coherence_fraction >= 0.8

## Limitations

- Marker-based annotations are only as good as the cell-type prescribed to the original dataset; no validation of source catalog curations is provided
- Occasional co-similarities between marker sets from heterogeneous cell types reflect either marker curation inaccuracies or cross-cell-type process similarities and are left unresolved
- At high permutation rates (50%) annotation sensitivity drops sharply (~54%), so results at degraded set quality should be interpreted as a stress test, not operational performance

## Evidence

- [methods] Miko scoring pipeline structure: "The miko scoring pipeline is a 3-step workflow that is implemented in nullScore(), mikoScore(), and sigScore() functions."
- [discussion] Hypothesis-testing accept/reject of annotations: "we demonstrate how to annotate clusters using our Miko Scoring pipeline, which facilitates unbiased scoring of a diverse set of variable-sized cell type-specific gene sets and accepts or rejects"
- [methods] Coherence filter definition: "The first is a coherence filter in which a positive correlation between component gene expression and the Miko score is enforced for a minimum fraction of component genes."
- [readme] scPipeline/scMiko dependency and scope: [evidence span withheld — anchor status: not_found]
- [results] Benchmark sensitivity/specificity finding: [evidence span withheld — anchor status: not_found]
