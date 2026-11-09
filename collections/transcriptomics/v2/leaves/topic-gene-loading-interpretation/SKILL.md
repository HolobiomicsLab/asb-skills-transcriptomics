---
name: topic-gene-loading-interpretation
description: Use when scETM topics have been flagged as differentially expressed (e.g.,
  AD topics 12 and 58 at permutation p ≤ 1e-5) and their biological character must
  be confirmed from gene loadings.
when_to_use_negative:
- When no topics have been called significant by the differential abundance test —
  loading inspection is most meaningful for pre-selected topics
- When the goal is batch correction or cell-type clustering rather than topic interpretability
  (tools like Harmony or Scanorama are more appropriate)
- When the gene embedding matrix ρ was fixed (trainable_gene_emb_dim = 0 in p-scETM)
  and topic-gene associations are constrained by external pathway sets rather than
  learned from data
edam_operation: http://edamontology.org/operation_0533
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scETM
  role: Trains the embedded topic model and provides the gene embedding matrix rho
    (varm['rho']) and topic embeddings alpha (uns['alpha']) whose loadings are inspected
    for pathway interpretation
  repo: github.com/hui2000ji/scETM
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-25534-2
    title: Learning interpretable cellular and gene signature embeddings from single-cell
      transcriptomic data
  source_content_hash: sha256:8d8f50e3c322d549f627644f08699404a7484a7ee0488a2dccd34e7b93b09852
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scETM
  edam_operation: http://edamontology.org/operation_0533
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# topic-gene-loading-interpretation

## Summary

Interpret scETM-learned topics by inspecting the topic gene loadings (varm['rho']) and topic embeddings (uns['alpha']) to characterize topics as biologically meaningful, disease-related pathways — e.g., confirming mitochondrial gene enrichment for topic 58 in the Alzheimer's Disease differential analysis.

## When to use

Use when scETM topics have been flagged as differentially expressed (e.g., AD topics 12 and 58 at permutation p ≤ 1e-5) and their biological character must be confirmed from gene loadings.

## When NOT to use

- When no topics have been called significant by the differential abundance test — loading inspection is most meaningful for pre-selected topics
- When the goal is batch correction or cell-type clustering rather than topic interpretability (tools like Harmony or Scanorama are more appropriate)
- When the gene embedding matrix ρ was fixed (trainable_gene_emb_dim = 0 in p-scETM) and topic-gene associations are constrained by external pathway sets rather than learned from data

## Inputs

- AnnData object with scETM outputs: normalized cell topic mixtures at obsm['theta'], gene embeddings at varm['rho'], topic embeddings at uns['alpha']
- List of significant topic IDs from the differential topic abundance test (e.g., AD: 12, 58; MDD: 1, 52, 68, 70, 86; cell-type-specific: 19/35/50/69/97)
- Optional: pathDIP4 pathways-by-genes matrix for p-scETM pathway-constrained interpretation

## Outputs

- Per-topic ranked gene loading lists from varm['rho'] showing which genes characterize each topic
- Interpretation table mapping significant topic IDs to confirmed pathway character (e.g., topic 58 → mitochondrial genes)
- Gene set enrichment results linking topics to biologically meaningful and disease-related pathways

## How to apply

After training scETM and extracting embeddings with model.get_all_embeddings_and_nll(adata), load the normalized cell topic mixtures from obsm['theta'], the topic embeddings from uns['alpha'], and the topic gene loadings from varm['rho']. For each significant topic identified by the differential topic abundance test (100,000 label permutations with Bonferroni correction over topics within each contrast), rank genes by their loadings in the corresponding row of varm['rho']. Inspect the top-loaded genes to confirm the pathway character of the topic, e.g., verifying that topic 58 in the AD dataset is highly enriched for mitochondrial genes, consistent with beta-amyloid buildup in AD mitochondria. Optionally strengthen interpretation with gene set enrichment analysis, since scETM topics are known to be enriched in biologically meaningful and disease-related pathways; p-scETM variants can fix the gene embedding matrix ρ to a pathDIP4 pathways-by-genes matrix to directly learn pathway-topic associations.

## Related tools

- **scETM** (Trains the embedded topic model and provides the gene embedding matrix rho (varm['rho']) and topic embeddings alpha (uns['alpha']) whose loadings are inspected for pathway interpretation) — github.com/hui2000ji/scETM

## Examples

```
model.get_all_embeddings_and_nll(mp); rho = mp.varm['rho']; top_genes = rho.loc['topic_58'].sort_values(ascending=False).index[:50]
```

## Evaluation signals

- Topic 58's top-ranked genes in varm['rho'] are dominated by mitochondrial genes, consistent with the reported AD finding
- The interpretation table lists topics 12 and 58 for the AD-vs-control contrast (p ≤ 1e-5), topics 1/52/68/70/86 for suicidal-vs-healthy, and topics 19/35/50/69/97 for AD cell-type contrasts
- Gene set enrichment analysis of topic gene loadings recovers biologically meaningful, disease-related pathways as reported for scETM topics
- Gene loadings align with the pathway-topic associations implied by uns['alpha'] topic embeddings

## Limitations

- Topic interpretation via gene loadings depends on the number of training epochs being sufficient (at least ~6k gradient updates for small datasets) or the topics may not have converged to meaningful gene signatures
- scETM's learned gene embeddings are unconstrained unless the p-scETM variant is used with trainable_gene_emb_dim = 0 to fix ρ to a pathDIP4 pathway matrix
- Interpretation is limited to genes present in the trained cells-by-genes AnnData matrix; genes absent from the dataset cannot inform topic character

## Evidence

- [results] topic 58 highly enriched for mitochondrial genes, consistent with beta-amyloid buildup in AD mitochondria: [evidence span withheld — anchor status: not_found]
- [abstract] scETM-learned topics are enriched in biologically meaningful and disease-related pathways via gene set enrichment analysis: "Using gene set enrichment analysis, we find that scETM-learned topics are enriched in biologically meaningful and disease-related pathways."
- [readme] gene embeddings at mp.varm['rho'], topic embeddings at mp.uns['alpha']: "gene embeddings at mp.varm['rho'], topic embeddings at mp.uns['alpha']"
- [methods] p-scETM fixes gene embedding matrix to pathways-by-genes matrix from pathDIP4: "p-scETM is a variant of scETM where part or all of the the gene embedding matrix ρ is fixed to a pathways-by-genes matrix"
