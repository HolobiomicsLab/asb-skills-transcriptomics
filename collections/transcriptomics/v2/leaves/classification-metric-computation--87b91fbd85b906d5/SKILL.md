---
name: classification-metric-computation
description: Use when you have predicted cell-type labels from one or more annotators
  on the Peng 2019 pancreatic cancer dataset and author-provided ground-truth annotations,
  and need per-method F1, balanced accuracy, accuracy, label similarity, and runtime
  in a comparison table.
when_to_use_negative:
- The dataset has no author-provided or expert ground-truth annotations, so F1 and
  balanced accuracy against a reference cannot be computed
- You only want Census predictions on a new sample and do not need a multi-method
  benchmark (directly run census_main instead)
- The task is not cell-level categorical classification evaluation (e.g., clustering
  quality or differential expression), where F1/balanced accuracy against fixed labels
  is not the right metric family
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Census
  role: Hierarchical gradient-boosted cell-type annotator under evaluation; its predictions
    (and cancer cell-of-origin calls) are scored against author annotations via the
    metric suite
  repo: https://github.com/sjdlabgroup/Census
- name: Seurat 4
  role: Required dependency for running Census and for the preprocessing pipeline
    (NormalizeData through RunUMAP) that yields the Seurat object fed to all annotators
  repo: https://github.com/sjdlabgroup/Census
- name: ggpubr
  role: Used for statistical annotation of benchmark comparisons (e.g., Wilcoxon significance
    brackets via stat_compare_means/geom_pwc) on the assembled metric table
  repo: https://github.com/kassambara/ggpubr
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad714
    title: Hierarchical and automated cell-type annotation and inference of cancer
      cell of origin with Census
  source_content_hash: sha256:0a1558529f8b436d3c42ea57151eab413b7de5a3a5fd00ee23ff0be9057c63ce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sjdlabgroup/Census
  tools:
  - Census
  - Seurat 4
  - ggpubr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# classification-metric-computation

## Summary

Benchmarking protocol that scores predicted cell-type annotations against author-provided reference labels on the Peng et al. 2019 pancreatic cancer scRNA-seq dataset (57,530 cells), computing F1, balanced accuracy, overall accuracy, label similarity, and runtime for Census versus scType, scATOMIC, scibet, and scCATCH, with Wilcoxon significance testing (P < 0.05).

## When to use

Use when you have predicted cell-type labels from one or more annotators on the Peng 2019 pancreatic cancer dataset and author-provided ground-truth annotations, and need per-method F1, balanced accuracy, accuracy, label similarity, and runtime in a comparison table.

## When NOT to use

- The dataset has no author-provided or expert ground-truth annotations, so F1 and balanced accuracy against a reference cannot be computed
- You only want Census predictions on a new sample and do not need a multi-method benchmark (directly run census_main instead)
- The task is not cell-level categorical classification evaluation (e.g., clustering quality or differential expression), where F1/balanced accuracy against fixed labels is not the right metric family

## Inputs

- Seurat object of the Peng et al. 2019 pancreatic cancer scRNA-seq dataset (57,530 cells) with a UMAP dimensionality reduction
- Author-provided cell-type annotations for the Peng 2019 dataset (ground truth)
- Predicted cell-type label tables from Census, scType, scATOMIC, scibet, and scCATCH

## Outputs

- benchmark_table.csv: per-method table reporting F1, balanced accuracy, label similarity, overall accuracy, and runtime for Census and all four baselines
- Wilcoxon test results establishing whether Census significantly outperforms each baseline (P < 0.05)

## How to apply

First run each annotator on the same processed Seurat object: Census via census_main(obj, organ='Pancreas', predict_cancer=T) after the standard Seurat pipeline (NormalizeData, ScaleData, FindVariableFeatures, RunPCA, RunUMAP dims=1:50), scibet with the '30_major_human_cell_types' pretrained model, and scType configured with pancreas tissue plus immune cell reference selection. Compare each method's predicted labels cell-by-cell against the author-provided cell-type annotations from Peng et al. 2019, computing F1, balanced accuracy, label similarity, overall accuracy, and per-method runtime. Assemble the five metrics for all five methods (Census + four baselines) into a per-method comparison table and save it as benchmark_table.csv. Assess significance of score differences between Census and each baseline with Wilcoxon tests (P < 0.05), which can be annotated with ggpubr's stat_compare_means/geom_pwc helpers. Validation: Census should rank first with a higher mean F1 and balanced accuracy than the second-place method and significantly higher scores than the remaining methods, matching the reported benchmark.

## Related tools

- **Census** (Hierarchical gradient-boosted cell-type annotator under evaluation; its predictions (and cancer cell-of-origin calls) are scored against author annotations via the metric suite) — https://github.com/sjdlabgroup/Census
- **Seurat 4** (Required dependency for running Census and for the preprocessing pipeline (NormalizeData through RunUMAP) that yields the Seurat object fed to all annotators) — https://github.com/sjdlabgroup/Census
- **ggpubr** (Used for statistical annotation of benchmark comparisons (e.g., Wilcoxon significance brackets via stat_compare_means/geom_pwc) on the assembled metric table) — https://github.com/kassambara/ggpubr

## Examples

```
res = census_main(obj, organ = 'Pancreas', predict_cancer = T)
```

## Evaluation signals

- benchmark_table.csv contains all five metrics (F1, balanced accuracy, label similarity, accuracy, runtime) for Census plus all four baselines
- Census attains the highest mean F1 score and balanced accuracy among the five methods on the Peng 2019 dataset
- Wilcoxon tests report P < 0.05 for Census versus the non-top-performing baselines
- Census metrics are consistent with the reported benchmark in which Census outperforms state-of-the-art methods across multiple metrics
- Runtime is recorded per method so speed comparisons (Census's 'high prediction speed') can be checked against quality metrics

## Limitations

- Census requires Seurat v4 loaded prior to loading the Census library in R; version mismatches will break the run
- F1 and balanced accuracy are only as reliable as the author-provided Peng 2019 annotations used as ground truth
- Label-similarity scoring depends on reconciling each method's label vocabulary (e.g., scibet's 30 major human cell types vs. Census's 175 Tabula Sapiens cell-types) with the reference labels

## Evidence

- [readme] Seurat preprocessing and Census invocation on Peng data: "This example uses Census to predict cell-types in a pancreatic cancer dataset data from Peng et al., Nature Cell Research 2019."
- [readme] Seurat v4 requirement: "Note: a demo script (demo.r) and dataset (peng_sub.RDS) are also included in this directory."
- [abstract] Benchmark scope statement: "When benchmarked on 44 atlas-scale normal and cancer, human and mouse tissues, Census significantly outperforms state-of-the-art methods across multiple metrics and naturally predicts the"
- [intro] Metric targets of the trained models: "The main Census model is trained on the Tabula Sapiens and Cancer Cell Line Encyclopedia and can identify 175 cell-types from 24 organs"
