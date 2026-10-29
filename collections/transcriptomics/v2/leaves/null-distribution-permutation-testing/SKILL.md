---
name: null-distribution-permutation-testing
description: Reference for building an empirical null distribution from shuffled/permuted
  single-cell data to classify embeddings or statistics (e.g., scDEED dubious vs trustworthy
  cells at dubious_cutoff=0.05 and trustworthy_cutoff=0.95).
when_to_use_negative:
- The analysis goal is hyperparameter search itself rather than significance classification
  — use the full scDEED optimize() grid search instead of a single permuted-vs-original
  comparison
- The input is raw count data that has not been normalized and scaled, since Permuted()
  shuffles rows of the scale.data slot and requires a completed Seurat preprocessing
  pipeline
- A parametric or theoretical null is appropriate and empirical marginal resampling
  would be too costly (scDEED's permuted runs double the embedding computations, and
  the joint UMAP grid search is already computationally expensive)
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3307
- http://edamontology.org/topic_3308
tools:
- name: scDEED
  role: Provides Permuted(), Distances.pre_embedding(), Cell.Similarity(), and Cell.Classify()
    to build the permuted null and classify cells against its percentiles
  repo: github.com/JSB-UCLA/scDEED
- name: EMBEDR
  role: Alternative empirical hypothesis test that generates the null distribution
    via marginal resampling to compute per-sample embedding p-values
  repo: ejohnson643/EMBEDR
- name: Seurat
  role: Supplies the Seurat object, scale.data slot, and PCA pre-embedding space that
    Permuted() shuffles
  repo: satijalab/seurat-wrappers
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_004
  - task_001
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-45891-y
    title: Statistical method scDEED for detecting dubious 2D single-cell embeddings
      and optimizing t-SNE and UMAP hyperparameters
  source_content_hash: sha256:025f070a0e5208899be72ed7512b81726a8f365b53c5d4b07beb2e8d64eaa993
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EMBEDR
  - R
  - Seurat
  - devtools
  - doParallel
  - scDEED
  - scDEED R package
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3307
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# null-distribution-permutation-testing

## Summary

Constructs a signalless null distribution by permuting (marginal resampling) the input data, then classifies per-cell embedding reliability scores against the null's 5th/95th percentiles. In the scDEED context, the Permuted() function shuffles each row of the Seurat scale.data slot so that the reliability scores of real cell embeddings can be compared to those expected by chance.

## When to use

Reference for building an empirical null distribution from shuffled/permuted single-cell data to classify embeddings or statistics (e.g., scDEED dubious vs trustworthy cells at dubious_cutoff=0.05 and trustworthy_cutoff=0.95).

## When NOT to use

- The analysis goal is hyperparameter search itself rather than significance classification — use the full scDEED optimize() grid search instead of a single permuted-vs-original comparison
- The input is raw count data that has not been normalized and scaled, since Permuted() shuffles rows of the scale.data slot and requires a completed Seurat preprocessing pipeline
- A parametric or theoretical null is appropriate and empirical marginal resampling would be too costly (scDEED's permuted runs double the embedding computations, and the joint UMAP grid search is already computationally expensive)

## Inputs

- Seurat object with a populated scale.data slot (e.g., after NormalizeData, ScaleData, FindVariableFeatures, RunPCA)
- Number of pre-embedding dimensions K (e.g., K=8 in the README tutorial, K=31 for Hydra, K=15 for CAR-T)
- Cutoff percentiles (dubious_cutoff=0.05, trustworthy_cutoff=0.95) and similarity_percent=0.5

## Outputs

- Permuted Seurat object (data.permuted) used as the signalless null
- Per-cell similarity scores for original and permuted data (rho_original, rho_permuted)
- Per-cell classification into dubious / trustworthy / intermediate sets
- Counts of dubious cell embeddings per hyperparameter setting (result$num_dubious, result$full_results)

## How to apply

Generate the null data by calling Permuted(data, default_assay='RNA', slot='scale.data', K), which shuffles each row of the scale.data slot within the Seurat object. Compute the statistic of interest (e.g., pre-embedding Pearson correlations of distances to nearest neighbors via Distances.pre_embedding on both original and permuted data, then Cell.Similarity with similarity_percent=0.5). Classify real scores against percentiles of the null distribution using Cell.Classify: scores below the dubious_cutoff percentile (default 0.05) are dubious, scores above the trustworthy_cutoff percentile (default 0.95) are trustworthy, and the rest are intermediate. The percentile cutoffs are user-tunable based on knowledge and preference, per the article's discussion. The same empirical-hypothesis-test principle underlies EMBEDR, which builds its null by marginal resampling.

## Related tools

- **scDEED** (Provides Permuted(), Distances.pre_embedding(), Cell.Similarity(), and Cell.Classify() to build the permuted null and classify cells against its percentiles) — github.com/JSB-UCLA/scDEED
- **EMBEDR** (Alternative empirical hypothesis test that generates the null distribution via marginal resampling to compute per-sample embedding p-values) — ejohnson643/EMBEDR
- **Seurat** (Supplies the Seurat object, scale.data slot, and PCA pre-embedding space that Permuted() shuffles) — satijalab/seurat-wrappers

## Examples

```
data.permuted = Permuted(data, default_assay = 'RNA', slot = 'scale.data', K); results.PCA = Distances.pre_embedding(data, data.permuted, K=K, pre_embedding = 'pca')
```

## Evaluation signals

- The permuted object has the same dimensions as the original but each row of scale.data is shuffled across cells (row-wise permutation of the scale.data slot)
- With defaults, most cell embeddings should be classified trustworthy, with a smaller number dubious or intermediate (null 5th/95th percentiles bound the intermediate band)
- Changing dubious_cutoff upward (toward higher percentile) must increase the number of cells called dubious, per the README's cutoff semantics
- Dubious/trustworthy percentages should reproduce reported values, e.g., Hydra 4.77%/92.84% at perplexity 40 vs 0.51%/99.1% at optimized perplexity 230

## Limitations

- Differences between dubious and trustworthy embeddings may reflect randomness rather than biological signals under optimized hyperparameters
- Seurat version differences (5.2.1 vs 5.1.0) were reported to affect installation of the scDEED package
- Cutoff percentiles are user-adjustable; the default 0.05/0.95 thresholds may need tuning based on knowledge and preference

## Evidence

- [methods] Permuted function shuffles each row of the scale.data slot: [evidence span withheld — anchor status: not_found]
- [methods] Cell.Classify cutoffs: "ClassifiedCells<-Cell.Classify(similarity_score$rho_original,similarity_score$rho_permuted, dubious_cutoff = 0.95, trustworthy_cutoff = 0.95)"
- [readme] null distribution cutoff semantics: "the cell's similarity score must be lower than the 5th percentile (or `dubious_cutoff` percentile) of cell embedding scores from the null distribution"
- [abstract] reliability score definition: "calculating a reliability score for every cell embedding based on the similarity between the cell's 2D-embedding neighbors and pre-embedding neighbors"
- [readme] EMBEDR empirical hypothesis test: [evidence span withheld — anchor status: external_doc]
