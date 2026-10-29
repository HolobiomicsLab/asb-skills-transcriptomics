---
name: symphony-query-mapping
description: Use when an mRNA-only query dataset must be placed into an existing integrated
  reference atlas (e.g., a CITE-seq memory T-cell reference of ~395,373 cells) to
  transfer cell-type annotations or infer modalities absent from the query, such as
  surface proteins.
when_to_use_negative:
- The goal is discovery of novel cell types that failed to map — Symphony does not
  replace de novo analyses of the query alone and mapping metrics may be less informative
  for a novel population very similar to an existing reference population
- Query and reference peak sets differ and no standardization exists (e.g., scATAC-seq
  feature selection), since the optimal feature-selection approach for mapping is
  not defined in that setting
- The input is already a feature/annotation table rather than raw single-cell expression
  data requiring projection into a reference embedding
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: Symphony
  role: Builds the compressed integrated reference (buildReference / buildReferenceFromHarmonyObj),
    maps query cells into the reference embedding (mapQuery), and predicts query annotations
    via nearest neighbors (knnPredict)
  repo: https://github.com/immunogenomics/symphony
- name: Harmony
  role: Performs the reference-side integration over donor while retaining the joint
    mRNA/protein structure, run with return_object = TRUE before reference compression
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-25957-x
    title: Efficient and precise single-cell reference atlas mapping with Symphony
  source_content_hash: sha256:e8a78de353d80c10fa21269bdc5380b396a20e21c11812228dd44716da519948
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/immunogenomics/symphony
  tools:
  - Harmony
  - Symphony
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# symphony-query-mapping

## Summary

Builds a compressed Symphony reference from a Harmony-integrated single-cell atlas and maps mRNA-only query cells into the stable reference embedding within seconds, enabling transfer of reference-defined annotations such as per-cell surface protein inference from a multimodal CITE-seq reference.

## When to use

Use when an mRNA-only query dataset must be placed into an existing integrated reference atlas (e.g., a CITE-seq memory T-cell reference of ~395,373 cells) to transfer cell-type annotations or infer modalities absent from the query, such as surface proteins.

## When NOT to use

- The goal is discovery of novel cell types that failed to map — Symphony does not replace de novo analyses of the query alone and mapping metrics may be less informative for a novel population very similar to an existing reference population
- Query and reference peak sets differ and no standardization exists (e.g., scATAC-seq feature selection), since the optimal feature-selection approach for mapping is not defined in that setting
- The input is already a feature/annotation table rather than raw single-cell expression data requiring projection into a reference embedding

## Inputs

- Reference expression matrix and cell metadata (e.g., 217 training donors, 395,373 joint mRNA/protein CITE-seq cells) with donor labels for integration
- mRNA-only query expression matrix and metadata (e.g., 54 held-out donors)
- Harmony-integrated multimodal embedding of the reference (CCA-based joint mRNA/protein space)
- Ground-truth surface protein abundances for held-out query donors (for validation)

## Outputs

- Compressed Symphony reference object in a portable format
- Query object with cells localized in the reference embedding and transferred annotations
- Per-cell inferred surface protein abundances (knnPredict, 50-NN)
- cite_seq_protein_correlations.csv with per-protein Pearson r for the CCA reference and the PCA-only baseline

## How to apply

First select variable genes per group (e.g., vargenes_vst with topn per donor), scale with the reference gene means and standard deviations, run PCA by SVD, and run Harmony with return_object = TRUE so the integration object can be compressed into a Symphony reference via symphony::buildReference (or buildReferenceFromHarmonyObj). Normalize the query with the same strategy as the reference, then project query cells into the reference embedding with mapQuery, which compresses the query against the reference mixture-model parameters. Transfer annotations or infer missing modalities from the query's reference coordinates; for surface-protein inference on a CITE-seq reference, use knnPredict with 50 nearest neighbors in the harmonized embedding. Validate by aggregating predicted versus measured values per query donor and computing per-protein correlations: CCA-based multimodal reference mapping of mRNA-only queries is expected to yield Pearson r of 0.88-0.99 across proteins, versus r of 0.65-0.97 for an mRNA-only PCA baseline lacking CCA/protein information.

## Related tools

- **Symphony** (Builds the compressed integrated reference (buildReference / buildReferenceFromHarmonyObj), maps query cells into the reference embedding (mapQuery), and predicts query annotations via nearest neighbors (knnPredict)) — https://github.com/immunogenomics/symphony
- **Harmony** (Performs the reference-side integration over donor while retaining the joint mRNA/protein structure, run with return_object = TRUE before reference compression)

## Examples

```
query = mapQuery(query_exp, query_metadata, reference, query_var_genes)
```

## Evaluation signals

- Per-protein Pearson correlations between predicted and measured surface proteins across all 54 query samples fall within the 0.88-0.99 band for the 50-NN CCA-based multimodal reference
- Baseline mapping against an mRNA-only PCA reference (no CCA/protein information) yields per-protein correlations in the 0.65-0.97 range, confirming the multimodal reference outperforms the transcriptomic-only baseline
- The output cite_seq_protein_correlations.csv contains per-protein Pearson r values for both the CCA reference and the PCA-only baseline
- Query cells are localized in the same low-dimensional reference embedding as the integrated reference cells, without requiring access to raw data on all individual reference cells
- Harmony integration on the reference was run with return_object = TRUE so the object could be successfully compressed with buildReferenceFromHarmonyObj

## Limitations

- Multimodal single-cell integration is an active research area; the CCA strategy used for CITE-seq may not be optimal for all applications
- Symphony mapping metrics may be less informative when a novel population is very similar to an existing reference population
- Identification of novel cell types that have failed to map is an unsolved direction; Symphony does not replace de novo query analyses for novel cell-type discovery

## Evidence

- [abstract] Symphony localizes query cells within a stable low-dimensional reference embedding, facilitating reproducible downstream transfer of reference-defined annotations to the query: "Symphony localizes query cells within a stable low-dimensional reference embedding, facilitating reproducible downstream transfer of reference-defined annotations to the query"
- [methods] knnPredict predicts query annotations from 50 nearest neighbors in the reference embedding: "query = knnPredict(query,       # query object"
- [discussion] Two-step annotation transfer: learn a predictive model in the reference embedding, then map query cells and use their reference coordinates to predict annotations: "We first learn a predictive model in the reference embedding, and then map query cells and use their reference coordinates to predict query annotations."
- [readme] README confirms Symphony's purpose and citation for efficient reference mapping: "Efficient and precise single-cell reference atlas mapping with Symphony"
