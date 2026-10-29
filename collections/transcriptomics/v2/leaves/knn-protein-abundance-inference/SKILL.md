---
name: knn-protein-abundance-inference
description: Use when a query dataset measures only mRNA (no CITE-seq ADT counts)
  but you need surface protein levels, and a multimodal CITE-seq reference atlas with
  a joint mRNA/protein embedding is available.
when_to_use_negative:
- The query already contains measured ADT/surface protein counts, so direct measurement
  rather than KNN prediction should be used
- Novel cell populations or cell types absent from the reference are expected — Symphony
  mapping metrics are less informative and it 'does not entirely supplant the need
  for users interested in novel cell type discovery to conduct de novo analyses of
  the query alone'
- No multimodal (mRNA + protein) reference embedding exists; protein inference from
  an mRNA-only PCA reference degrades correlations substantially (baseline r 0.65-0.97
  vs 0.88-0.99)
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: Symphony
  role: Builds the compressed multimodal reference (buildReference), maps mRNA-only
    query cells (mapQuery), and infers surface protein levels via k-nearest-neighbor
    prediction (knnPredict, k=50) in the harmonized embedding
  repo: https://github.com/immunogenomics/symphony
- name: Harmony
  role: Provides the CCA-based donor-integrated multimodal embedding on which the
    Symphony reference is built
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-25957-x
    title: Efficient and precise single-cell reference atlas mapping with Symphony
  source_content_hash: sha256:133e7b2bb2447dee123694db0d63edd119c35c02b2401bd022b0be9459a0e67d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/immunogenomics/symphony
  tools:
  - Harmony
  - Symphony
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# knn-protein-abundance-inference

## Summary

Infers per-cell surface protein abundance for mRNA-only query cells by predicting protein levels from the k nearest reference neighbors (k=50) in a Symphony-harmonized multimodal CITE-seq embedding, enabling annotation transfer beyond cell-type labels.

## When to use

Use when a query dataset measures only mRNA (no CITE-seq ADT counts) but you need surface protein levels, and a multimodal CITE-seq reference atlas with a joint mRNA/protein embedding is available.

## When NOT to use

- The query already contains measured ADT/surface protein counts, so direct measurement rather than KNN prediction should be used
- Novel cell populations or cell types absent from the reference are expected — Symphony mapping metrics are less informative and it 'does not entirely supplant the need for users interested in novel cell type discovery to conduct de novo analyses of the query alone'
- No multimodal (mRNA + protein) reference embedding exists; protein inference from an mRNA-only PCA reference degrades correlations substantially (baseline r 0.65-0.97 vs 0.88-0.99)

## Inputs

- CITE-seq memory T-cell reference expression matrix with joint mRNA and surface protein measurements (395,373 cells, 217 training donors)
- Reference metadata (donor, cell type annotations)
- Held-out mRNA-only query samples (54 donors) with matching gene normalization
- Ground-truth surface protein measurements for the query donors (for validation)

## Outputs

- Per-cell predicted surface protein abundance for each query cell
- Per-protein Pearson correlations between predicted and measured protein levels per held-out donor (e.g. cite_seq_protein_correlations.csv)

## How to apply

Build the reference on the CCA-integrated multimodal embedding from the CITE-seq memory T-cell atlas (395,373 cells, 217 training donors) with symphony::buildReference, integrating over donor so the joint mRNA/protein structure is retained. Map each mRNA-only query sample with mapQuery using the same normalization as the reference, projecting query cells into the reference embedding. Then call knnPredict with k=50 nearest neighbors in the harmonized embedding to infer per-cell protein abundance; the two-step strategy (learn a model in the reference embedding, then predict from query coordinates) is the paper's stated annotation-transfer approach. Baseline against an mRNA-only PCA reference (no CCA/protein information) to confirm the multimodal embedding's contribution. Validate by aggregating predicted vs measured proteins per held-out donor and computing per-protein Pearson correlations.

## Related tools

- **Symphony** (Builds the compressed multimodal reference (buildReference), maps mRNA-only query cells (mapQuery), and infers surface protein levels via k-nearest-neighbor prediction (knnPredict, k=50) in the harmonized embedding) — https://github.com/immunogenomics/symphony
- **Harmony** (Provides the CCA-based donor-integrated multimodal embedding on which the Symphony reference is built)

## Examples

```
reference = symphony::buildReference(expr, meta, vars, k = 50); query = mapQuery(query_exp, query_meta, reference, vars); query = knnPredict(query, reference, celltypes, k = 50)
```

## Evaluation signals

- Per-protein Pearson correlations between predicted and measured proteins across the 54 held-out mRNA-only donors fall in the 0.88-0.99 range for the CCA multimodal reference
- The mRNA-only PCA baseline yields lower correlations (0.65-0.97), confirming the multimodal CCA embedding is what drives accuracy
- Output CSV contains per-protein Pearson r values for BOTH the CCA reference and the PCA-only baseline for all 54 query samples
- Query cells localize within the same low-dimensional reference embedding as reference cells, enabling annotation transfer from reference coordinates

## Limitations

- The CCA strategy used for CITE-seq multimodal integration may not be optimal for all applications since multimodal single-cell integration is an active research area
- Condition I (comprehensive cell type coverage in the reference) is required and limits flexibility relative to de novo Harmony integration
- Mapping metrics may be less informative when a novel query population closely resembles an existing reference population

## Evidence

- [abstract] Symphony was demonstrated on inferring surface protein expression with a multimodal CITE-seq atlas of memory T cells: [evidence span withheld — anchor status: not_found]
- [discussion] Two-step annotation transfer via knnPredict: "We first learn a predictive model in the reference embedding, and then map query cells and use their reference coordinates to predict query annotations."
- [methods] knnPredict call in methods: "query = knnPredict(query,       # query object"
- [abstract] Symphony localizes query cells in a stable embedding for annotation transfer: "Symphony localizes query cells within a stable low-dimensional reference embedding, facilitating reproducible downstream transfer of reference-defined annotations to the query"
- [discussion] Symphony does not replace de novo analyses for novel cell types: "Symphony does not entirely supplant the need for users interested in novel cell type discovery to conduct de novo analyses of the query alone."
