---
name: annotation-throughput-measurement
description: Use when benchmarking cell-type annotation methods and you need quantitative
  runtime/throughput (cells/min) on held-out datasets with known cell counts, alongside
  predictive metrics like F1 and balanced accuracy.
when_to_use_negative:
- The goal is annotation accuracy evaluation without any runtime component — timing
  adds nothing to F1/balanced accuracy comparisons
- Input is not a Seurat v4 object with a UMAP reduction; Census requires Seurat version
  4 and a UMAP embedding before timing can begin
- The annotator requires a custom-trained model on a new reference; training time
  must be excluded or reported separately from per-cell annotation throughput
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Census
  role: Hierarchical gradient-boosted decision tree cell-type annotator whose runtime
    is measured; achieved ~13,000 cells/min on the 44 held-out datasets
  repo: https://github.com/sjdlabgroup/Census
- name: Seurat
  role: Required R framework (version 4) in which Census runs and which provides the
    processed Seurat object with UMAP used as the timed annotation input
  repo: https://zenodo.org/records/7017103
- name: ggpubr
  role: R package used to run and visualize paired statistical comparisons (e.g.,
    Wilcoxon signed-rank) of annotation metrics, including runtime, across the 44
    datasets
  repo: https://github.com/kassambara/ggpubr
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad714
    title: Hierarchical and automated cell-type annotation and inference of cancer
      cell of origin with Census
  source_content_hash: sha256:4eaa8de4365bcf50334e3636b277323f2d4433a78681160fb73d3389c0e84579
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sjdlabgroup/Census
  tools:
  - Census
  - Seurat
  - ggpubr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# annotation-throughput-measurement

## Summary

Measures cell-type annotation runtime and throughput (cells per minute) on held-out scRNA-seq data, as done when benchmarking Census against scType on 44 atlas-scale human and mouse normal/cancer tissue datasets.

## When to use

Use when benchmarking cell-type annotation methods and you need quantitative runtime/throughput (cells/min) on held-out datasets with known cell counts, alongside predictive metrics like F1 and balanced accuracy.

## When NOT to use

- The goal is annotation accuracy evaluation without any runtime component — timing adds nothing to F1/balanced accuracy comparisons
- Input is not a Seurat v4 object with a UMAP reduction; Census requires Seurat version 4 and a UMAP embedding before timing can begin
- The annotator requires a custom-trained model on a new reference; training time must be excluded or reported separately from per-cell annotation throughput

## Inputs

- Seurat v4 object with processed UMAP dimensionality reduction (held-out scRNA-seq dataset)
- Known cell counts per held-out dataset (e.g., 1,769,071 cells across 44 datasets)
- Annotation outputs from each method being compared (Census census_main predictions, scType predictions)

## Outputs

- Per-dataset annotation runtime and throughput in cells/min (e.g., ~13,000 cells/min for Census)
- benchmark_metrics.csv with runtime alongside F1, balanced accuracy, label similarity, and overall accuracy per dataset
- Paired runtime comparison across the 44 datasets (e.g., Wilcoxon signed-rank via ggpubr)

## How to apply

Load each held-out dataset (e.g., the 44 human and mouse normal/cancer tissue datasets totaling 1,769,071 cells) into R with Seurat v4 and process it through the standard Seurat pipeline (NormalizeData, ScaleData, FindVariableFeatures, RunPCA, RunUMAP dims 1:50) so the annotator's input is fixed. Wrap the annotation call — census_main for Census, or the scType annotation function — in a timing harness (e.g., system.time) applied to the held-out cells only, excluding preprocessing time. Record elapsed time per dataset and compute throughput as cells divided by elapsed minutes; in the Census benchmark this yielded ~13,000 cells/min. Compile per-dataset runtimes for all methods alongside F1, balanced accuracy, label similarity, and overall accuracy into a metrics table (benchmark_metrics.csv), and use ggpubr to run paired statistical comparisons across datasets. In the reported benchmark, scType had shorter run times while Census achieved significantly higher F1, balanced accuracy, label similarity, and overall accuracy (Wilcoxon P < 2e-16), so throughput must be interpreted as a trade-off axis rather than a sole quality signal.

## Related tools

- **Census** (Hierarchical gradient-boosted decision tree cell-type annotator whose runtime is measured; achieved ~13,000 cells/min on the 44 held-out datasets) — https://github.com/sjdlabgroup/Census
- **Seurat** (Required R framework (version 4) in which Census runs and which provides the processed Seurat object with UMAP used as the timed annotation input) — https://zenodo.org/records/7017103
- **ggpubr** (R package used to run and visualize paired statistical comparisons (e.g., Wilcoxon signed-rank) of annotation metrics, including runtime, across the 44 datasets) — https://github.com/kassambara/ggpubr

## Examples

```
res = census_main(obj, organ = 'Pancreas', predict_cancer = T)
```

## Evaluation signals

- Throughput lands in the reported range (~13,000 cells/min for Census on atlas-scale held-out data)
- Timing excludes Seurat preprocessing; the timed section covers only the annotation call on held-out cells
- Runtime is recorded per dataset for all 44 datasets and written to benchmark_metrics.csv alongside accuracy metrics
- Directional check: scType exhibits shorter run times than Census while Census dominates on F1, balanced accuracy, label similarity, and overall accuracy (Wilcoxon P < 2e-16)
- Subtype tally from the same run reproduces Census 81/105 vs scType 35/89 correctly identified cell subtypes

## Limitations

- scType had shorter run times than Census, so Census's throughput advantage claims rest on accuracy metrics, not speed
- Timing depends on the fixed Seurat v4 pipeline (NormalizeData, ScaleData, FindVariableFeatures, RunPCA, RunUMAP dims 1:50) and hardware; throughput numbers are not directly transferable across preprocessing choices
- Census requires Seurat version 4 specifically and an input Seurat object containing a UMAP dimensionality reduction; other versions or missing UMAP invalidate the timed run
- The pretrained census_main model is trained on Tabula Sapiens and CCLE (175 cell-types, 24 organs); runtime for custom models may differ from the reported throughput

## Evidence

- [abstract] Speed framing from abstract: [evidence span withheld — anchor status: not_found]
- [abstract] Benchmark scale from abstract: "When benchmarked on 44 atlas-scale normal and cancer, human and mouse tissues, Census significantly outperforms state-of-the-art methods across multiple metrics"
- [readme] Seurat v4 requirement from README: "Note: a demo script (demo.r) and dataset (peng_sub.RDS) are also included in this directory. Also note: Census requires Seurat version 4 to run."
- [readme] Standard Seurat preprocessing that precedes the timed annotation: "obj = obj %>% CreateSeuratObject() %>% NormalizeData() %>% ScaleData() %>% FindVariableFeatures() %>% RunPCA() %>% RunUMAP(dims=1:50)"
