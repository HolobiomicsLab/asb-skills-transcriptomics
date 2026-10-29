---
name: cross-dataset-node-importance-comparison
description: Use when you have scBONITA rule-inference node importance scores for
  KEGG pathway networks from one or more scRNA-seq datasets and need to check whether
  importance scores carry context-dependent signaling information beyond topology
  alone.
when_to_use_negative:
- When the goal is to rank genes for experimental prioritization rather than to validate
  that importance scores encode context-dependent signaling information.
- When subpopulations across datasets are not biologically comparable (e.g., monocytes
  vs CD8+ T cells), since cross-dataset Pearson correlations assume matched cell types
  over shared network nodes.
- When networks are not available as KEGG or GRAPHML topologies from which centrality
  measures can be computed.
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: scBONITA
  role: Source of the node importance scores produced by rule inference on KEGG pathway
    networks from scRNA-seq data
  repo: https://github.com/Thakar-Lab/scBONITA
- name: Python 3.6
  role: Runtime for computing centrality measures with networkx and Spearman/Pearson
    correlations
- name: KEGG
  role: Source of pathway topologies (e.g., hsa00010, hsa00020) shared across datasets
    for node-wise correlation
- name: GRAPHML
  role: Network file format used to load pathway topologies for centrality computation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41540-022-00246-5
    title: Executable models of immune signaling pathways in HIV-associated atherosclerosis
  source_content_hash: sha256:5bd51a0e64a8389115c859898580d969c29c4f4c78ada773053722bb656de2a5
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# cross-dataset-node-importance-comparison

## Summary

Validate scBONITA node importance scores by testing whether they are decoupled from network topology (Spearman correlations against six centrality measures) and reproducible across datasets (Pearson correlations of importance scores over shared KEGG networks between comparable cell subpopulations).

## When to use

Use when you have scBONITA rule-inference node importance scores for KEGG pathway networks from one or more scRNA-seq datasets and need to check whether importance scores carry context-dependent signaling information beyond topology alone.

## When NOT to use

- When the goal is to rank genes for experimental prioritization rather than to validate that importance scores encode context-dependent signaling information.
- When subpopulations across datasets are not biologically comparable (e.g., monocytes vs CD8+ T cells), since cross-dataset Pearson correlations assume matched cell types over shared network nodes.
- When networks are not available as KEGG or GRAPHML topologies from which centrality measures can be computed.

## Inputs

- scBONITA rule-inference node importance scores for KEGG pathway networks (per dataset, per cell subpopulation)
- KEGG pathway network topologies in GRAPHML format
- Paired subpopulation identity mappings across datasets (e.g., cytotoxic T cells in SCP256 vs CD8+ T cells in the HIV/AS dataset)

## Outputs

- importance_centrality_correlations.csv containing the importance-vs-centrality Spearman correlation table and the cross-subpopulation Pearson correlation table
- Per-network Spearman p-values for six centrality measures
- Cross-dataset Pearson correlation coefficients per shared KEGG network

## How to apply

Load the node importance scores from scBONITA rule-inference output for each KEGG pathway network. Using networkx in Python 3.6, compute six node centrality measures from each network's topology (degree, betweenness, closeness, eigenvector, in-degree, out-degree) and calculate Spearman rank correlations between importance scores and each centrality measure per network; non-significance at the p<0.01 threshold indicates importance carries information distinct from topology. Then pair comparable cell subpopulations across datasets (e.g., cytotoxic T cells in SCP256 vs CD8+ T cells in the HIV/AS dataset GSE198339) and compute Pearson correlations of importance scores across the nodes of each shared KEGG network, expecting values in the 0.71–0.91 range. Write both the importance-vs-centrality and cross-subpopulation correlation tables to importance_centrality_correlations.csv.

## Related tools

- **scBONITA** (Source of the node importance scores produced by rule inference on KEGG pathway networks from scRNA-seq data) — https://github.com/Thakar-Lab/scBONITA
- **Python 3.6** (Runtime for computing centrality measures with networkx and Spearman/Pearson correlations)
- **KEGG** (Source of pathway topologies (e.g., hsa00010, hsa00020) shared across datasets for node-wise correlation)
- **GRAPHML** (Network file format used to load pathway topologies for centrality computation)

## Examples

```
import networkx as nx, scipy.stats as st; G = nx.read_graphml('hsa00010.graphml'); rho, p = st.spearmanr(imp_scores, list(nx.degree_centrality(G).values()))
```

## Evaluation signals

- All Spearman correlations between scBONITA importance scores and the six centrality measures are non-significant (p<0.01 threshold) per KEGG network.
- Pearson correlations of node importance scores between comparable subpopulations (cytotoxic T cells vs CD8+ T cells) fall between 0.71 and 0.91.
- The correlation tables are written to importance_centrality_correlations.csv with both the importance-vs-centrality and cross-subpopulation results.
- Centrality measures computed per network include all six: degree, betweenness, closeness, eigenvector, in-degree, and out-degree.
- Cross-dataset comparisons are restricted to nodes of shared KEGG networks only.

## Limitations

- The null result against centrality measures indicates importance is decoupled from topology, but does not by itself identify what biological signal the score encodes.
- scBONITA inference is expected to improve when pure cell populations are sequenced; mixed PBMC subpopulations add variability to importance scores.
- Pre-classified subpopulations were used to reduce variability and improve specificity of scBONITA-PA, so results depend on the upstream clustering/annotation choice.

## Evidence

- [other] decoupling importance from topology: [evidence span withheld — anchor status: not_found]
- [methods] GRAPHML pathway list: "Usually networks from non-KEGG sources, saved in GRAPHML format. The default value is to use sll the networks initially used for rule inference."
- [discussion] attractor/steady-state and rule inference scope: "we developed the scBONITA algorithms for regulatory rule inference, network simulation, pathway analysis, and attractor/steady-state analysis."
- [readme] conda environment setup: "Install the scBonita conda environment from the provided yml file OR the provided spec-file (PREFERRED), and activate the conda environment."
- [readme] pathway analysis script invocation: "python3.6 pathwayAnalysis.py --dataFile "data/trainingData.csv" --conditions "data/conditions.txt" --contrast "data/contrast.txt" --conditions_separator "comma" --contrast_separator "comma""
