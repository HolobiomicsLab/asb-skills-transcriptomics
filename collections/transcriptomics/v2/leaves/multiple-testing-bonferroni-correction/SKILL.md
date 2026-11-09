---
name: multiple-testing-bonferroni-correction
description: Use when many topics/features are tested per contrast (e.g. scETM topic
  abundance across disease vs control or cell types) and a family-wise-error-controlled
  significance call is needed.
when_to_use_negative:
- When the goal is controlling the false discovery rate rather than family-wise error
  (Bonferroni is the most conservative choice and may miss weaker but real topics)
- When only one hypothesis is tested per contrast, where no correction is needed
- When corrected tests span multiple different contrasts that should each have their
  own family of hypotheses (correction is applied over all tested topics within each
  contrast, not pooled across contrasts)
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scETM
  role: Provides the topic model outputs (obsm['theta'] cell topic mixtures, uns['alpha']
    topic embeddings, varm['rho'] gene loadings) on which the permutation tests and
    Bonferroni correction are performed
  repo: https://github.com/hui2000ji/scETM
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-25534-2
    title: Learning interpretable cellular and gene signature embeddings from single-cell
      transcriptomic data
  source_content_hash: sha256:75606b8e34479916f5a49be49066f66a8f1a7d3dc75da95e49dd1c6b9eaab3d1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hui2000ji/scETM
  tools:
  - scETM
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# multiple-testing-bonferroni-correction

## Summary

A conservative multiple-testing correction applied to per-topic permutation p-values in scETM differential topic-abundance analysis, so that significantly differential topics (e.g. AD topics 12 and 58) are called within each tested contrast.

## When to use

Use when many topics/features are tested per contrast (e.g. scETM topic abundance across disease vs control or cell types) and a family-wise-error-controlled significance call is needed.

## When NOT to use

- When the goal is controlling the false discovery rate rather than family-wise error (Bonferroni is the most conservative choice and may miss weaker but real topics)
- When only one hypothesis is tested per contrast, where no correction is needed
- When corrected tests span multiple different contrasts that should each have their own family of hypotheses (correction is applied over all tested topics within each contrast, not pooled across contrasts)

## Inputs

- Per-topic permutation test p-values from 100,000-label-permutation differential topic abundance tests on scETM cell topic mixtures (obsm['theta'])
- Contrast labels (disease condition and cell type) from AnnData obs metadata columns
- Topic gene loadings varm['rho'] for post-hoc pathway inspection of significant topics

## Outputs

- Bonferroni-adjusted significance calls per topic per contrast
- A results table listing topic IDs, contrasts, permutation p-values, and Bonferroni-adjusted significance (e.g. AD: topics 12, 58; MDD: topics 1, 52, 68, 70, 86; cell-type topics 19/35/50/69/97)

## How to apply

For every topic and contrast (e.g. AD vs control, suicidal vs healthy, or AD cell-type contrasts), perform a differential topic abundance test on the normalized cell topic mixtures in obsm['theta'] with 100,000 label permutations, comparing topic loadings between the labeled cell groups from the obs metadata. Collect the raw permutation p-values for all tested topics within a single contrast, then apply the Bonferroni correction by multiplying each p-value by the number of tested topics (equivalently, comparing p to alpha/n_topics) so family-wise error across topics is controlled within that contrast. Only topics surviving Bonferroni adjustment are called significantly differential, and their gene loadings from varm['rho'] are inspected to confirm pathway character (e.g. mitochondrial gene enrichment for topic 58). Assemble topic IDs, contrasts, permutation p-values, and Bonferroni-adjusted significance into a single results table.

## Related tools

- **scETM** (Provides the topic model outputs (obsm['theta'] cell topic mixtures, uns['alpha'] topic embeddings, varm['rho'] gene loadings) on which the permutation tests and Bonferroni correction are performed) — https://github.com/hui2000ji/scETM

## Examples

```
from statsmodels.stats.multitest import multipletests; reject, p_adj, _, _ = multipletests(topic_pvals, alpha=0.05, method='bonferroni')
```

## Evaluation signals

- The AD-vs-control contrast yields topics 12 and 58 with permutation p-value at or below 1e-5 after Bonferroni correction
- The suicidal-vs-healthy contrast yields topics 1, 52, 68, 70, and 86 as significant
- The AD cell-type contrasts yield topics 19, 35, 50, 69, and 97 as significant
- Topic 58's gene loadings (varm['rho']) show enrichment for mitochondrial genes, consistent with beta-amyloid buildup in AD mitochondria
- The number of tests used in the Bonferroni multiplier equals the number of topics tested within each contrast

## Limitations

- Bonferroni is conservative; with many topics tested per contrast, genuinely differential topics with borderline permutation p-values may be discarded
- Correction must be re-applied per contrast; pooling p-values across disease and cell-type contrasts would over-penalize and is not the reported procedure

## Evidence

- [methods] permutation p-value and Bonferroni workflow: [evidence span withheld — anchor status: not_found]
- [methods] Bonferroni correction step: [evidence span withheld — anchor status: not_found]
- [results] AD finding: [evidence span withheld — anchor status: not_found]
- [readme] scETM embedding locations: "Unnormalized cell embeddings will be stored at mp.obsm['delta'], normalized cell embeddings at mp.obsm['theta'], gene embeddings at mp.varm['rho']"
