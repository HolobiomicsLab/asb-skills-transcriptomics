---
name: benjamini-hochberg-multiple-testing-correction
description: Use when a single analysis produces many pathway-enrichment p-values
  (e.g. 100 scETM topics tested against MSigDB Canonical Pathways or PsyGeNET gene
  sets) and significant topic-pathway associations must be controlled for false discovery.
when_to_use_negative:
- Testing a single pre-specified hypothesis (one topic against one pathway), where
  no family-wise correction is needed
- Input p-values are not from an exchangeable permutation structure (e.g. dependent
  ad-hoc scores without permutation calibration), where BH FDR assumptions are questionable
- Ranking topics purely exploratorily without making any significance claim — correction
  would only add conservative distortion
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scETM
  role: Supplies the 100-topic models, learned gene (rho) and topic (alpha) embeddings,
    and per-topic gene rankings whose GSEA p-value family the BH correction is applied
    over
  repo: github.com/hui2000ji/scETM
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-25534-2
    title: Learning interpretable cellular and gene signature embeddings from single-cell
      transcriptomic data
  source_content_hash: sha256:c3fdfba1df34fc11f465bf6d85830be46ecd09bb87674bb834c5980f2d25a584
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scETM
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# benjamini-hochberg-multiple-testing-correction

## Summary

Applies Benjamini-Hochberg False Discovery Rate correction to the large family of topic-by-pathway p-values produced by permutation-based GSEA of scETM-learned topics, converting raw enrichment p-values into BH-adjusted q-values used to declare biologically meaningful topic-pathway associations.

## When to use

Use when a single analysis produces many pathway-enrichment p-values (e.g. 100 scETM topics tested against MSigDB Canonical Pathways or PsyGeNET gene sets) and significant topic-pathway associations must be controlled for false discovery.

## When NOT to use

- Testing a single pre-specified hypothesis (one topic against one pathway), where no family-wise correction is needed
- Input p-values are not from an exchangeable permutation structure (e.g. dependent ad-hoc scores without permutation calibration), where BH FDR assumptions are questionable
- Ranking topics purely exploratorily without making any significance claim — correction would only add conservative distortion

## Inputs

- Vector of raw permutation GSEA p-values, one per topic-pathway pair (topics from 100-topic scETM models on HP, AD, MDD)
- Gene-set collections: MSigDB Canonical Pathways and PsyGeNET gene sets
- Per-topic gene scores derived from gene embedding matrix mp.varm['rho'] and topic embedding matrix mp.uns['alpha']

## Outputs

- BH-adjusted q-values for every topic-pathway pair
- Significant topic-pathway association table (topic_pathway_enrichment.tsv), e.g. HP Topic 54 / Insulin Processing with BH-adjusted q-value of 0

## How to apply

Rank genes by their per-topic score computed from the learned gene embedding matrix (mp.varm['rho']) combined with the topic embedding matrix (mp.uns['alpha']), then run a permutation-based GSEA with 1000 cell-label shuffles to obtain one raw p-value per topic-pathway pair. Collect all raw p-values across all topics and all gene-set collections (MSigDB Canonical Pathways for HP and AD, PsyGeNET for MDD) into a single vector and apply the Benjamini-Hochberg step-up procedure to derive adjusted q-values that control the FDR. Declare a topic-pathway association significant only at BH-corrected significance; the rationale is that with 100 topics x many pathways, raw p < 0.05 hits would be dominated by chance associations. Use the adjusted q-values to populate the final enrichment table (topic_pathway_enrichment.tsv) and to confirm expected hits such as HP Topic 54 / Insulin Processing.

## Related tools

- **scETM** (Supplies the 100-topic models, learned gene (rho) and topic (alpha) embeddings, and per-topic gene rankings whose GSEA p-value family the BH correction is applied over) — github.com/hui2000ji/scETM

## Examples

```
from statsmodels.stats.multitest import multipletests; qvals = multipletests(raw_gsea_pvalues, method='fdr_bh')[1]
```

## Evaluation signals

- All returned q-values lie in [0, 1] and q >= raw p for every tested topic-pathway pair (monotonicity of BH adjustment)
- Positive-control associations appear at BH-corrected significance: HP Topic 54 / Insulin Processing, and AD topics / Reactome Amyloid Fiber Formation and KEGG AD
- The reported HP Topic 54 Insulin Processing enrichment has a BH-adjusted q-value of 0 (or effectively zero after floating-point rounding)
- Number of significant topic-pathway pairs after BH correction is far smaller than the number of raw p < 0.05 hits, consistent with FDR control across ~100 topics x many pathways
- GSEA was run with the specified 1000 cell-label shuffles before correction, so p-values are permutation-calibrated

## Limitations

- BH control assumes the raw p-values are valid (uniform under the null); GSEA p-values depend on the chosen permutation scheme (1000 cell-label shuffles) and gene-set definition
- Keeping only pathways with more than 5 genes changes the tested family size and hence the correction denominator
- Very small datasets (e.g. MousePancreas with 1886 cells) may yield underpowered enrichment tests whose corrected q-values are uninformative

## Evidence

- [results] BH-corrected GSEA finding: [evidence span withheld — anchor status: not_found]
- [methods] GSEA procedure with BH correction: [evidence span withheld — anchor status: not_found]
- [abstract] Article-level support for pathway enrichment: "Using gene set enrichment analysis, we find that scETM-learned topics are enriched in biologically meaningful and disease-related pathways."
