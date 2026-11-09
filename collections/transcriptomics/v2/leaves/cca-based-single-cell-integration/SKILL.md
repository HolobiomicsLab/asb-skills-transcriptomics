---
name: cca-based-single-cell-integration
description: Use when query cells lack surface-protein measurements (mRNA-only scRNA-seq)
  but a multimodal CITE-seq reference atlas of the same tissue (e.g. memory T cells)
  exists and per-cell protein abundance must be predicted by reference mapping.
when_to_use_negative:
- The goal is de novo discovery of novel cell types absent from the reference — Symphony
  mapping metrics may be uninformative and 'Symphony does not entirely supplant the
  need for users interested in novel cell type discovery to conduct de novo analyses
  of the query alone.'
- The reference is itself mRNA-only (no CITE-seq protein data or CCA integration)
  — protein inference cannot exceed the PCA-only baseline and the CCA advantage is
  lost.
- Multimodal integration beyond CCA is required for the modality pair at hand — the
  paper cautions that 'multimodal single-cell integration is an important area of
  active research' and the CCA strategy may not be optimal for all applications.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Symphony
  role: Builds the compressed multimodal reference (buildReference), maps mRNA-only
    queries into the reference embedding (mapQuery), and predicts surface protein
    annotations via k-nearest neighbors (knnPredict)
  repo: https://github.com/immunogenomics/symphony
- name: Harmony
  role: Performs the donor-integration step underlying the reference embedding, run
    with return_object = TRUE so the Harmony object can be compressed by Symphony
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-25957-x
    title: Efficient and precise single-cell reference atlas mapping with Symphony
  source_content_hash: sha256:2b899949ddac8af2154161e6cd5caf19975b9ea46db223bc510e8dfe32900214
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/immunogenomics/symphony
  tools:
  - Harmony
  - Symphony
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# cca-based-single-cell-integration

## Summary

Builds a CCA-based multimodal Symphony reference from CITE-seq memory T-cell data and maps mRNA-only query cells onto it to infer surface protein expression via 50-NN prediction in the harmonized embedding. Demonstrated by Kang et al. (Nat Comm 2021) to yield per-protein Pearson correlations of 0.88-0.99 against a 0.65-0.97 PCA-only baseline.

## When to use

Use when query cells lack surface-protein measurements (mRNA-only scRNA-seq) but a multimodal CITE-seq reference atlas of the same tissue (e.g. memory T cells) exists and per-cell protein abundance must be predicted by reference mapping.

## When NOT to use

- The goal is de novo discovery of novel cell types absent from the reference — Symphony mapping metrics may be uninformative and 'Symphony does not entirely supplant the need for users interested in novel cell type discovery to conduct de novo analyses of the query alone.'
- The reference is itself mRNA-only (no CITE-seq protein data or CCA integration) — protein inference cannot exceed the PCA-only baseline and the CCA advantage is lost.
- Multimodal integration beyond CCA is required for the modality pair at hand — the paper cautions that 'multimodal single-cell integration is an important area of active research' and the CCA strategy may not be optimal for all applications.

## Inputs

- CITE-seq reference expression matrix and metadata for training donors (joint mRNA + surface protein ADT measurements, e.g. 217 memory T-cell donors, 395,373 cells)
- mRNA-only query expression matrix and metadata for held-out donors (e.g. 54 donors)
- Per-donor variable gene means/standard deviations (vargenes_means_sds) for consistent scaling of reference and query

## Outputs

- Symphony reference object embedding 217 donors in a CCA/Harmony harmonized low-dimensional space
- Per-cell predicted surface protein abundances for each held-out query cell (50-NN in reference embedding)
- cite_seq_protein_correlations.csv with per-protein Pearson r for the CCA reference (expected 0.88-0.99) and the PCA-only baseline (expected 0.65-0.97)

## How to apply

Load the CITE-seq reference expression and metadata (here 217 training donors, 395,373 cells retaining joint mRNA/protein structure) and split off held-out mRNA-only donors as the query. Select variable genes with vargenes_vst per donor, scale with scaleDataWithStats, run PCA via SVD, then run Harmony integration with return_object = TRUE to remove donor confounding while CCA holds the multimodal mRNA/protein structure. Compress the Harmony object into a Symphony reference with symphony::buildReference (or buildReferenceFromHarmonyObj), map each held-out mRNA-only query with mapQuery using identical normalization, and infer per-cell protein abundance with knnPredict using k = 50 nearest neighbors in the harmonized embedding. Repeat against an mRNA-only PCA reference (no CCA/protein information) as a baseline, then aggregate predicted vs measured protein per donor and compute per-protein Pearson correlations.

## Related tools

- **Symphony** (Builds the compressed multimodal reference (buildReference), maps mRNA-only queries into the reference embedding (mapQuery), and predicts surface protein annotations via k-nearest neighbors (knnPredict)) — https://github.com/immunogenomics/symphony
- **Harmony** (Performs the donor-integration step underlying the reference embedding, run with return_object = TRUE so the Harmony object can be compressed by Symphony)

## Examples

```
query = mapQuery(query_exp, query_metadata, reference, query_vargenes = vargenes, do_normalize = TRUE); query = knnPredict(query, reference, k = 50)
```

## Evaluation signals

- cite_seq_protein_correlations.csv contains per-protein Pearson r values for both the CCA reference and the PCA-only baseline across all 54 query samples
- 50-NN CCA-reference correlations fall within the reported 0.88-0.99 band for all proteins
- PCA-only reference baseline correlations fall within the 0.65-0.97 band
- Per-protein Pearson r from the CCA reference exceeds the matching PCA-reference baseline correlation
- Query cells localize in the reference embedding: knnPredict is run with k = 50 neighbors in the harmonized (CCA/Harmony) embedding, not raw expression space

## Limitations

- Symphony mapping metrics may be less informative when a novel population is very similar to an existing reference population
- Symphony does not replace de novo analyses of the query alone for users interested in novel cell type discovery
- The CCA strategy used for CITE-seq integration may not be optimal for all multimodal applications
- Identifying novel cell types that failed to map is an unsolved problem; Condition I (comprehensive cell type coverage in the reference) limits flexibility relative to de novo Harmony integration

## Evidence

- [abstract] Demonstration on CITE-seq surface protein inference: [evidence span withheld — anchor status: not_found]
- [abstract] Stable embedding enables annotation transfer: "Symphony localizes query cells within a stable low-dimensional reference embedding, facilitating reproducible downstream transfer of reference-defined annotations"
- [methods] buildReference call: "reference = symphony::buildReference("
- [methods] knnPredict call: "query = knnPredict(query,       # query object"
- [discussion] Multimodal integration caveat: "multimodal single-cell integration is an important area of active research."
- [readme] Symphony availability in R: "Symphony is available on CRAN:

 r
install.packages("symphony")"
