---
name: macro-f1-and-accuracy-evaluation
description: Use when comparing cell-type label-transfer or integration methods on
  scRNA-seq data and you need quantitative annotation metrics (macro F1, accuracy)
  plus integration metrics (silhouette, batch entropy) with statistical significance
  testing.
when_to_use_negative:
- The query data contains unseen cell types absent from the reference — accuracy/F1
  against reference-only labels will be misleading; combine MASI with SCCAF to identify
  subtypes instead
- The analysis goal is single-method quality control of one annotation rather than
  head-to-head method benchmarking across datasets — silhouette/entropy aggregation
  and paired t-tests are designed for method comparison
- Reference data quality or resolution is not matched to the query — performance scores
  will reflect reference mismatch rather than method differences
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: MASI
  role: Method under evaluation; produces gene2cell scores and masi.parallel batch
    annotations to be scored with F1/accuracy
  repo: https://github.com/hayatlab/MASI
- name: scNym
  role: Baseline label-transfer method run with default pipelines before metric computation
  repo: https://github.com/calico/scnym
- name: scArches scANVI
  role: Baseline label-transfer method run with the scanvi surgery pipeline before
    metric computation
  repo: https://scarches.readthedocs.io/en/latest/scanvi_surgery_pipeline.html
- name: Symphony
  role: Baseline reference-mapping method (variable gene selection, scaling, PCA,
    Harmony correction) whose query labels are scored
  repo: https://github.com/immunogenomics/symphony
- name: Scanpy
  role: Provides preprocessing (pp.highly_variable_genes) and the AnnData data structures
    on which silhouette and entropy are computed
- name: MACA
  role: Over-clustering engine behind MASI label transfer whose ensemble labels are
    the predictions being evaluated
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-023-04820-3
    title: MASI enables fast model-free standardization and integration of single-cell
      transcriptomics data
  source_content_hash: sha256:f05f801023076f62466921c635a89e156121d5bb44ae444e5a0a5d770c76eb85
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hayatlab/MASI
  tools:
  - Harmony
  - LIGER
  - MACA
  - MASI
  - Scanorama
  - Scanpy
  - Seurat
  - Symphony
  - scArches
  - scArches scANVI
  - scNym
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# macro-f1-and-accuracy-evaluation

## Summary

Benchmark evaluation of cell-type annotation and integration methods using macro F1, overall accuracy, cell-type silhouette, and batch entropy mixing, aggregated into annotation, integration, and overall performance scores and compared with paired t-tests. This is the metric framework used to show MASI outperforms SVM, scNym, scArches scANVI, and Symphony across six mixed-batch scRNA-seq datasets.

## When to use

Use when comparing cell-type label-transfer or integration methods on scRNA-seq data and you need quantitative annotation metrics (macro F1, accuracy) plus integration metrics (silhouette, batch entropy) with statistical significance testing.

## When NOT to use

- The query data contains unseen cell types absent from the reference — accuracy/F1 against reference-only labels will be misleading; combine MASI with SCCAF to identify subtypes instead
- The analysis goal is single-method quality control of one annotation rather than head-to-head method benchmarking across datasets — silhouette/entropy aggregation and paired t-tests are designed for method comparison
- Reference data quality or resolution is not matched to the query — performance scores will reflect reference mismatch rather than method differences

## Inputs

- Predicted cell-type labels per cell from each annotated method (e.g., MASI gene2cell + masi.parallel output, SVM predictions, scNym/scANVI/Symphony transfers)
- Ground-truth cell-type annotations for the six processed benchmark scRNA-seq datasets (figshare deposit 18866264)
- Batch labels for each cell (needed for batch entropy mixing computation)
- Integrated embeddings/expressions per method (needed for cell-type silhouette)

## Outputs

- Per-method, per-dataset metric table with macro F1, overall accuracy, cell-type silhouette, and batch entropy
- Aggregated annotation, integration, and overall performance scores per method
- Paired t-test p-values comparing the focal method against each baseline (written to benchmark_results.csv)

## How to apply

