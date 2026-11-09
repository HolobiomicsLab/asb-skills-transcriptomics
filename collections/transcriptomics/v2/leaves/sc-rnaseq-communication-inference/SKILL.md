---
name: sc-rnaseq-communication-inference
description: Use when you have a normalized single-cell (or spatially resolved) transcriptomics
  matrix with cell group labels and want to infer neuron-neuron communication networks,
  especially neurotransmitter signaling absent from protein-only ligand-receptor databases.
when_to_use_negative:
- Your signaling question concerns short-range autocrine/paracrine protein signaling
  only; NeuronChat is designed for long-distance neurotransmitter-based neuron-neuron
  communication and NeuronChatDB may not cover non-neural interactions
- You lack cell group labels — the method requires pre-defined cell groups/clusters
  as input to createNeuronChat
- Only protein-level interaction databases (OmniPath, STRING) are needed for non-neural
  cell types; NeuronChatDB is neural-specific
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0605
tools:
- name: NeuronChat
  role: 'Primary R package: creates the NeuronChat object, infers communication networks
    via run_NeuronChat, aggregates via net_aggregation, and visualizes via netVisual_circle_neuron'
  repo: https://github.com/Wei-BioMath/NeuronChat
- name: NeuronChatDB
  role: Neural-specific interaction database (183 mouse, 190 human entries) curated
    from KEGG, IUPHAR/BPS Guide to PHARMACOLOGY and literature; supports update via
    update_interactionDB
  repo: https://github.com/Wei-BioMath/NeuronChat
- name: CellChat
  role: Baseline comparison tool; implemented with the same computational workflow
    as NeuronChat for benchmarking ligand-abundance computation without functional
    gene grouping
  repo: https://github.com/sqjin/CellChat
- name: CellPhoneDB
  role: Baseline comparison tool for communication inference benchmarking
- name: NeuronChatAnalysis2022
  role: Repository of scripts used to reproduce figures and results of the study,
    including benchmark analysis
  repo: https://github.com/Wei-BioMath/NeuronChatAnalysis2022
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-36800-w
    title: Inferring neuron-neuron communications from single-cell transcriptomics
      through NeuronChat
  source_content_hash: sha256:866f5e928acbe6e05fad4a9ce1303db96d02d97f2a78969cc12c0928b9e3ef84
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Wei-BioMath/NeuronChat
  tools:
  - CellChat
  - CellPhoneDB
  - NeuronChat
  - NeuronChatAnalysis2022
  - NeuronChatDB
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# sc-rnaseq-communication-inference

## Summary

Infer neural-specific cell-cell communication networks from scRNA-seq data using NeuronChat, which models neurotransmitter-based signaling via the curated NeuronChatDB (373 interactions; 190 human, 183 mouse) and aggregates per-interaction link strengths into an intercellular network.

## When to use

Use when you have a normalized single-cell (or spatially resolved) transcriptomics matrix with cell group labels and want to infer neuron-neuron communication networks, especially neurotransmitter signaling absent from protein-only ligand-receptor databases.

## When NOT to use

- Your signaling question concerns short-range autocrine/paracrine protein signaling only; NeuronChat is designed for long-distance neurotransmitter-based neuron-neuron communication and NeuronChatDB may not cover non-neural interactions
- You lack cell group labels — the method requires pre-defined cell groups/clusters as input to createNeuronChat
- Only protein-level interaction databases (OmniPath, STRING) are needed for non-neural cell types; NeuronChatDB is neural-specific

## Inputs

- Normalized scRNA-seq gene-by-cell count matrix (e.g., cortex_data from Yao 2021)
- Cell group/subclass label vector (group.by), e.g., cell_subclass from metadata
- Species identifier for NeuronChatDB ('mouse' or 'human')
- For spatial extension: spatial metadata with centroid_x, centroid_y coordinates and slice IDs (e.g., MERFISH 2021, seqFISH+ 2019 datasets)

## Outputs

- NeuronChat object (x@net) containing per-interaction communication strength matrices
- Aggregated intercellular communication network (weighted directed matrix from net_aggregation)
- Benchmark metrics (AUROC/AUPRC) when ranked against a ground-truth connectivity network

## How to apply

