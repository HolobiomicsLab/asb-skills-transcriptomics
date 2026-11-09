---
name: single-cell-rna-seq-expression-scoring
description: Use when you have a preprocessed scRNA-seq expression matrix with cell-type
  labels and need to compute, rank, or compare expression-based ligand-receptor interaction
  scores across CCC methods and prior-knowledge resources.
when_to_use_negative:
- The method of interest additionally infers intracellular processes (e.g., NicheNet,
  SoptSC, full Cytotalk) — these are not directly comparable to expression-based ligand-receptor
  scoring
- The biological question concerns long-distance endocrine signalling, which scRNA-seq
  gene expression within the dataset does not capture
- Only raw unprocessed counts without cell-type annotation are available, since sender-receiver
  scoring requires cell-type-labelled expression data
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: LIANA
  role: Unified framework to run any combination of the 7 CCC scoring methods with
    any of the 16 resources and compute the consensus
  repo: https://github.com/saezlab/liana
- name: CellChat
  role: Provides the probability score based on expression of differentially expressed
    transmitter and receiver genes and their mediators
- name: CellPhoneDB
  role: Provides the truncated-mean score; handles heteromeric complexes via minimum
    subunit expression by default
- name: Connectome
  role: Provides the weight_norm score derived from the product of normalised transmitter
    and receiver expression
- name: NATMI
  role: Provides the mean-expression edge weight (transmitter and receiver gene expression
    product); runs with connectomeDB2020 or user-supplied interaction lists
  repo: https://github.com/asrhou/NATMI.git
- name: SingleCellSignalR
  role: Provides the LRscore, a regularised score calculated using the squared expression
    of the transmitter and receiver; custom fork allows custom ligand-receptor resources
  repo: https://github.com/saezlab/SingleCellSignalR_v1
- name: iTALK
  role: Inspiration for the logFC Mean score, the mean of the logged one-versus-all
    fold change
- name: Cytotalk
  role: Inspiration for the Crosstalk score derived from transmitter and receiver
    expression
- name: OmnipathR
  role: R client providing prior-knowledge ligand-receptor resources from the OmniPath
    web service
  repo: github.com/saezlab/OmnipathR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-30755-0
    title: Comparison of methods and resources for cell-cell communication inference
      from single-cell RNA-Seq data
  source_content_hash: sha256:670b991c0e24199cea1dfbffc5d0f9b48d312b0f0c9b8986d27c4ce9ad90d05f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saezlab/liana
  tools:
  - CellChat
  - CellPhoneDB
  - Connectome
  - Cytotalk
  - LIANA
  - NATMI
  - OmnipathR
  - SingleCellSignalR
  - iTALK
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# single-cell-rna-seq-expression-scoring

## Summary

Score candidate ligand-receptor interactions from single-cell RNA-seq expression data using method-specific formulas (CellChat probability, CellPhoneDB truncated mean, Connectome weight_norm, NATMI mean-expression edge weight, SingleCellSignalR LRscore, iTALK logFC Mean, Cytotalk Crosstalk score) decoupled from their default prior-knowledge resources, to quantify and compare predicted cell-cell communication.

## When to use

Use when you have a preprocessed scRNA-seq expression matrix with cell-type labels and need to compute, rank, or compare expression-based ligand-receptor interaction scores across CCC methods and prior-knowledge resources.

## When NOT to use

- The method of interest additionally infers intracellular processes (e.g., NicheNet, SoptSC, full Cytotalk) — these are not directly comparable to expression-based ligand-receptor scoring
- The biological question concerns long-distance endocrine signalling, which scRNA-seq gene expression within the dataset does not capture
- Only raw unprocessed counts without cell-type annotation are available, since sender-receiver scoring requires cell-type-labelled expression data

## Inputs

- Preprocessed scRNA-seq expression matrix (e.g., from breast cancer subtypes, cord blood mononuclear cells, pancreatic islets, or colorectal cancer datasets) with cell-type labels
- Ligand-receptor prior-knowledge resource (e.g., CellChatDB, CellPhoneDB, Ramilowski, ConnectomeDB, LRdb, or OmniPath via OmnipathR)
- Choice of CCC scoring method (CellChat probability, CellPhoneDBv2 truncated mean, Connectome weight_norm, NATMI mean-expression edge weight, SingleCellSignalR LRscore, iTALK logFC Mean, Cytotalk Crosstalk score)

## Outputs

- Per-interaction score table (ligand, receptor, sender cell type, receiver cell type, method score) for each method-resource-dataset combination
- Magnitude-ranked top-1000 interaction list per method-resource-dataset combination
- Presence-absence matrix of top-1000 predicted interactions per method and per resource
- Median pairwise Jaccard indices for method-method and resource-resource comparisons (e.g., median_jaccard_fig5.csv) and a Fig 5-style similarity heatmap

