---
name: permutation-based-significance-testing
description: Use when you need to test which scETM-learned topics are differentially
  abundant between labeled cell groups (e.g., AD vs control, suicidal vs healthy,
  or across cell types) and require nonparametric p-values with multiple-testing control.
when_to_use_negative:
- The cell groups lack reliable condition or cell-type labels in obs — permutation
  of unlabeled cells cannot define the contrast
- You need differential expression at the gene level rather than differential abundance
  of learned topic loadings
- The AnnData has not been run through scETM training and lacks obsm['theta'] topic
  mixtures
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scETM
  role: Trains the topic model whose theta topic mixtures are permuted and tested;
    supplies alpha and rho embeddings for interpreting significant topics
  repo: github.com/hui2000ji/scETM
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-25534-2
    title: Learning interpretable cellular and gene signature embeddings from single-cell
      transcriptomic data
  source_content_hash: sha256:a30804349818bd9b61860da733a44e5616424c5230d1ab1c1e530d0149c4dde7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scETM
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# permutation-based-significance-testing

## Summary

Nonparametric differential-topic testing of scETM cell topic mixtures: for each topic and contrast, topic loadings in obsm['theta'] are compared between labeled cell groups using 100,000 label permutations, with Bonferroni correction over all topics, to call significantly differential topics in disease vs control and across cell types.

## When to use

Use when you need to test which scETM-learned topics are differentially abundant between labeled cell groups (e.g., AD vs control, suicidal vs healthy, or across cell types) and require nonparametric p-values with multiple-testing control.

## When NOT to use

- The cell groups lack reliable condition or cell-type labels in obs — permutation of unlabeled cells cannot define the contrast
- You need differential expression at the gene level rather than differential abundance of learned topic loadings
- The AnnData has not been run through scETM training and lacks obsm['theta'] topic mixtures

## Inputs

- AnnData object with trained scETM cell topic mixtures in obsm['theta']
- scETM topic embeddings in uns['alpha'] and gene loadings in varm['rho']
- obs metadata columns with disease condition and cell-type labels (batch_col/cell_type_col conventions: 'batch_indices', 'cell_types')

## Outputs

- Table of topic IDs, contrasts, permutation p-values, and Bonferroni-adjusted significance per dataset and contrast
- Set of significant topics per contrast (AD: 12 and 58; MDD: 1, 52, 68, 70, 86; cell-type-specific: 19/35/50/69/97)
- Pathway characterization of significant topics (e.g., mitochondrial enrichment for topic 58)

## How to apply

Load trained scETM outputs from the AnnData objects, reading normalized cell topic mixtures at obsm['theta'], topic embeddings at uns['alpha'], and gene loadings at varm['rho']. Define the contrasts from obs metadata: disease condition (AD vs control; suicidal vs healthy) and cell-type identity. For every topic and contrast, compare topic loadings between the labeled groups with 100,000 label permutations, then apply Bonferroni correction over all tested topics within each contrast to call significant topics. Finally inspect the significant topics' gene loadings (varm['rho']) to confirm pathway character, e.g., mitochondrial gene enrichment for topic 58.

## Related tools

- **scETM** (Trains the topic model whose theta topic mixtures are permuted and tested; supplies alpha and rho embeddings for interpreting significant topics) — github.com/hui2000ji/scETM

## Examples

```
from scETM import scETM, UnsupervisedTrainer; model.get_all_embeddings_and_nll(mp)  # then permute labels on mp.obsm['theta'] across 100,000 permutations per topic/contrast
```

## Evaluation signals

- Results table contains topics 12 and 58 for the AD-vs-control contrast with permutation p-value at or below 1e-5
- Topics 1/52/68/70/86 are recovered for the suicidal-vs-healthy contrast in the MDD dataset
- Topics 19/35/50/69/97 are recovered for the AD cell-type contrasts
- Bonferroni adjustment was applied within each contrast over the full set of tested topics
- Gene loadings of topic 58 (varm['rho']) show mitochondrial gene enrichment consistent with beta-amyloid buildup in AD mitochondria

## Limitations

- Bonferroni correction over all topics is conservative and may miss weaker differential topics within a contrast
- Significance relies on the trained scETM fit; poorly converged topics (few gradient updates) could produce spurious or unstable differential calls
- Reported findings (topic IDs, p-values) are specific to the AD and MDD datasets and contrasts; other datasets require re-running the full permutation scheme

## Evidence

- [results] AD finding with expected p-value threshold: [evidence span withheld — anchor status: not_found]
- [methods] Permutation test parameters: [evidence span withheld — anchor status: not_found]
- [readme] Where scETM stores the tested embeddings: "Unnormalized cell embeddings will be stored at mp.obsm['delta'], normalized cell embeddings at mp.obsm['theta'], gene embeddings at mp.varm['rho'], topic embeddings at mp.uns['alpha']."
