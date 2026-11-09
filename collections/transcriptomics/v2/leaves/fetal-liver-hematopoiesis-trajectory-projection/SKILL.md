---
name: fetal-liver-hematopoiesis-trajectory-projection
description: Use when a query dataset (e.g., a 10x 5' fetal liver sample) must be
  positioned along a continuous reference-defined differentiation gradient — such
  as fetal liver hematopoiesis — instead of, or in addition to, per-cluster annotation
  transfer.
when_to_use_negative:
- The goal is discovery of novel cell types absent from the reference — Symphony does
  not replace de novo analyses of the query alone.
- The query population is very similar to an existing reference population, where
  mapping metrics may be less informative about novel states.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Symphony
  role: Builds the compressed fetal liver reference and maps query cells into the
    harmonized embedding via mapQuery() and knnPredict()
  repo: https://github.com/immunogenomics/symphony
- name: Harmony
  role: Performs the underlying integration of the reference from which the Symphony
    reference object is compressed
  repo: https://github.com/immunogenomics/singlecellmethods
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-25957-x
    title: Efficient and precise single-cell reference atlas mapping with Symphony
  source_content_hash: sha256:7dc4a065ba49b0a9ebe7c5ad026040f5b7f859e52f2f2a73e9416e7fdd6c9d9e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/immunogenomics/symphony
  tools:
  - Harmony
  - Symphony
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# fetal-liver-hematopoiesis-trajectory-projection

## Summary

Projects 10x 5' query cells into a Symphony fetal liver reference embedding and localizes them along the reference-defined FDG hematopoiesis developmental trajectory, verifying smooth transitions in canonical markers (PPBP, HBB, KIT) rather than only transferring discrete cell type labels.

## When to use

Use when a query dataset (e.g., a 10x 5' fetal liver sample) must be positioned along a continuous reference-defined differentiation gradient — such as fetal liver hematopoiesis — instead of, or in addition to, per-cluster annotation transfer.

## When NOT to use

- The goal is discovery of novel cell types absent from the reference — Symphony does not replace de novo analyses of the query alone.
- The query population is very similar to an existing reference population, where mapping metrics may be less informative about novel states.

## Inputs

- Saved Symphony fetal liver reference object
- 10x 5' query expression matrix normalized as log(CP10k+1)
- Query cell metadata with reference-derived true labels for validation

## Outputs

- query_mapping_results.csv containing per-cell predictions, knnPredict confidence scores, and reference UMAP coordinates
- Per-cell F1 and overall accuracy metrics versus reference-derived true labels (target: median F1 0.83, accuracy 85.0%)
- MEM-lineage query cells positioned on the FDG trajectory with marker gene (PPBP, HBB, KIT) gradient plots

## How to apply

Load the saved Symphony fetal liver reference and the 10x 5' query expression matrix, normalizing the query as log(CP10k+1) to match the reference convention. Run Symphony's mapQuery() to project query cells into the harmonized reference embedding and UMAP coordinates (do_umap = TRUE), then annotate with knnPredict() using k = 30 neighbors and confidence = TRUE. To localize cells on the FDG developmental trajectory, average the coordinates of each MEM-lineage query cell's 10 nearest reference neighbors. Validate by scoring annotation transfer against reference-derived true labels (expected median cell type F1 of 0.83 and accuracy of 85.0% at k = 30) and by confirming monotonic PPBP (megakaryocyte), HBB (erythrocyte), and KIT (mast cell) gradients along the trajectory branch. Write predictions, confidences, and UMAP coordinates to query_mapping_results.csv.

## Related tools

- **Symphony** (Builds the compressed fetal liver reference and maps query cells into the harmonized embedding via mapQuery() and knnPredict()) — https://github.com/immunogenomics/symphony
- **Harmony** (Performs the underlying integration of the reference from which the Symphony reference object is compressed) — https://github.com/immunogenomics/singlecellmethods

## Examples

```
query = mapQuery(query_exp, query_metadata, reference, do_umap = TRUE); query = knnPredict(query, reference, ref_metadata$cell_type, k = 30, confidence = TRUE)
```

## Evaluation signals

- Median cell type F1 from knnPredict (k = 30, confidence = TRUE) reaches 0.83 against reference-derived true labels
- Overall annotation accuracy reaches 85.0% at k = 30
- MEM-lineage query cells recapitulate the FDG trajectory branch when placed via their 10 nearest reference neighbors
- Marker genes show monotonic gradients along the trajectory: PPBP toward megakaryocytes, HBB toward erythrocytes, KIT toward mast cells
- Query normalization matches the reference convention (log(CP10k+1)) before mapQuery()

## Limitations

- Identification of novel cell types that have failed to map is an unsolved future direction for mapping algorithms
- Mapping metrics may be less informative when a novel population is very similar to an existing reference population
- Symphony does not replace de novo analyses of the query alone for users interested in novel cell type discovery
- Comprehensive cell type coverage in the reference (Condition I) is required and less flexible than de novo Harmony integration

## Evidence

- [abstract] Symphony localizes query cells within a stable low-dimensional reference embedding, facilitating reproducible downstream transfer of reference-defined annotations to the query: "Symphony localizes query cells within a stable low-dimensional reference embedding, facilitating reproducible downstream transfer of reference-defined annotations to the query"
- [abstract] Localizing query cells along a developmental trajectory: "localizing query cells along a developmental trajectory of fetal liver hematopoiesis"
- [methods] mapQuery projects query cells into the reference embedding: "query = mapQuery(query_exp,"
- [methods] knnPredict transfers reference cell type labels to query cells: "query = knnPredict(query,       # query object"
- [readme] Symphony installation: "Symphony is available on CRAN:

 r
install.packages("symphony")"
