---
name: classification-metric-evaluation-f1-accuracy
description: Use when a Symphony mapQuery/knnPredict annotation transfer must be quantitatively
  validated against known reference labels, e.g. reproducing the reported median F1
  of 0.83 and accuracy of 85.0% at k = 30.
when_to_use_negative:
- The query population is a novel cell type absent from the reference — Symphony mapping
  metrics may be less informative when the novel population is very similar to an
  existing reference population, and annotation transfer is inappropriate for novel
  cell type discovery
- The query expression is not normalized under the same convention as the reference
  (log(CP10k+1)) — label transfer will be confounded by preprocessing mismatch rather
  than reflecting true annotation quality
- De novo cell state definition is the goal rather than transfer of reference-defined
  annotations
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Symphony
  role: Provides mapQuery() for projecting query cells into the reference embedding
    and knnPredict() for k-nearest-neighbor label transfer (k = 30, confidence = TRUE)
    whose F1/accuracy are evaluated by this skill
  repo: https://github.com/immunogenomics/symphony
- name: Harmony
  role: Underlying integration method whose harmonized embedding Symphony compresses;
    the stable embedding is the coordinate system in which predictions are scored
  repo: https://github.com/immunogenomics/singlecellmethods
- name: scRNAseq_Benchmark
  role: Benchmark of automated cell identification methods for scRNA-seq, contextualizing
    F1/accuracy-based evaluation of cell type classifiers
  repo: https://github.com/tabdelaal/scRNAseq_Benchmark
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-25957-x
    title: Efficient and precise single-cell reference atlas mapping with Symphony
  source_content_hash: sha256:a3491fa02783428ff5baf932ca01fe052a6a163d0e79bb3e8a4073ae7a6db63f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/immunogenomics/symphony
  tools:
  - Harmony
  - Symphony
  - scRNAseq_Benchmark
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# classification-metric-evaluation-f1-accuracy

## Summary

Evaluates Symphony's reference-to-query annotation transfer by comparing knnPredict cell type labels against reference-derived true labels, quantifying per-cell-type F1 and overall accuracy. Validates that mapped query cells (e.g., a 10x 5' PBMC or 10x 5' fetal liver query) are annotated correctly after projection into the harmonized reference embedding.

## When to use

Use when a Symphony mapQuery/knnPredict annotation transfer must be quantitatively validated against known reference labels, e.g. reproducing the reported median F1 of 0.83 and accuracy of 85.0% at k = 30.

## When NOT to use

- The query population is a novel cell type absent from the reference — Symphony mapping metrics may be less informative when the novel population is very similar to an existing reference population, and annotation transfer is inappropriate for novel cell type discovery
- The query expression is not normalized under the same convention as the reference (log(CP10k+1)) — label transfer will be confounded by preprocessing mismatch rather than reflecting true annotation quality
- De novo cell state definition is the goal rather than transfer of reference-defined annotations

## Inputs

- Symphony query object returned by mapQuery() (query cells projected into the harmonized reference embedding)
- Reference cell type labels from the Symphony fetal liver (or PBMC) reference object
- Reference-derived true labels for query cells (ground truth for scoring)
- Per-cell confidence scores from knnPredict (confidence = TRUE)

## Outputs

- Per-cell predicted cell type labels and confidence scores
- Per-cell-type F1 scores (target median F1 = 0.83 at k = 30)
- Overall annotation accuracy (target 85.0% at k = 30)
- query_mapping_results.csv containing predictions, confidences, and UMAP coordinates

## How to apply

After mapping the query with mapQuery() into the Symphony reference embedding (query normalized as log(CP10k+1) to match the reference), run knnPredict() against the reference cell type labels with k = 30 nearest neighbors and confidence = TRUE to obtain per-cell predicted labels and confidence scores. Treat reference-derived labels on the query as ground truth, then compute per-cell-type F1 scores and overall accuracy from the predicted-versus-true label pairs. For the fetal liver hematopoiesis demonstration, the published benchmark is a median cell type F1 of 0.83 and 85.0% accuracy at k = 30; the mapping is judged successful only when these thresholds are met. Additionally verify that mapped MEM-lineage query cells recapitulate the reference FDG developmental trajectory with monotonic marker gradients (PPBP, HBB, KIT) — confidence scores should be reported alongside predictions in the output (query_mapping_results.csv). The rationale is Symphony's two-step design: a predictive model is learned in the reference embedding, and query cells are annotated using their coordinates in that same stable low-dimensional space.

## Related tools

- **Symphony** (Provides mapQuery() for projecting query cells into the reference embedding and knnPredict() for k-nearest-neighbor label transfer (k = 30, confidence = TRUE) whose F1/accuracy are evaluated by this skill) — https://github.com/immunogenomics/symphony
- **Harmony** (Underlying integration method whose harmonized embedding Symphony compresses; the stable embedding is the coordinate system in which predictions are scored) — https://github.com/immunogenomics/singlecellmethods
- **scRNAseq_Benchmark** (Benchmark of automated cell identification methods for scRNA-seq, contextualizing F1/accuracy-based evaluation of cell type classifiers) — https://github.com/tabdelaal/scRNAseq_Benchmark

## Examples

```
query = knnPredict(query, reference, reference$cell_type, k = 30, confidence = TRUE)
```

## Evaluation signals

- Median per-cell-type F1 reaches 0.83 at k = 30 neighbors when knnPredict labels are scored against reference-derived true labels
- Overall annotation accuracy reaches 85.0% at k = 30
- Per-cell confidence scores are non-degenerate (present for every query cell) and correlate with per-cell correctness
- query_mapping_results.csv contains aligned columns for predictions, confidences, and UMAP coordinates for every query cell
- MEM-lineage query cells placed on the FDG trajectory show monotonic PPBP (megakaryocyte), HBB (erythrocyte), and KIT (mast cell) gradients, confirming correct localization underlying the annotations

## Limitations

- Symphony mapping metrics may be less informative when a novel population is very similar to an existing reference population
- Identification of novel cell types that failed to map is unsolved — unmapped novel populations will silently distort F1/accuracy if scored against the nearest reference label
- Condition I (comprehensive cell type coverage in the reference) is required for equivalence with de novo Harmony integration and limits the flexibility of annotation transfer
- Transfer of annotations from reference to query remains an open area of research

## Evidence

- [abstract] Symphony localizes query cells within a stable low-dimensional reference embedding, facilitating transfer of reference-defined annotations to the query: [evidence span withheld — anchor status: not_found]
- [discussion] Two-step annotation transfer in the reference embedding: "We first learn a predictive model in the reference embedding, and then map query cells and use their reference coordinates to predict query annotations."
- [methods] knnPredict is the label-transfer step being evaluated: "query = knnPredict(query,       # query object"
- [discussion] Annotation transfer remains open research: "Transfer of annotations from reference to query is an open area of research that includes algorithms for automated cell type classification"
- [readme] Symphony installation via CRAN README: "Efficient and precise single-cell reference atlas mapping with Symphony"
