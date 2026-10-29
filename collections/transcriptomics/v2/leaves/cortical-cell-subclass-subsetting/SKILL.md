---
name: cortical-cell-subclass-subsetting
description: Use when preparing the cortex_data example dataset (Yao 2021) for NeuronChat
  benchmarking or region-specific communication inference, when cells must be restricted
  to one cortical region (e.g., ALM or VISp) and to Glutamatergic CTX subclasses before
  network inference.
when_to_use_negative:
- The input is spatial transcriptomics data (e.g., MERFISH or seqFISH+ slices), where
  cells are handled per-spot/slice with centroid coordinates rather than by subsetting
  subclass labels from cortex_data
- No cell_subclass or region annotation exists in the metadata, so the ALM/Glutamatergic
  CTX restriction cannot be applied faithfully
- The goal is inference on the full multi-region cortex dataset rather than a single-region
  benchmark against a retrograde-labeled projection network
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: NeuronChat
  role: Consumes the subsetted gene-by-cell matrix and cell_subclass grouping via
    createNeuronChat(DB='mouse', group.by=cell_subclass) to build the communication
    network
  repo: https://github.com/Wei-BioMath/NeuronChat
- name: NeuronChatAnalysis2022
  role: Repository of scripts reproducing the paper's results, including preparation
    of the cortex_data example dataset and the ALM benchmark workflow
  repo: https://github.com/Wei-BioMath/NeuronChatAnalysis2022
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-36800-w
    title: Inferring neuron-neuron communications from single-cell transcriptomics
      through NeuronChat
  source_content_hash: sha256:49683f5964c554e0ddd24dd0fdbd237d244299a36bd00e9e2df532bd7b4e998c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Wei-BioMath/NeuronChat
  tools:
  - CellChat
  - NeuronChat
  - NeuronChatAnalysis2022
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# cortical-cell-subclass-subsetting

## Summary

Subset the processed Yao 2021 cortex scRNA-seq dataset (cortex_data) to cells of a target cortical region — for the ALM benchmark, the ALM region's Glutamatergic CTX subclasses — so that NeuronChat infers communication networks on a neuroanatomically coherent cell population grouped by cell_subclass.

## When to use

Use when preparing the cortex_data example dataset (Yao 2021) for NeuronChat benchmarking or region-specific communication inference, when cells must be restricted to one cortical region (e.g., ALM or VISp) and to Glutamatergic CTX subclasses before network inference.

## When NOT to use

- The input is spatial transcriptomics data (e.g., MERFISH or seqFISH+ slices), where cells are handled per-spot/slice with centroid coordinates rather than by subsetting subclass labels from cortex_data
- No cell_subclass or region annotation exists in the metadata, so the ALM/Glutamatergic CTX restriction cannot be applied faithfully
- The goal is inference on the full multi-region cortex dataset rather than a single-region benchmark against a retrograde-labeled projection network

## Inputs

- cortex_data example dataset from Yao 2021 (processed gene-by-cell count matrix, loaded via data(list='cortex_data'))
- Cell metadata (meta) with region label (ALM/VISp) and cell_subclass annotations

## Outputs

- Region-restricted, Glutamatergic-CTX-subclass-only cell subset of cortex_data
- cell_subclass grouping vector suitable for createNeuronChat(group.by=...)

## How to apply

Load the processed example data with data(list='cortex_data'), obtaining target_df and the accompanying meta table from Yao 2021. From the metadata, select cells whose region label is the target area (ALM for the projection-network benchmark) and whose cell_subclass is a Glutamatergic CTX subclass, retaining the gene-by-cell count matrix restricted to those cells. Carry the cell_subclass labels through as the group.by vector so that createNeuronChat(x, DB='mouse', group.by=cell_subclass) infers communication between subclass-level groups rather than individual cells. This subsetting defines the cell population on which run_NeuronChat(M=100) is run; region-mismatched subsets (e.g., mixing ALM and VISp cells) would invalidate comparison against the ALM retrograde-labeling ground truth used for the AUROC 0.764 benchmark.

## Related tools

- **NeuronChat** (Consumes the subsetted gene-by-cell matrix and cell_subclass grouping via createNeuronChat(DB='mouse', group.by=cell_subclass) to build the communication network) — https://github.com/Wei-BioMath/NeuronChat
- **NeuronChatAnalysis2022** (Repository of scripts reproducing the paper's results, including preparation of the cortex_data example dataset and the ALM benchmark workflow) — https://github.com/Wei-BioMath/NeuronChatAnalysis2022

## Examples

```
data(list='cortex_data'); idx <- meta$region=='ALM' & grepl('CTX', meta$cell_subclass); x <- createNeuronChat(cortex_data[, idx], DB='mouse', group.by=meta$cell_subclass[idx])
```

## Evaluation signals

- Every retained cell carries the ALM region label and a Glutamatergic CTX cell_subclass; no inter-region mixture remains in the subset
- The subset matrix is a gene-by-cell count matrix whose cell identities match the filtered meta rows from cortex_data
- The grouping vector passed to createNeuronChat is cell_subclass (not individual cells or clusters from a fresh clustering)
- Downstream benchmark on the subset recovers AUROC 0.764 and AUPRC 0.768 against the ALM retrograde-labeling ground truth (VISp comparison yields 0.832), within numerical tolerance

## Limitations

- Subsetting quality depends entirely on the Yao 2021 metadata annotations; mislabeled regions or subclasses propagate directly into the inferred network
- The benchmark setting ties this subset to the ALM retrograde-labeling ground truth; other regions need their own experimental connectivity references
- run_NeuronChat subsequently filters genes expressed in less than 25% of cells, so very small subclass subsets may lose sparse genes and alter inference

## Evidence

- [methods] Load processed cortex example data with region/subclass filtering: "# load example data: cortex data from Yao 2021, processed
data(list='cortex_data')"
- [readme] NeuronChat object is created with a cell-subclass grouping vector on the subset matrix: [evidence span withheld — anchor status: external_doc]
- [methods] Database choice for mouse data grouping: "# creat NeuronChat object; choose the database 'mouse' for mouse data; 'human' for human data"
- [other] Task-card workflow specifies the ALM Glutamatergic CTX subset from cortex_data: [evidence span withheld — anchor status: not_found]