## How to apply

Preprocess the scRNA-seq data with LIANA's pipeline, then decouple each of the 7 CCC methods from its default resource and run every method with each of the 16 resources (e.g., CellChatDB, CellPhoneDB, Ramilowski, ConnectomeDB, LRdb) on the dataset. Compute the method-specific score per ligand-receptor pair per sender-receiver cell-type pair: CellChat probability (based on expression of differentially expressed transmitter/receiver genes and their mediators), CellPhoneDBv2 truncated mean (average expression of transmitter and receivers, minimum expression for heteromeric complex subunits by default), Connectome weight_norm (product of normalised transmitter and receiver expression), NATMI mean-expression edge weight (transmitter/receiver expression product), SingleCellSignalR LRscore (regularised score using squared expression), iTALK-inspired logFC Mean (mean of logged one-versus-all fold change), and Cytotalk-inspired Crosstalk score. Rank interactions by each method's magnitude rank and retain the top 1000 per method-resource-dataset combination; optionally aggregate into the methods' consensus. To validate method-vs-resource sensitivity, build presence-absence matrices of top-1000 hits, compute pairwise Jaccard indices, and take the median across datasets for method-method and resource-resource comparisons.

## Related tools

- **LIANA** (Unified framework to run any combination of the 7 CCC scoring methods with any of the 16 resources and compute the consensus) — https://github.com/saezlab/liana
- **CellChat** (Provides the probability score based on expression of differentially expressed transmitter and receiver genes and their mediators)
- **CellPhoneDB** (Provides the truncated-mean score; handles heteromeric complexes via minimum subunit expression by default)
- **Connectome** (Provides the weight_norm score derived from the product of normalised transmitter and receiver expression)
- **NATMI** (Provides the mean-expression edge weight (transmitter and receiver gene expression product); runs with connectomeDB2020 or user-supplied interaction lists) — https://github.com/asrhou/NATMI.git
- **SingleCellSignalR** (Provides the LRscore, a regularised score calculated using the squared expression of the transmitter and receiver; custom fork allows custom ligand-receptor resources) — https://github.com/saezlab/SingleCellSignalR_v1
- **iTALK** (Inspiration for the logFC Mean score, the mean of the logged one-versus-all fold change)
- **Cytotalk** (Inspiration for the Crosstalk score derived from transmitter and receiver expression)
- **OmnipathR** (R client providing prior-knowledge ligand-receptor resources from the OmniPath web service) — github.com/saezlab/OmnipathR

## Evaluation signals

- Top-1000 overlap (median pairwise Jaccard) falls near expected values: ~0.080 for different-method comparisons and ~0.119 for different-resource comparisons, showing consistently low overlap either way
- Heteromeric complexes are handled by the method default (e.g., CellPhoneDBv2 uses the minimum expression of complex subunits)
- Each method-resource-dataset combination yields exactly 1000 magnitude-ranked interactions, and presence-absence matrices reproduce the pairwise Jaccard structure of the Fig 5-style heatmap
- Scores are comparable across resources because each method is decoupled from its default resource and run with all 16 resources
- Predicted interactions are coherent with independent modalities (spatial colocalisation, cytokine activities, receptor protein abundance)

## Limitations

- No gold standard exists and the biological ground truth for CCC is largely unknown, so no definitive best method can be identified
- scRNA-seq gene expression does not capture long-distance endocrine signalling events or processes preceding transmitter-receiver interactions (translation, processing, secretion, diffusion)
- Benchmarks cannot distinguish simple co-expression from actual CCC events, and current methods are limited to single species
- Both the choice of method and resource strongly influence predictions, so the number of interactions between two cell types is a biased proxy for communication intensity

## Evidence

- [intro] CellChat probability score: "CellChat#14CellChatDB(1) Probability—based on the expression of differentially expressed transmitter and receiver genes and their mediators"
- [intro] CellPhoneDB truncated mean with complex handling: "CellPhoneDBv2#8CellPhoneDB(1) Truncated Mean—average expression of transmitter and receivers, the minimum expression (by default) of heteromeric complex of subunits"
- [intro] SingleCellSignalR LRscore formula: "SingleCellSignalR#12LRdb(1) LRscore—a regularised score calculated using the squared expression of the transmitter and receiver"
- [intro] iTALK logFC Mean definition: "logFC Mean†—iTALK-inspired6 logFC means, derived using the mean of the logged one-versus-all fold change"
- [readme] LIANA combines methods and resources: "LIANA enables the use of any combination of ligand-receptor methods and resources, and their consensus."
- [readme] NATMI requires expression tables with cell-type labels: [evidence span withheld — anchor status: external_doc]
- [abstract] Method and resource both impact predictions: "We then examine all possible combinations of methods and resources and show that both strongly influence the predicted intercellular interactions."
