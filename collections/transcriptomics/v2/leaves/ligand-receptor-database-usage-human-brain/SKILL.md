---
name: ligand-receptor-database-usage-human-brain
description: Use when inferring neural-specific ligand-receptor communication from
  human single-nucleus/single-cell RNA-seq of brain tissue (e.g., cortical ASD vs
  control) and you must pick the correct species-specific interaction DB.
when_to_use_negative:
- Input data are from mouse — use DB='mouse' (183 entries) instead of DB='human'
- The analysis requires small-molecule neurotransmitter signaling under non-neural,
  protein-only ligand-receptor databases such as CellChat or CellPhoneDB, which exclude
  glutamate and GABA and underestimate their ligand abundance
- The task is spatial proximity modeling without expression matrices — spatially constrained
  workflows need centroid metadata and proximity thresholds, not just the interaction
  DB
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3308
tools:
- name: NeuronChat
  role: R package whose createNeuronChat(DB='human') loads the curated human NeuronChatDB
    and infers neural-specific communication networks via run_NeuronChat
  repo: https://github.com/Wei-BioMath/NeuronChat
- name: NeuronChatDB
  role: Neural-specific interaction database with 190 human entries curated from KEGG,
    IUPHAR/BPS Guide to PHARMACOLOGY and literature, including neurotransmitter signaling
  repo: https://github.com/Wei-BioMath/NeuronChat
- name: CellChat
  role: Dependency of NeuronChat and a comparator workflow; its protein-based ligand-receptor
    computation without functional gene grouping can underestimate small-molecule
    neurotransmitter ligand abundance
  repo: https://github.com/sqjin/CellChat
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-36800-w
    title: Inferring neuron-neuron communications from single-cell transcriptomics
      through NeuronChat
  source_content_hash: sha256:870deb3ed3f87f40543a7e0ead7128a6a2c73404c8a6d6a109289aca6627ac66
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Wei-BioMath/NeuronChat
  tools:
  - CellChat
  - NeuronChat
  - NeuronChatDB
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# ligand-receptor-database-usage-human-brain

## Summary

Select and apply NeuronChat's curated human neural interaction database (NeuronChatDB, 190 human entries of 373 total) when inferring cell-cell communication from human brain snRNA-seq data, e.g., ASD vs control cortical comparisons.

## When to use

Use when inferring neural-specific ligand-receptor communication from human single-nucleus/single-cell RNA-seq of brain tissue (e.g., cortical ASD vs control) and you must pick the correct species-specific interaction DB.

## When NOT to use

- Input data are from mouse — use DB='mouse' (183 entries) instead of DB='human'
- The analysis requires small-molecule neurotransmitter signaling under non-neural, protein-only ligand-receptor databases such as CellChat or CellPhoneDB, which exclude glutamate and GABA and underestimate their ligand abundance
- The task is spatial proximity modeling without expression matrices — spatially constrained workflows need centroid metadata and proximity thresholds, not just the interaction DB

## Inputs

- Normalized human snRNA-seq count matrix (scaled to 10,000, pseudocount 1, log-transformed)
- Cell-type/group annotation vector for group.by
- NeuronChatDB human interaction database (DB='human', 190 entries)

## Outputs

- NeuronChat object with inferred communication probability network (x@net)
- Aggregated weight/count network via net_aggregation(x@net, method='weight')
- Per-interaction significance (permutation p-values, BH-corrected) and differential comparison tables/plots

## How to apply

Build the NeuronChat object with createNeuronChat(normalized_count_mtx, DB='human', group.by=cell_type_vector), ensuring the input is a normalized count matrix (counts scaled to 10,000, plus pseudocount 1, natural-log transformed). The 'human' option selects NeuronChatDB's 190 curated human interaction entries, curated from KEGG and IUPHAR/BPS Guide to PHARMACOLOGY and literature; choose 'mouse' only for mouse data. Then infer networks with run_NeuronChat(x, M=100), which permutes cell-group labels 100 times and keeps links significant at p < 0.05 after Benjamini-Hochberg correction, setting non-significant strengths to zero. Because NeuronChatDB includes neurotransmitter signaling (glutamate, GABA) with functional gene grouping (AND geometric mean across groups, OR arithmetic mean within redundant groups), it is preferred over CellChat/CellPhoneDB protein-based databases for neural tissue.

## Related tools

- **NeuronChat** (R package whose createNeuronChat(DB='human') loads the curated human NeuronChatDB and infers neural-specific communication networks via run_NeuronChat) — https://github.com/Wei-BioMath/NeuronChat
- **NeuronChatDB** (Neural-specific interaction database with 190 human entries curated from KEGG, IUPHAR/BPS Guide to PHARMACOLOGY and literature, including neurotransmitter signaling) — https://github.com/Wei-BioMath/NeuronChat
- **CellChat** (Dependency of NeuronChat and a comparator workflow; its protein-based ligand-receptor computation without functional gene grouping can underestimate small-molecule neurotransmitter ligand abundance) — https://github.com/sqjin/CellChat

## Examples

```
library(NeuronChat); x <- createNeuronChat(normalized_count_mtx, DB='human', group.by = celltype_labels); x <- run_NeuronChat(x, M=100); net_aggregated_x <- net_aggregation(x@net, method='weight')
```

## Evaluation signals

- The DB argument resolves to the human subset of NeuronChatDB (190 entries), not the mouse subset (183 entries)
- All retained communication links satisfy permutation p < 0.05 after Benjamini-Hochberg correction with M=100 permutations; non-significant strengths are zero
- Input matrix preprocessing matches the package default: scale factor 10,000, pseudocount of 1, natural-log transform
- Aggregated weight/count per condition can be reproduced with net_aggregation(x@net, method='weight') and compared across conditions with mergeNeuronChat/compareInteractions
- In the ASD vs control application, NLGN3-mediated links to oligodendrocyte precursor cells (e.g., NRXN1-NLGN3, NRXN2-NLGN3, NRXN3-NLGN3) show reduced strength in ASD

## Limitations

- NeuronChatDB may contain missing curated interaction entries, leading to inference bias
- Subunit stoichiometries of heteromeric receptors are not contained in NeuronChatDB and are largely unknown
- Ligand/target abundance is estimated from transcriptomics and may be inconsistent with protein or metabolite levels
- Genes expressed in less than 25% of cells are filtered out, which can remove low-abundance but biologically relevant interactions

## Evidence

- [methods] Human DB selection for human data: "# creat NeuronChat object; choose the database 'mouse' for mouse data; 'human' for human data"
- [intro] NeuronChatDB curation and size: "Development of NeuronChat required manual curation of a new neural-specific database containing 373 entries of intercellular molecular interactions for both human (190) and mouse (183)."
- [discussion] Significance filtering rule: "communications with p-value < 0.05 are considered significant; for non-significant communication links, the communication strength values are set to be zeros"
- [discussion] Count preprocessing default: "The RNA counts for each cell were divided by the total counts in the cell and multiplied by a scale factor (10,000 by default), and these values are added with a pseudocount of 1 and then natural-log"
- [readme] Basic usage from README: [evidence span withheld — anchor status: external_doc]
