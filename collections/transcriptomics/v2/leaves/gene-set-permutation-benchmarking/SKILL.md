---
name: gene-set-permutation-benchmarking
description: Use when you need to quantitatively validate a gene-set-scoring cell-type
  annotation pipeline — i.e., measure how annotation accuracy (sensitivity/specificity)
  degrades as marker sets are corrupted with random genes at increasing replacement
  rates.
when_to_use_negative:
- Your goal is annotating a new dataset with trusted, unmodified marker sets — this
  skill is a benchmarking exercise with deliberately degraded gene sets, not a production
  annotation workflow.
- You lack author-curated (ground-truth) cell-type labels, since benchmarkScores()
  sensitivity/specificity cannot be computed against reference annotations.
- The task is comparing alternative automated annotation tools (e.g., ScType) rather
  than stress-testing the Miko scoring pipeline itself.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scMiko
  role: Provides nullScore(), mikoScore(), sigScore(), coherentFraction() and benchmarkScores()
    used to score permuted gene sets and compute sensitivity/specificity
  repo: https://github.com/NMikolajewicz/scMiko
- name: scPipeline
  role: Modular Rmarkdown workflow scripts in which the permutation benchmark is executed
    and reported
  repo: https://github.com/NMikolajewicz/scPipeline
- name: Seurat
  role: Framework for building the Seurat object from the gastrulation count matrix,
    per-sample SCTransform normalization, and merging
- name: sctransform
  role: SCTransform normalization (glmGamPoi, v2 flavor) applied per sample prior
    to merging
  repo: https://github.com/satijalab/sctransform
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42003-022-04093-2
    title: Multi-level cellular and functional annotation of single-cell transcriptomes
      using scPipeline
  source_content_hash: sha256:b75a30d0d984c4093f5a1752d8a2c67204a1d273a0e642716282751a628056ff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NMikolajewicz/scMiko
  tools:
  - Seurat
  - scMiko
  - scPipeline
  - sctransform
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# gene-set-permutation-benchmarking

## Summary

Benchmark the sensitivity and specificity of Miko score-based cell-type annotation by degrading cell-type-specific marker gene sets with 0%, 25% and 50% random-gene permutation and comparing called annotations against author-curated labels, as done for the mouse gastrulation dataset (Pijuan-Sala et al.) in scPipeline.

## When to use

Use when you need to quantitatively validate a gene-set-scoring cell-type annotation pipeline — i.e., measure how annotation accuracy (sensitivity/specificity) degrades as marker sets are corrupted with random genes at increasing replacement rates.

## When NOT to use

- Your goal is annotating a new dataset with trusted, unmodified marker sets — this skill is a benchmarking exercise with deliberately degraded gene sets, not a production annotation workflow.
- You lack author-curated (ground-truth) cell-type labels, since benchmarkScores() sensitivity/specificity cannot be computed against reference annotations.
- The task is comparing alternative automated annotation tools (e.g., ScType) rather than stress-testing the Miko scoring pipeline itself.

## Inputs

- ps_count_matrix.rds — murine gastrulation raw count matrix (Pijuan-Sala et al.)
- ps_meta_data.rds — sample/cell metadata with author-curated cell-type labels
- Cell-type-specific marker gene sets derived by DE from author-annotated cell types
- Permutation levels (0%, 25%, 50% gene replacement)

## Outputs

- benchmark_permutation_results.csv — sensitivity/specificity per permutation level
- Significant cluster enrichment calls per permuted gene set (FDR < 0.05, miko_score > 0)
- Coherence-filter accept/reject decisions per candidate annotation

## How to apply

