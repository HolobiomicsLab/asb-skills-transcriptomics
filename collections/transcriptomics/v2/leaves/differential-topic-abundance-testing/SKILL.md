---
name: differential-topic-abundance-testing
description: Use when AnnData outputs from a trained scETM model (normalized cell
  topic mixtures in obsm['theta'], topic embeddings in uns['alpha']) are available
  with obs metadata labeling disease condition and cell type, and you need to identify
  which learned topics are differentially enriched between groups.
when_to_use_negative:
- The AnnData lacks trained scETM embeddings (no obsm['theta'] / uns['alpha']) or
  the topic model has not been fit to the cells-by-genes matrix
- The obs metadata does not contain the categorical group labels (disease condition
  or cell type) needed to define the tested contrasts
- The goal is cell-type clustering or batch correction rather than testing topic-level
  differences between labeled groups
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scETM
  role: Trains the embedded topic model and exports the cell topic mixtures (obsm['theta']),
    gene embeddings (varm['rho']), and topic embeddings (uns['alpha']) that are contrasted
    in the permutation test
  repo: https://github.com/hui2000ji/scETM
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-25534-2
    title: Learning interpretable cellular and gene signature embeddings from single-cell
      transcriptomic data
  source_content_hash: sha256:1397f351822267db9d2c9a990f03f7f56fb6650bc0175aebb9770a515589ae43
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hui2000ji/scETM
  tools:
  - scETM
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# differential-topic-abundance-testing

## Summary

Statistical testing of scETM-learned topic loadings for differential abundance between disease/control conditions and across cell types, using large-scale label permutation and Bonferroni correction, followed by inspection of topic gene loadings (varm['rho']) to assign pathway character.

## When to use

Use when AnnData outputs from a trained scETM model (normalized cell topic mixtures in obsm['theta'], topic embeddings in uns['alpha']) are available with obs metadata labeling disease condition and cell type, and you need to identify which learned topics are differentially enriched between groups (e.g., AD vs control).

## When NOT to use

- The AnnData lacks trained scETM embeddings (no obsm['theta'] / uns['alpha']) or the topic model has not been fit to the cells-by-genes matrix
- The obs metadata does not contain the categorical group labels (disease condition or cell type) needed to define the tested contrasts
- The goal is cell-type clustering or batch correction rather than testing topic-level differences between labeled groups

## Inputs

- AnnData object from a trained scETM model with normalized cell topic mixtures at obsm['theta']
- Topic embeddings stored at uns['alpha']
- Topic gene loadings stored at varm['rho']
- obs metadata columns annotating disease condition (AD vs control; suicidal vs healthy) and cell type

## Outputs

- Single table listing topic IDs, contrasts, permutation p-values, and Bonferroni-adjusted significance
- Set of significant differential topics per contrast (e.g., AD: topics 12 and 58 at p-value = 1e-5)
- Pathway-character confirmation for significant topics from varm['rho'] gene loadings (e.g., mitochondrial gene enrichment for topic 58)

## How to apply

Load the trained scETM outputs from the AnnData objects, reading the normalized cell topic mixtures at obsm['theta'] and the topic embeddings at uns['alpha'] (these are produced by model.get_all_embeddings_and_nll). For each dataset define the tested contrasts from obs metadata columns: disease condition (AD vs control; suicidal vs healthy) and cell-type identity. For every topic and contrast, perform a differential topic abundance test with 100,000 label permutations comparing topic loadings between the labeled cell groups, then apply Bonferroni correction over all tested topics within each contrast to call significant topics. For significant topics (AD: 12 and 58; MDD: 1, 52, 68, 70, 86; cell-type-specific: 19/35/50/69/97), inspect the topic gene loadings in varm['rho'] to confirm pathway character, e.g., mitochondrial gene enrichment for topic 58 consistent with beta-amyloid buildup in AD mitochondria. Assemble per-dataset and per-contrast results into a single table of topic IDs, contrasts, permutation p-values, and Bonferroni-adjusted significance.

## Related tools

- **scETM** (Trains the embedded topic model and exports the cell topic mixtures (obsm['theta']), gene embeddings (varm['rho']), and topic embeddings (uns['alpha']) that are contrasted in the permutation test) — https://github.com/hui2000ji/scETM

## Examples

```
model.get_all_embeddings_and_nll(mp); theta = mp.obsm['theta']; rho = mp.varm['rho']; alpha = mp.uns['alpha']
```

## Evaluation signals

- Result table contains topics 12 and 58 for the AD-vs-control contrast with permutation p-value at or below 1e-5 (reported p-value = 1e-5)
- Result table contains topics 1, 52, 68, 70, 86 for the suicidal-vs-healthy (MDD) contrast
- Result table contains topics 19/35/50/69/97 for the AD cell-type contrasts
- Each contrast's significance calls survive Bonferroni correction over all tested topics within that contrast
- Topic 58 gene loadings in varm['rho'] show enrichment for mitochondrial genes, matching the reported pathway character

## Limitations

- The scETM model only considers a single categorical batch variable; multi-batch-variable correction is future work, so confounding batch structure beyond one categorical variable is not modeled
- Interpretation of differential topics depends on gene set enrichment of topic loadings; the reported signal for topic 58 (mitochondrial genes, beta-amyloid buildup in AD mitochondria) is an interpretation of enrichment, not a direct causal measurement

## Evidence

- [readme] Unnormalized cell embeddings will be stored at mp.obsm['delta'], normalized cell embeddings at mp.obsm['theta'], gene embeddings at mp.varm['rho'], topic embeddings at mp.uns['alpha'].: "Unnormalized cell embeddings will be stored at mp.obsm['delta'], normalized cell embeddings at mp.obsm['theta'], gene embeddings at mp.varm['rho'], topic embeddings at mp.uns['alpha']."
- [abstract] Using gene set enrichment analysis, we find that scETM-learned topics are enriched in biologically meaningful and disease-related pathways.: "Using gene set enrichment analysis, we find that scETM-learned topics are enriched in biologically meaningful and disease-related pathways."
- [methods] scETM requires a cells-by-genes matrix `adata` as input, in the format of an AnnData object: "scETM requires a cells-by-genes matrix `adata` as input, in the format of an AnnData object"
