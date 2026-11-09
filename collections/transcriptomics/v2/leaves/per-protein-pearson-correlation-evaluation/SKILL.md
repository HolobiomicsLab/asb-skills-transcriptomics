---
name: per-protein-pearson-correlation-evaluation
description: Use when a reference-mapping workflow predicts per-cell protein or other
  modal annotations from mRNA-only query cells and you need a quantitative per-feature
  accuracy metric against measured values.
when_to_use_negative:
- When the query retains measured protein measurements (true CITE-seq query) — direct
  comparison, not imputation evaluation, is appropriate
- When evaluating cell-type label transfer accuracy rather than continuous per-feature
  (protein) prediction fidelity
- When reference and query are not cell-type-matched (novel populations in query),
  since correlations are only meaningful for proteins expressed in reference-covered
  states
edam_operation: http://edamontology.org/operation_0570
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3308
tools:
- name: Symphony
  role: Provides mapQuery to project mRNA-only query cells into the reference embedding
    and knnPredict to infer per-cell surface protein abundance from the 50 nearest
    reference neighbors
  repo: https://github.com/immunogenomics/symphony
- name: Harmony
  role: Produces the CCA-integrated multimodal embedding over which the reference
    is built and the correlation evaluation is performed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-25957-x
    title: Efficient and precise single-cell reference atlas mapping with Symphony
  source_content_hash: sha256:a5951d724d4d63f0a7435a91f64c522e9239ac3b8d3cb6ace2abfca4ee409b77
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/immunogenomics/symphony
  tools:
  - Harmony
  - Symphony
  edam_operation: http://edamontology.org/operation_0570
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# per-protein-pearson-correlation-evaluation

## Summary

Evaluation skill that scores Symphony-derived surface protein imputation by computing per-protein Pearson correlations between predicted (knnPredict, 50-NN in the harmonized embedding) and ground-truth measured protein levels, aggregated per held-out donor from the CITE-seq memory T-cell reference experiment.

## When to use

Use when a reference-mapping workflow predicts per-cell protein or other modal annotations from mRNA-only query cells and you need a quantitative per-feature accuracy metric against measured values.

## When NOT to use

- When the query retains measured protein measurements (true CITE-seq query) — direct comparison, not imputation evaluation, is appropriate
- When evaluating cell-type label transfer accuracy rather than continuous per-feature (protein) prediction fidelity
- When reference and query are not cell-type-matched (novel populations in query), since correlations are only meaningful for proteins expressed in reference-covered states

## Inputs

- Per-cell predicted surface protein abundances from knnPredict on mapped query cells (50-NN in harmonized embedding)
- Ground-truth measured surface protein abundances for the 54 held-out mRNA-only query donors
- Query metadata (donor labels) for per-donor aggregation
- Baseline predictions from the mRNA-only PCA reference for comparison

## Outputs

- cite_seq_protein_correlations.csv containing per-protein Pearson r values for the CCA reference and the PCA-only baseline
- Per-protein correlation summary across all 54 query samples

## How to apply

After mapping the 54 held-out mRNA-only query donors onto the multimodal CITE-seq reference (395,373 cells from 217 training donors) with symphony::mapQuery, infer per-cell surface protein abundance using knnPredict with k=50 nearest neighbors in the reference embedding. Aggregate predicted and measured protein levels per held-out donor and compute a Pearson correlation for each protein, yielding per-protein r values. For the successful CCA-based multimodal reference, per-protein correlations should fall in the 0.88-0.99 range; repeating the evaluation against an mRNA-only PCA reference (no CCA/protein information) gives the weaker baseline of r 0.65-0.97. Write both sets of per-protein correlations to cite_seq_protein_correlations.csv and validate that all values lie within the expected bands for the respective reference type.

## Related tools

- **Symphony** (Provides mapQuery to project mRNA-only query cells into the reference embedding and knnPredict to infer per-cell surface protein abundance from the 50 nearest reference neighbors) — https://github.com/immunogenomics/symphony
- **Harmony** (Produces the CCA-integrated multimodal embedding over which the reference is built and the correlation evaluation is performed)

## Examples

```
query = knnPredict(query, refs, K = 50); cor_per_protein <- sapply(proteins, function(p) cor(aggregate(predicted[,p], by=list(donor), mean)[,2], aggregate(measured[,p], by=list(donor), mean)[,2]))
```

## Evaluation signals

- cite_seq_protein_correlations.csv contains per-protein Pearson r values for both the CCA reference and the PCA-only baseline
- 50-NN CCA-reference per-protein correlations all fall within the reported 0.88-0.99 band
- PCA-reference (mRNA-only) baseline per-protein correlations fall within the 0.65-0.97 band
- Correlations are computed after aggregating predicted versus measured protein levels per held-out donor across all 54 query samples
- Strong concordance is observed for all proteins, not just a subset, when using the multimodal CCA reference

## Limitations

- The CCA strategy used for CITE-seq multimodal integration may not be optimal for all applications, as multimodal single-cell integration is an active research area
- Performance depends on comprehensive cell type coverage in the reference; novel populations not covered degrade imputation fidelity
- If a query population is very similar to an existing reference population, mapping metrics may be less informative about mapping quality

## Evidence

- [abstract] Symphony was demonstrated on inferring surface protein expression with a multimodal CITE-seq atlas of memory T cells: [evidence span withheld — anchor status: not_found]
- [methods] knnPredict filter step: "query = knnPredict(query,       # query object"
- [methods] mapQuery filter step: "query = mapQuery(query_exp,"
- [discussion] two-step annotation transfer: "We first learn a predictive model in the reference embedding, and then map query cells and use their reference coordinates to predict query annotations."
- [discussion] multimodal integration caveat: "multimodal single-cell integration is an important area of active research."
