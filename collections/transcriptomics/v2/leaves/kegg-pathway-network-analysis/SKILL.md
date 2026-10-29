---
name: kegg-pathway-network-analysis
description: Use when scRNA-seq data must be analyzed in the context of KEGG signaling
  pathway networks, or when node importance scores need to be compared against network
  centrality measures to test topology-dependence.
when_to_use_negative:
- Input networks are not pathway topologies and are not in GRAPHML or KEGG form —
  scBONITA rule inference is defined over topologically characterized networks
- The goal is conventional over-representation analysis of differentially expressed
  genes — the article notes conventional ORA methods were inefficacious at identifying
  coherent molecular processes here
- Running on a desktop without adequate resources — the README warns rule inference
  and attractor analysis on large scRNA-seq datasets require in excess of 10G RAM
  on a high-performance/SLURM system
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0203
tools:
- name: scBONITA
  role: Downloads KEGG pathway topologies and performs Boolean rule inference and
    node importance scoring on scRNA-seq data
  repo: https://github.com/Thakar-Lab/scBONITA
- name: Python 3.6
  role: Runtime for the scBONITA pipeline and networkx centrality/correlation analyses
- name: conda
  role: Environment setup via the provided spec-file or yml for scBonita dependencies
- name: KEGG
  role: Source of pathway topologies (e.g., hsa00010, hsa00020) used as the network
    layer for rule inference
- name: GRAPHML
  role: File format for supplying non-KEGG pathway networks to scBONITA pathway analysis
    via --pathwayList
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41540-022-00246-5
    title: Executable models of immune signaling pathways in HIV-associated atherosclerosis
  source_content_hash: sha256:adbc3c5aa19ce8b0f7b586b101aa120b48babb3333474320543dfddaa37455ae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Thakar-Lab/scBONITA
  tools:
  - GRAPHML
  - KEGG
  - Python
  - Python 3.6
  - conda
  - scBONITA
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# kegg-pathway-network-analysis

## Summary

Retrieves KEGG pathway topologies (GRAPHML) and uses scBONITA rule inference on scRNA-seq data to compute node importance scores and pathway-level signaling states for each network, enabling evaluation of whether importance scores decouple from pure network topology via centrality correlations.

## When to use

Use when scRNA-seq data must be analyzed in the context of KEGG signaling pathway networks, or when node importance scores need to be compared against network centrality measures to test topology-dependence.

## When NOT to use

- Input networks are not pathway topologies and are not in GRAPHML or KEGG form — scBONITA rule inference is defined over topologically characterized networks
- The goal is conventional over-representation analysis of differentially expressed genes — the article notes conventional ORA methods were inefficacious at identifying coherent molecular processes here
- Running on a desktop without adequate resources — the README warns rule inference and attractor analysis on large scRNA-seq datasets require in excess of 10G RAM on a high-performance/SLURM system

## Inputs

- Processed scRNA-seq training data matrix (tab/comma-delimited, genes as rows, cell IDs as columns)
- Metadata/conditions file (cells x conditions, binary 0/1 entries)
- Contrast file listing pairs of conditions to compare
- KEGG pathway identifiers (e.g., hsa00010, hsa00020) or GRAPHML pathway network files
- scBONITA rule-inference output with node importance scores per KEGG network

## Outputs

- Inferred Boolean regulatory rules per KEGG pathway network
- Node importance scores per network and condition
- Spearman correlation table of importance scores vs six centrality measures (importance_centrality_correlations.csv)
- Cross-subpopulation Pearson correlation table of node importance scores

## How to apply

Set up the scBONITA pipeline by providing a training data matrix (cells as columns, features as rows, log2(TPM+1)-style values), a 0/1 metadata conditions file, and a contrast file, then run pipeline.py with --getKEGGPathways 1 and a list of KEGG pathway IDs (e.g., '00010' '00020') so scBONITA downloads the KEGG pathway topologies for rule inference. Rule inference produces Boolean rules and node importance scores per KEGG network; downstream pathway analysis can also accept non-KEGG networks in GRAPHML format via --pathwayList. To test whether importance scores capture information beyond topology, compute six centrality measures (degree, betweenness, closeness, eigenvector, in-/out-degree) with networkx on each KEGG network and calculate Spearman rank correlations against the importance scores, accepting the result only if all correlations are non-significant at p<0.01. For cross-dataset validation, pair comparable subpopulations (cytotoxic T cells vs CD8+ T cells) and compute Pearson correlations of node importance across shared network nodes, expecting values in the 0.71–0.91 range, writing both tables to importance_centrality_correlations.csv.

## Related tools

- **scBONITA** (Downloads KEGG pathway topologies and performs Boolean rule inference and node importance scoring on scRNA-seq data) — https://github.com/Thakar-Lab/scBONITA
- **Python 3.6** (Runtime for the scBONITA pipeline and networkx centrality/correlation analyses)
- **conda** (Environment setup via the provided spec-file or yml for scBonita dependencies)
- **KEGG** (Source of pathway topologies (e.g., hsa00010, hsa00020) used as the network layer for rule inference)
- **GRAPHML** (File format for supplying non-KEGG pathway networks to scBONITA pathway analysis via --pathwayList)

## Examples

```
python3.6 pipeline.py --dataFile "example.csv" --fullPipeline 1 --maxNodes 20000 --separator "," --listOfKEGGPathways "00010" "00020" --getKEGGPathways 1 --organism hsa cvThreshold None
```

## Evaluation signals

- All Spearman correlations between node importance scores and the six centrality measures (degree, betweenness, closeness, eigenvector, in-/out-degree) are non-significant at the p<0.01 threshold
- Pearson correlations of node importance scores between comparable subpopulations across datasets (cytotoxic T cells vs CD8+ T cells) fall within 0.71–0.91
- Setup ran with --getKEGGPathways 1 and the KEGG pathways (e.g., '00010' '00020') were downloaded for rule inference
- Both correlation tables were written to importance_centrality_correlations.csv

## Limitations

- scBONITA is designed for SLURM systems and large scRNA-seq datasets require in excess of 10G RAM; rule inference and attractor analysis are not recommended on a desktop computer
- scBONITA assumes scRNA-seq data samples the state space of a dynamic Boolean network in lieu of time-series data
- Inference is expected to improve when pure cell populations are sequenced; mixed PBMC subpopulations and small cohorts (four AS+ and four AS- subjects) limit power
- Nodes in the supplied C code must be recompiled to reflect dataset size: NODE is the number of nodes and CELL the number of cells to be sampled

## Evidence

- [methods] It downloads the KEGG pathways hsa00010 and hsa00020 for rule inference.: "It downloads the KEGG pathways hsa00010 and hsa00020 for rule inference."
- [readme] Paths to GRAPHML files that should be used for scBONITA analysis. Usually networks from non-KEGG sources, saved in GRAPHML format.: "Paths to GRAPHML files that should be used for scBONITA analysis. Usually networks from non-KEGG sources, saved in GRAPHML format."
- [readme] We find that the amount of RAM required for large scRNA-seq datasets is in excess of 10G; we hence don't recommend running rule inference and attractor analysis on a desktop computer.: "We find that the amount of RAM required for large scRNA-seq datasets is in excess of 10G; we hence don't recommend running rule inference and attractor analysis on a desktop computer."
- [abstract] scBONITA infers executable dynamic pathway models and performs a perturbation analysis to identify high impact genes: "scBONITA infers executable dynamic pathway models and performs a perturbation analysis to identify high impact genes"
