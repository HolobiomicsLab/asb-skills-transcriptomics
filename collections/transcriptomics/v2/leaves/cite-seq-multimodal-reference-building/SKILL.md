---
name: cite-seq-multimodal-reference-building
description: Use when a CITE-seq reference with paired mRNA and surface protein measurements
  is available but query samples are mRNA-only, and you need to infer query surface
  protein levels from reference mapping.
when_to_use_negative:
- The query has measured surface protein data — no inference via mapping is needed
- The goal is discovering novel cell types absent from the reference — Symphony does
  not replace de novo analyses of the query alone
- The reference and query are single-modality only with no protein or other multimodal
  layer to transfer
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: Symphony
  role: Builds the compressed multimodal reference (buildReference) and maps mRNA-only
    query cells (mapQuery, knnPredict) to infer surface protein expression
  repo: https://github.com/immunogenomics/symphony
- name: Harmony
  role: Provides the donor-integrated CCA embedding on which the Symphony reference
    is compressed; reference building requires return_object = TRUE
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-25957-x
    title: Efficient and precise single-cell reference atlas mapping with Symphony
  source_content_hash: sha256:953f3fa6255096e390b2a82aac2ab726e8a67bc3e5a6c5ade083eee37e6cd52f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/immunogenomics/symphony
  tools:
  - Harmony
  - Symphony
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# cite-seq-multimodal-reference-building

## Summary

Build a Symphony reference from a multimodal CITE-seq atlas (joint mRNA + surface protein, CCA-integrated with Harmony) and map mRNA-only query cells onto it to infer their surface protein expression via 50-NN prediction in the harmonized embedding.

## When to use

Use when a CITE-seq reference with paired mRNA and surface protein measurements is available but query samples are mRNA-only, and you need to infer query surface protein levels from reference mapping.

## When NOT to use

- The query has measured surface protein data — no inference via mapping is needed
- The goal is discovering novel cell types absent from the reference — Symphony does not replace de novo analyses of the query alone
- The reference and query are single-modality only with no protein or other multimodal layer to transfer

## Inputs

- CITE-seq memory T-cell reference expression matrix (joint mRNA + surface protein) for 217 training donors (395,373 cells)
- Reference cell metadata (donor, cell type annotations)
- Held-out mRNA-only query expression matrices for 54 donors
- ADT ground-truth surface protein measurements for the held-out donors (for validation)

## Outputs

- Symphony reference object built on the CCA-integrated multimodal embedding
- Mapped query object with projected coordinates in the reference embedding
- Per-cell predicted surface protein abundances (knnPredict, k=50)
- cite_seq_protein_correlations.csv with per-protein Pearson r for the CCA reference and the PCA-only baseline

## How to apply

Load the CITE-seq memory T-cell reference (395,373 cells, 217 training donors) and split off the 54 held-out mRNA-only donors as the query. Build the Symphony reference with symphony::buildReference on the CCA-integrated multimodal embedding, integrating over donor while retaining the joint mRNA/protein structure; the same pipeline with an mRNA-only PCA reference (no CCA/protein information) serves as the baseline. Map each held-out query with mapQuery using the same normalization as the reference, projecting query cells into the reference embedding. Infer per-cell surface protein abundance with knnPredict using k=50 nearest neighbors in the reference embedding. Aggregate predicted vs. measured protein levels per donor, compute per-protein Pearson correlations, and write both reference conditions to cite_seq_protein_correlations.csv; the CCA-based reference should reach r 0.88-0.99 versus 0.65-0.97 for the PCA-only baseline.

## Related tools

- **Symphony** (Builds the compressed multimodal reference (buildReference) and maps mRNA-only query cells (mapQuery, knnPredict) to infer surface protein expression) — https://github.com/immunogenomics/symphony
- **Harmony** (Provides the donor-integrated CCA embedding on which the Symphony reference is compressed; reference building requires return_object = TRUE)

## Examples

```
reference = symphony::buildReference(exprs_ref, meta_ref, vargenes, harmony_obj, vars = 'donor', return_object = TRUE); query = mapQuery(query_exp, query_meta, reference, vars = 'donor'); query = knnPredict(query, reference, ref_exp_adt, k = 50)
```

## Evaluation signals

- cite_seq_protein_correlations.csv contains per-protein Pearson r values for both the CCA reference and the PCA-only baseline across all 54 query samples
- 50-NN knnPredict correlations against the CCA-based multimodal reference fall within the reported 0.88-0.99 band for all proteins
- Baseline correlations against the mRNA-only PCA reference fall within 0.65-0.97, confirming the multimodal CCA embedding outperforms mRNA-only mapping
- Query cells are localized in the same low-dimensional embedding as the integrated reference cells after mapQuery, verifying successful projection

## Limitations

- Multimodal single-cell integration is an active research area; the CCA strategy used for CITE-seq may not be optimal for all applications
- Symphony does not replace de novo analyses of the query alone for users interested in novel cell type discovery
- Symphony mapping metrics may be less informative when a novel population is very similar to an existing reference population

## Evidence

- [abstract] Symphony was demonstrated on inferring surface protein expression with a multimodal CITE-seq atlas of memory T cells: [evidence span withheld — anchor status: not_found]
- [methods] knnPredict method step: "query = knnPredict(query,       # query object"
- [discussion] Two-step annotation transfer: "We first learn a predictive model in the reference embedding, and then map query cells and use their reference coordinates to predict query annotations."
- [readme] Symphony installable from CRAN: "Symphony is available on CRAN:

 r
install.packages("symphony")
"