For each method (e.g., MASI gene2cell/parallel with MACA resolutions 3, 5, 7 and neighborhood sizes 5, 10, 15; SVM baselines trained on the same cell-type markers; scNym and scArches scANVI with default pipelines; Symphony with Harmony-corrected reference; Scanorama, LIGER, Seurat), predict cell-type labels or integrated embeddings on the six benchmark datasets from the figshare deposit 18866264. Compute macro F1 and overall accuracy from predicted vs. true cell-type labels, cell-type silhouette to measure annotation coherence, and batch entropy to quantify batch mixing, then aggregate the four metrics into annotation, integration, and overall performance scores. Finally, run paired t-tests comparing the focal method against each baseline across the six datasets (significance threshold p < 0.05) and write all metrics and p-values to benchmark_results.csv.

## Related tools

- **MASI** (Method under evaluation; produces gene2cell scores and masi.parallel batch annotations to be scored with F1/accuracy) — https://github.com/hayatlab/MASI
- **scNym** (Baseline label-transfer method run with default pipelines before metric computation) — https://github.com/calico/scnym
- **scArches scANVI** (Baseline label-transfer method run with the scanvi surgery pipeline before metric computation) — https://scarches.readthedocs.io/en/latest/scanvi_surgery_pipeline.html
- **Symphony** (Baseline reference-mapping method (variable gene selection, scaling, PCA, Harmony correction) whose query labels are scored) — https://github.com/immunogenomics/symphony
- **Scanpy** (Provides preprocessing (pp.highly_variable_genes) and the AnnData data structures on which silhouette and entropy are computed)
- **MACA** (Over-clustering engine behind MASI label transfer whose ensemble labels are the predictions being evaluated)

## Examples

```
from scnym.api import scnym_api; scnym_api(adata=adata, task='train', groupby='cell_ontology_class', out_path='./scnym_output', config='no_new_identity')  # then compute macro F1/accuracy vs true labels
```

## Evaluation signals

- benchmark_results.csv contains a row per (method, dataset) with all four metrics (macro F1, accuracy, silhouette, batch entropy) populated and no missing values
- Macro F1 and overall accuracy are computed against the same ground-truth cell-type labels across all methods, ensuring a fair paired comparison
- Paired t-tests are run across the six datasets per baseline, and significant results are reported at p < 0.05
- Aggregated annotation, integration, and overall scores are consistent with (derivable from) the four raw metrics per method
- The focal method (MASI) should rank among the top methods for both integration and annotation, reproducing the published finding

## Limitations

- Benchmark conclusions hold only for the datasets used in the study; the specific benchmark datasets are not enumerated in the benchmarking section
- scNym and scArches were benchmarked only with default pipelines; optimal performance may require customized parameter tuning, so baseline scores may underestimate those methods
- No software version numbers are reported for any benchmarked tool, complicating exact reproduction
- Cell-type silhouette and batch entropy depend on preprocessing choices (e.g., LogNormalization with scale factor 10,000 and log-transformation) that must be kept identical across methods

## Evidence

- [results] MASI achieved significantly higher overall performance than all other tested methods (paired t-tests, p-values < 0.05) except Seurat, and was ranked among the top methods for both integration and annotation.: [evidence span withheld — anchor status: not_found]
- [methods] Compute macro F1, overall accuracy, cell-type silhouette, and batch entropy for each method, and aggregate them into annotation, integration, and overall performance scores.: [evidence span withheld — anchor status: not_found]
- [abstract] We benchmark MASI with other well-established methods and demonstrate that MASI outperforms other methods, in terms of integration, annotation, and speed.: "We benchmark MASI with other well-established methods and demonstrate that MASI outperforms other methods, in terms of integration, annotation, and speed."
- [discussion] for benchmarking, we used default pipelines of both methods for all data involved in this study.: "an optimal outcome for a specific data might require customized parameter tuning. However, for benchmarking, we used default pipelines of both methods"
- [methods] Processed data can be found at https://figshare.com/articles/dataset/Fast_model-free_integration_and_transfer_learning_via_MASI_for_single-cell_expression_data/18866264.: "Processed data can be found at https://figshare.com/articles/dataset/Fast_model-free_integration_and_transfer_learning_via_MASI_for_single-cell_expression_data/18866264."
