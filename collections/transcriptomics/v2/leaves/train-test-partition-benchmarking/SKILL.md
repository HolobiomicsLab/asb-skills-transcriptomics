---
name: train-test-partition-benchmarking
description: Use when comparing a reference-mapping annotation method against supervised
  cell type classifiers across cross-technology or held-out-donor single-cell RNA-seq
  splits, e.g. replicating Abdelaal et al.'s PbmcBench benchmark with Symphony references.
when_to_use_negative:
- You need novel cell type discovery in the query — Symphony does not replace de novo
  analyses and identification of unmapped novel populations is an unsolved problem
- The query contains a cell population absent from the reference — Condition I requires
  comprehensive cell type coverage in the reference for annotation transfer to be
  equivalent to de novo Harmony integration
- The task is de novo integration of reference and query without a fixed reference
  atlas — use Harmony/BBKNN/Seurat integration instead of reference mapping
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: Symphony
  role: Builds the compressed integrated reference per split and maps held-out query
    cells into the harmonized embedding via buildReference and mapQuery
  repo: https://github.com/immunogenomics/symphony
- name: Harmony
  role: Performs the donor-integrated dimension reduction (K=100 clusters, 20 PCs)
    underlying each Symphony reference
  repo: https://github.com/immunogenomics/symphony
- name: singlecellmethods
  role: Provides utility functions (e.g. vst variable gene selection, evaluation helpers)
    used in the reference-building and scoring workflow
  repo: https://github.com/immunogenomics/singlecellmethods
- name: scRNAseq_Benchmark
  role: Source of the Abdelaal et al. PbmcBench benchmark dataset and the comparator
    supervised classifiers (scmap, SingleCellNet, SCINA) F1 ranges
  repo: https://github.com/tabdelaal/scRNAseq_Benchmark
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-25957-x
    title: Efficient and precise single-cell reference atlas mapping with Symphony
  source_content_hash: sha256:b608da905cfe335d7f0f5916babf3fb28acb2c12c14878376c547431e456c514
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/immunogenomics/symphony
  tools:
  - Harmony
  - Symphony
  - scRNAseq_Benchmark
  - singlecellmethods
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# train-test-partition-benchmarking

## Summary

Benchmark Symphony-based cell type classification against supervised classifiers (scmap-cell, SingleCellNet, SCINA) by constructing 48 held-out protocol/donor train-test partitions on the PbmcBench PBMC dataset and scoring per-cell-type F1.

## When to use

Use when comparing a reference-mapping annotation method against supervised cell type classifiers across cross-technology or held-out-donor single-cell RNA-seq splits, e.g. replicating Abdelaal et al.'s PbmcBench benchmark with Symphony references.

## When NOT to use

- You need novel cell type discovery in the query — Symphony does not replace de novo analyses and identification of unmapped novel populations is an unsolved problem
- The query contains a cell population absent from the reference — Condition I requires comprehensive cell type coverage in the reference for annotation transfer to be equivalent to de novo Harmony integration
- The task is de novo integration of reference and query without a fixed reference atlas — use Harmony/BBKNN/Seurat integration instead of reference mapping

## Inputs

- PbmcBench PBMC scRNA-seq expression matrix (multiple technologies/protocols)
- PbmcBench cell metadata with donor, protocol, and cell type annotations
- 48 protocol/donor train-test partition definitions (each held-out protocol or donor serves once as query)

## Outputs

- Symphony reference RDS and uwot model per train-test split
- Per-experiment median cell-type F1 table (f1_benchmark_results.csv)
- Average median F1 per classifier (Symphony 5-NN, SVM radial kernel, multinomial logistic regression) compared to scmap-cell, SingleCellNet, SCINA ranges

## How to apply

Load the PbmcBench PBMC expression matrix and cell metadata and construct 48 train-test partitions in which each held-out protocol or donor serves once as the query. For each training split, build a Symphony reference with buildReference using vst variable gene selection (top 1,000 genes per group), 20 PCs, K=100 Harmony clusters, and integration over the donor variable, saving the reference RDS and uwot model per split. Normalize each held-out query to log(CP10k+1) and map it into the harmonized reference embedding with mapQuery. Predict query cell types with three downstream classifiers trained on the reference embedding: 5-nearest-neighbor voting via knnPredict (k=5, with confidence scores), a linear SVM, and a multinomial (generalized linear) logistic regression. Score every experiment with per-cell-type F1 and take the median across cell types using the evaluate utility, then average the median F1 over the 48 experiments and compare against the top automated classifiers' reported range (0.77-0.83) from Abdelaal et al., writing the per-experiment F1 table to f1_benchmark_results.csv.

## Related tools

- **Symphony** (Builds the compressed integrated reference per split and maps held-out query cells into the harmonized embedding via buildReference and mapQuery) — https://github.com/immunogenomics/symphony
- **Harmony** (Performs the donor-integrated dimension reduction (K=100 clusters, 20 PCs) underlying each Symphony reference) — https://github.com/immunogenomics/symphony
- **singlecellmethods** (Provides utility functions (e.g. vst variable gene selection, evaluation helpers) used in the reference-building and scoring workflow) — https://github.com/immunogenomics/singlecellmethods
- **scRNAseq_Benchmark** (Source of the Abdelaal et al. PbmcBench benchmark dataset and the comparator supervised classifiers (scmap, SingleCellNet, SCINA) F1 ranges) — https://github.com/tabdelaal/scRNAseq_Benchmark

## Examples

```
reference = symphony::buildReference(exprs_scaled, var_genes, metadata, vars = 'donor', K = 100, verbose = TRUE, save_uwot_path = 'ref_uwot')
```

## Evaluation signals

- Average median cell type F1 across all 48 experiments falls in the reported 0.79-0.87 range for the three Symphony-based classifiers
- Symphony classifier range is comparable to the top three supervised classifiers' reported 0.77-0.83 average median F1 (Fig. 3a)
- Per-experiment F1 table (f1_benchmark_results.csv) contains one median-F1 row per train-test split per classifier
- Each per-split Symphony reference harmonizes donors (integration variable = donor) with 20 PCs and K=100 Harmony clusters, and the saved uwot model reproduces the embedding on remap
- knnPredict returns confidence scores alongside 5-NN (k=5) predicted labels for each query cell

## Limitations

- Identification of novel cell types that failed to map is an unsolved future direction; mapping metrics may be less informative when the novel population closely resembles an existing reference population
- Condition I (comprehensive cell type coverage in the reference) is inflexible and limits equivalence with de novo Harmony integration
- Transfer of annotations from reference to query remains an open area of research

## Evidence

- [abstract] Symphony localizes query cells within a stable low-dimensional reference embedding, facilitating transfer of reference-defined annotations to the query: "Symphony localizes query cells within a stable low-dimensional reference embedding, facilitating reproducible downstream transfer of reference-defined annotations"
- [discussion] Two-step annotation transfer: "We first learn a predictive model in the reference embedding, and then map query cells and use their reference coordinates to predict query annotations."
- [methods] vst variable gene selection: "var_genes = vargenes_vst(ref_exp_full, groups = as.character(ref_metadata[['donor']]), topn = 1000)"
- [methods] buildReference and mapQuery: "reference = symphony::buildReference("
- [methods] knnPredict classification: "query = knnPredict(query,       # query object"
- [discussion] Novel cell types unmapped: "Identification of novel cell-types that have failed to map is an important future direction for mapping algorithms."
