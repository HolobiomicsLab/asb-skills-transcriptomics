---
name: scrna-seq-preprocessing-normalization
description: Use when raw single-cell or spatial transcriptomic count matrices must
  be prepared before inferring neuron-neuron communication networks with NeuronChat,
  or before benchmarking inferred connectivity against anatomical tracing data.
when_to_use_negative:
- The input matrix is already normalized (e.g., log-normalized by Seurat's NormalizeData
  at the default scale factor) — re-applying scaling would distort ligand/target abundance
  estimates
- 'The goal is protein- or metabolite-level inference: single-cell proteomics and
  metabolomics lag behind transcriptomics in coverage/throughput, and NeuronChat estimates
  ligand/target abundance from transcriptomics, which may be inconsistent with protein
  or metabolite levels'
- The analysis targets short-range autocrine/paracrine protein signaling, for which
  NeuronChat's neural-specific, neurotransmitter-oriented database (NeuronChatDB)
  is not the appropriate interaction source
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: NeuronChat
  role: Consumes the normalized count matrix via createNeuronChat to build the communication-inference
    object; its preprocessing convention (10,000 scale, pseudocount 1, log transform,
    25% gene filter) defines the normalization standard
  repo: https://github.com/Wei-BioMath/NeuronChat
- name: CellChat
  role: Dependency of NeuronChat (CellChat >= 1.1.3); also used in the same computational
    workflow for comparison, with the caveat that CellChat's ligand-abundance computation
    without functional gene grouping can underestimate small-molecule neurotransmitter
    signaling
  repo: https://github.com/sqjin/CellChat
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-36800-w
    title: Inferring neuron-neuron communications from single-cell transcriptomics
      through NeuronChat
  source_content_hash: sha256:a87840bb24b8a1c1ce85b2f1b288ee694ac9c3539bedcefa48cad2c4cbce4b6b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Wei-BioMath/NeuronChat
  tools:
  - CellChat
  - NeuronChat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# scrna-seq-preprocessing-normalization

## Summary

Normalize single-cell RNA-seq count matrices (library-size scaling to 10,000, pseudocount of 1, natural-log transform) and filter low-prevalence genes (<25% of cells) so they are suitable input for NeuronChat's neural communication inference and downstream benchmarking such as the VISp projection-network comparison.

## When to use

Use when raw single-cell or spatial transcriptomic count matrices must be prepared before inferring neuron-neuron communication networks with NeuronChat, or before benchmarking inferred connectivity against anatomical tracing data.

## When NOT to use

- The input matrix is already normalized (e.g., log-normalized by Seurat's NormalizeData at the default scale factor) — re-applying scaling would distort ligand/target abundance estimates
- The goal is protein- or metabolite-level inference: single-cell proteomics and metabolomics lag behind transcriptomics in coverage/throughput, and NeuronChat estimates ligand/target abundance from transcriptomics, which may be inconsistent with protein or metabolite levels
- The analysis targets short-range autocrine/paracrine protein signaling, for which NeuronChat's neural-specific, neurotransmitter-oriented database (NeuronChatDB) is not the appropriate interaction source

## Inputs

- Raw single-cell RNA-seq count matrix (e.g., cortex_data from Yao 2021, subset to region_name == 'VISp')
- Per-cell total UMI/read counts
- Cell metadata with grouping labels (e.g., cell_subclass)

## Outputs

- Normalized, natural-log-transformed count matrix scaled to 10,000 counts per cell with pseudocount 1
- Gene-filtered expression matrix (genes expressed in >=25% of cells retained)
- Normalized matrix ready as normalized_count_mtx input to createNeuronChat

## How to apply

Divide the RNA counts of each cell by the total counts in the cell and multiply by a scale factor of 10,000 (default), add a pseudocount of 1, and natural-log transform the result. Filter out genes expressed in less than 25% of cells, since NeuronChat's inference excludes such genes. Pass the normalized matrix into createNeuronChat with the appropriate interaction database ('mouse' for mouse data such as the Yao 2021 cortex_data VISp subset, 'human' for human data) and a cell-grouping vector (e.g., target_df_single$cell_subclass). Correct preprocessing is essential because NeuronChat computes ensemble average expression per cell group using Tukey's trimean and communication strength as the product of ligand and target abundance, both sensitive to the normalization scale.

## Related tools

- **NeuronChat** (Consumes the normalized count matrix via createNeuronChat to build the communication-inference object; its preprocessing convention (10,000 scale, pseudocount 1, log transform, 25% gene filter) defines the normalization standard) — https://github.com/Wei-BioMath/NeuronChat
- **CellChat** (Dependency of NeuronChat (CellChat >= 1.1.3); also used in the same computational workflow for comparison, with the caveat that CellChat's ligand-abundance computation without functional gene grouping can underestimate small-molecule neurotransmitter signaling) — https://github.com/sqjin/CellChat

## Examples

```
x <- createNeuronChat(normalized_count_mtx, DB='mouse', group.by = cell_group_vector); x <- run_NeuronChat(x, M=100)
```

## Evaluation signals

- Each cell's normalized values sum to approximately 10,000 before the log transform (log1p scale after pseudocount)
- All retained genes are expressed in at least 25% of cells; low-prevalence genes are absent from the matrix used by run_NeuronChat
- The downstream VISp benchmark reproduces AUROC ≈ 0.832 and AUPRC ≈ 0.915 against retrograde-labeling projection labels when M=100 permutations, BH correction, and p < 0.05 significance filtering are applied
- createNeuronChat accepts the matrix without scale errors and the cell-grouping vector length matches the number of cells

## Limitations

- Ligand/target abundance is estimated from transcriptomics and may be inconsistent with protein or metabolite levels, since single-cell proteomics and metabolomics lag behind transcriptomics in coverage and throughput
- Genes expressed in less than 25% of cells are filtered out, so low-abundance but biologically relevant signaling genes can be lost
- There is a trade-off between accuracy and computation speed for permutation-based p-value calculations (time complexity O(n) in the number of permutations), which constrains how downstream inference on the normalized data can be scaled

## Evidence

- [discussion] The RNA counts for each cell were divided by the total counts in the cell and multiplied by a scale factor (10,000 by default), and these values are added with a pseudocount of 1 and then natural-log: "The RNA counts for each cell were divided by the total counts in the cell and multiplied by a scale factor (10,000 by default), and these values are added with a pseudocount of 1 and then natural-log"
- [discussion] Because the genes expressed in less than 25% of cells are filtered out: "Because the genes expressed in less than 25% of cells are filtered out"
- [methods] x <- createNeuronChat(normalized_count_mtx,DB='mouse',group.by = cell_group_vector) # use DB='human' for human data: [evidence span withheld — anchor status: external_doc]
- [readme] NeuronChat is to infer, visualize and analyze neural-specific cell-cell communication from single cell transcriptomics or spatially resolved transcriptomics data.: "The goal of NeuronChat is to infer, visualize and analyze neural-specific cell-cell communication from single cell transcriptomics or spatially resolved transcriptomics data."
