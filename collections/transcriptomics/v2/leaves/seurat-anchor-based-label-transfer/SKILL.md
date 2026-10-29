---
name: seurat-anchor-based-label-transfer
description: Use when benchmarking single-cell cell-type classifiers against Seurat
  v3, or when transferring cell-type labels from an annotated reference scRNA-seq
  dataset to a query dataset with FindTransferAnchors + TransferData.
when_to_use_negative:
- When the goal is the paper's proposed method — SciBet achieves order-of-magnitude
  speed gains and uses absolute similarity to a null dataset, which Seurat v3 label
  transfer does not provide.
- When the input is full-length read count data normalized as TPM + log-normalization
  rather than UMI data, since the default Seurat v3 UMI preprocessing does not apply.
- When the task requires flagging unrepresented cell types; the paper's benchmark
  explicitly used scmapCluster/TransferData 'without considering the unassigned cells'.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Seurat v3
  role: 'Anchor-based label transfer classifier used as a baseline: FindTransferAnchors
    to identify anchors and TransferData to classify query cells'
- name: SciBet
  role: Multinomial-MLE classifier installed via devtools::install_github("PaulingLiu/scibet"),
    used to benchmark speed against Seurat v3 label transfer
  repo: https://github.com/PaulingLiu/scibet
- name: scmap
  role: scmapCluster-based classification baseline run under the same single-CPU timing
    conditions
- name: ggplot2
  role: Plots runtimes versus dataset size on a logarithmic scale to reproduce the
    Fig. 2e timing comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-15523-2
    title: SciBet as a portable and fast single cell type identifier
  source_content_hash: sha256:06121be0872b43ba473ffb5afca17f24fa1dcf31decfe335b827d81f78cf8381
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PaulingLiu/scibet
  tools:
  - SciBet
  - Seurat v3
  - ggplot2
  - scmap
  - tidyverse
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# seurat-anchor-based-label-transfer

## Summary

Seurat v3 anchor-based label transfer classifies newly sequenced cells by identifying anchors between a reference and query dataset with FindTransferAnchors and assigning labels via TransferData. In the SciBet paper it serves as the reference baseline against which SciBet's multinomial-MLE classifier is compared for accuracy and classification speed.

## When to use

Use when benchmarking single-cell cell-type classifiers against Seurat v3, or when transferring cell-type labels from an annotated reference scRNA-seq dataset to a query dataset with FindTransferAnchors + TransferData.

## When NOT to use

- When the goal is the paper's proposed method — SciBet achieves order-of-magnitude speed gains and uses absolute similarity to a null dataset, which Seurat v3 label transfer does not provide.
- When the input is full-length read count data normalized as TPM + log-normalization rather than UMI data, since the default Seurat v3 UMI preprocessing does not apply.
- When the task requires flagging unrepresented cell types; the paper's benchmark explicitly used scmapCluster/TransferData 'without considering the unassigned cells'.

## Inputs

- Annotated reference scRNA-seq expression matrix (e.g., UMI count matrix with cell-type labels)
- Query scRNA-seq UMI count matrix to be classified
- Optionally: simulated scRNA-seq count matrices with 500 genes and 1,000–50,000 cells for benchmarking

## Outputs

- Per-cell cell-type label predictions for the query dataset
- Timing/runtimes per dataset recorded as e.g. scibet_timing_benchmark.csv with derived throughput (cells per second)
- Timing comparison plot (e.g., fig2e_timing_comparison.png, runtimes vs dataset size on log scale)

## How to apply

Preprocess UMI count data with Seurat v3's default normalization (size-factor 10,000 per cell, add one, log normalization), then identify integration anchors between the annotated reference and the query with the FindTransferAnchors function. Assign cell-type labels to query cells by running TransferData using the identified anchors. For timing comparisons, run classification on each dataset size (e.g., simulated scRNA-seq matrices with 500 genes and 1,000–50,000 cells) on a single CPU and record elapsed wall-clock time, excluding consideration of unassigned cells when mirroring the paper's benchmark protocol. Aggregate per-dataset runtimes and derive throughput in cells per second for comparison against SciBet and scmap.

## Related tools

- **Seurat v3** (Anchor-based label transfer classifier used as a baseline: FindTransferAnchors to identify anchors and TransferData to classify query cells)
- **SciBet** (Multinomial-MLE classifier installed via devtools::install_github("PaulingLiu/scibet"), used to benchmark speed against Seurat v3 label transfer) — https://github.com/PaulingLiu/scibet
- **scmap** (scmapCluster-based classification baseline run under the same single-CPU timing conditions)
- **ggplot2** (Plots runtimes versus dataset size on a logarithmic scale to reproduce the Fig. 2e timing comparison)

## Examples

```
anchors <- FindTransferAnchors(reference = ref, query = query, dims = 1:30); predictions <- TransferData(anchorset = anchors, refdata = ref$celltype)
```

## Evaluation signals

- Timing benchmark CSV contains per-dataset runtimes for all three methods plus derived throughput in cells per second for each of the four simulated dataset sizes (1,000 / 10,000 / 20,000 / 50,000 cells).
- Seurat v3 runtime is recorded under identical single-CPU conditions as SciBet and scmap so runtimes are comparable.
- SciBet attains throughput of approximately 100,000 cells per second, orders of magnitude faster than Seurat v3 across all dataset sizes.
- Plot (fig2e_timing_comparison.png) shows runtimes versus dataset size on a logarithmic scale covering all three methods.
- UMI preprocessing matches the paper's protocol: size-factor 10,000 normalization, add one, log normalization, default parameters.

## Limitations

- No explicit version numbers reported for Seurat v3 minor version, complicating exact reproduction.
- The benchmark does not consider unassigned cells, so Seurat v3's behavior on query cells whose types are absent from the reference is not evaluated in the timing comparison.
- Seurat v3 was orders of magnitude slower than SciBet in classification speed, limiting its use for very large query sets.

## Evidence

- [discussion] For Seurat v3, we identified anchors and classified cells using the FindTransferAnchors function and TransferData functions: "For Seurat v3, we identified anchors and classified cells using the FindTransferAnchors function and TransferData functions"
- [other] SciBet outperformed scmap and Seurat v3 by orders of magnitude in classification speed, ~100,000 cells/s: [evidence span withheld — anchor status: not_found]
- [discussion] normalizing the UMI count of each cell with size-factor 10,000, adding one and then log normalization: "normalizing the UMI count of each cell with size-factor 10,000, adding one and then log normalization"
- [readme] README: reproduction scripts location in repository: "The scripts for producing all the quantitative results in our manuscript can be found in [scripts](./scripts)."