Preprocess counts by normalizing each cell to 10,000, adding a pseudocount of 1, and natural-log transforming. Create a NeuronChat object with createNeuronChat() specifying DB='mouse' or 'human' and a cell grouping vector, then run run_NeuronChat(x, M=100): genes expressed in <25% of cells are filtered out, ensemble expression per group is the Tukey trimean, ligand abundance uses AND logic (geometric mean) across functional gene groups and OR logic (arithmetic mean) within redundant groups, and significance is assessed by label-permutation (100 permutations) with Benjamini-Hochberg correction, zeroing links with p >= 0.05. Aggregate per-interaction networks with net_aggregation(x@net, method='weight'); for benchmarking, the thresholded-weight variant thresholds link weights ak at the 80% quantile. For spatial data, optionally remove links with spatial proximity enrichment scores below a threshold computed from 1000 cell/spot label permutations.

## Related tools

- **NeuronChat** (Primary R package: creates the NeuronChat object, infers communication networks via run_NeuronChat, aggregates via net_aggregation, and visualizes via netVisual_circle_neuron) — https://github.com/Wei-BioMath/NeuronChat
- **NeuronChatDB** (Neural-specific interaction database (183 mouse, 190 human entries) curated from KEGG, IUPHAR/BPS Guide to PHARMACOLOGY and literature; supports update via update_interactionDB) — https://github.com/Wei-BioMath/NeuronChat
- **CellChat** (Baseline comparison tool; implemented with the same computational workflow as NeuronChat for benchmarking ligand-abundance computation without functional gene grouping) — https://github.com/sqjin/CellChat
- **CellPhoneDB** (Baseline comparison tool for communication inference benchmarking)
- **NeuronChatAnalysis2022** (Repository of scripts used to reproduce figures and results of the study, including benchmark analysis) — https://github.com/Wei-BioMath/NeuronChatAnalysis2022

## Examples

```
x <- createNeuronChat(normalized_count_mtx, DB='mouse', group.by=cell_group_vector); x <- run_NeuronChat(x, M=100); net_aggregated_x <- net_aggregation(x@net, method='weight')
```

## Evaluation signals

- Reproducing the ALM benchmark: aggregated inferred network should recover AUROC 0.764 and AUPRC 0.768 against the retrograde-labeling ground truth (VISp comparison: AUROC 0.832), within numerical tolerance
- All communication links with BH-corrected p >= 0.05 must have communication strength set to zero in x@net
- Genes expressed in fewer than 25% of cells must be excluded before ligand/target abundance computation
- Aggregated network from net_aggregation(..., method='weight') must be square with dimensions equal to the number of cell groups and directed weights from sender to target groups
- Permutation count M=100 in run_NeuronChat should match the reported setting; changing M alters p-value stability (O(n) time trade-off)

## Limitations

- Ligand/target abundance estimated from transcriptomics may be inconsistent with actual protein or metabolite levels, since single-cell proteomics and metabolomics lag in coverage and throughput
- NeuronChatDB may contain missing curated interaction entries leading to inference bias, and lacks subunit stoichiometries of heteromeric receptors
- Permutation-based p-value computation has a trade-off between accuracy and speed (time complexity O(n) in the number of permutations)
- The model uses only expressions of synthetic enzymes and vesicular transporters; stoichiometric metabolism effects and auxiliary/downstream gene knowledge are not yet incorporated
- CellChat and CellPhoneDB-style workflows without functional gene grouping use AND logic (geometric mean or minimum) for all contributing genes, which can dramatically underestimate ligand abundance for small-molecule neurotransmitters like glutamate and GABA

## Evidence

- [intro] NeuronChatDB curation: "Development of NeuronChat required manual curation of a new neural-specific database containing 373 entries of intercellular molecular interactions for both human (190) and mouse (183)."
- [discussion] preprocessing: "The RNA counts for each cell were divided by the total counts in the cell and multiplied by a scale factor (10,000 by default), and these values are added with a pseudocount of 1 and then natural-log"
- [discussion] gene filter: "Because the genes expressed in less than 25% of cells are filtered out"
- [discussion] aggregation logic: "the AND logic (i.e., geometric mean) is applied among different groups of genes; since the genes within the same group are redundant for the same function, the OR logic (i.e., arithmetic mean) is"
- [discussion] significance testing: "The statistical significance of each communication is calculated through the permutation test by randomly permutating group labels of cells"
- [discussion] benchmark threshold: "This method is the default setting for benchmarking NeuronChat, with threshold ak defined as the 80% quantile"
- [readme] usage workflow: [evidence span withheld — anchor status: external_doc]
- [methods] example data: "# load example data: cortex data from Yao 2021, processed
data(list='cortex_data')"