Load the murine gastrulation count matrix (ps_count_matrix.rds) and metadata (ps_meta_data.rds) into a Seurat object, split by sample, and normalize each sample with SCTransform (glmGamPoi, v2 flavor) before merging. Derive cell-type-specific marker gene sets from author-annotated cell types by differential expression using benchmarkScores' deg.logFC.threshold and deg.fdr.threshold settings, then permute gene-set membership at 0%, 25% and 50% replacement levels to produce degraded marker sets of matched size. Run nullScore() to fit the gene-set-size-dependent null model (nbin = 24, min.gs.size = 2, max.gs.size = 200, step.size = 10), score the permuted sets with mikoScore(), and call significant cluster enrichments with sigScore() (FDR < 0.05, miko_score > 0), applying the coherence filter (coherence_fraction >= 0.8) to accept or reject annotations. Finally compute sensitivity and specificity against author-curated labels with benchmarkScores() at each permutation level and write benchmark_permutation_results.csv; the expected trend is 100%/94% at 0% permutation, 93%/96% at 25%, and 54%/98% at 50% (Fig. 4d-e), i.e., sensitivity declines toward ~54% while specificity remains high (~94-98%).

## Related tools

- **scMiko** (Provides nullScore(), mikoScore(), sigScore(), coherentFraction() and benchmarkScores() used to score permuted gene sets and compute sensitivity/specificity) — https://github.com/NMikolajewicz/scMiko
- **scPipeline** (Modular Rmarkdown workflow scripts in which the permutation benchmark is executed and reported) — https://github.com/NMikolajewicz/scPipeline
- **Seurat** (Framework for building the Seurat object from the gastrulation count matrix, per-sample SCTransform normalization, and merging)
- **sctransform** (SCTransform normalization (glmGamPoi, v2 flavor) applied per sample prior to merging) — https://github.com/satijalab/sctransform

## Examples

```
devtools::install_github(repo = "NMikolajewicz/scMiko"); ps.data <- readRDS("../data/demo/ps_count_matrix.rds")
```

## Evaluation signals

- Sensitivity ~100% (100%) and specificity ~94% at 0% permutation; 93%/96% at 25%; 54%/98% at 50% — reproducing the Fig. 4d-e trend of declining sensitivity with stable high specificity.
- Permuted gene sets have the same size as the original marker sets (matched-size replacement).
- All enrichment calls pass FDR < 0.05 with miko_score > 0, and accepted annotations satisfy coherence_fraction >= 0.8.
- nullScore() null model fitted with nbin = 24, min.gs.size = 2, max.gs.size = 200, step.size = 10, covering gene-set sizes 2-200.
- benchmark_permutation_results.csv exists and contains one sensitivity/specificity row per permutation level (0%, 25%, 50%).

## Limitations

- Marker-based annotations are only as good as the cell-type prescribed to the original dataset; no validation of source catalog curations is provided.
- At 50% permutation sensitivity drops to ~54%, so heavily corrupted marker sets are frequently rejected or misassigned even though specificity stays high — the benchmark measures degradation, not robustness at high corruption.
- Annotation performance is tied to the gastrulation dataset's author-curated labels; sensitivity/specificity values may not transfer to other tissues or datasets.

## Evidence

- [methods] Miko scoring pipeline steps: "The miko scoring pipeline is a 3-step workflow that is implemented in nullScore(), mikoScore(), and sigScore() functions."
- [methods] Null model size-bias correction: "In step 1 we run nullScore(), which fits a null model that corrects for gene set-size biases."
- [methods] Coherence filter: "The first is a coherence filter in which a positive correlation between component gene expression and the Miko score is enforced for a minimum fraction of component genes."
- [discussion] Hypothesis-testing accept/reject framework: "we demonstrate how to annotate clusters using our Miko Scoring pipeline, which facilitates unbiased scoring of a diverse set of variable-sized cell type-specific gene sets"
- [methods] scMiko install command: "devtools::install_github(repo = "NMikolajewicz/scMiko")"
- [methods] Datasets for benchmark: "ps.data <- readRDS("../data/demo/ps_count_matrix.rds")"
- [discussion] Annotation quality caveat: "we emphasize that marker-based annotations are only as good as the cell-type prescribed to the original dataset."
- [readme] scPipeline reproducible reports: [evidence span withheld — anchor status: external_doc]
