---
name: spearman-rank-correlation-analysis
description: Use when you have per-node scores (e.g., scBONITA node importance from
  rule inference on KEGG networks) plus graph-derived node statistics and need to
  test their association via Spearman rank correlation.
when_to_use_negative:
- When the goal is to validate that scores DO track network topology — this workflow
  is designed to demonstrate independence from centrality measures, not to predict
  topology from scores
- When node scores and network topology come from different, non-corresponding networks
  — the correlation must be computed per shared network
- When the data are paired per-cell measurements across two conditions; Pearson correlation
  of importance vectors (as done for cross-subpopulation comparison) is more appropriate
  than rank correlation
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_2259
tools:
- name: scBONITA
  role: Produces the node importance scores (via rule inference and perturbation analysis)
    that are correlated against centrality measures
  repo: https://github.com/Thakar-Lab/scBONITA
- name: Python 3.6
  role: Runtime for computing centrality measures with networkx and Spearman/Pearson
    correlations; scBONITA requires Python 3.6 or newer
- name: KEGG
  role: Source of pathway topologies (e.g., hsa00010, hsa00020) for which node centrality
    is computed
- name: GRAPHML
  role: Format for non-KEGG network topologies that can be used as pathwayList input
- name: conda
  role: Environment setup for the scBONITA pipeline (conda create --name scBonita
    --file spec-file.txt)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41540-022-00246-5
    title: Executable models of immune signaling pathways in HIV-associated atherosclerosis
  source_content_hash: sha256:a9900367f68a3484c176eedcd7f0d6c1906933e34b9767e24f930d2d96218620
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
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_2259
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# spearman-rank-correlation-analysis

## Summary

Statistical test of whether scBONITA node importance scores are correlated with network-topology centrality measures (degree, betweenness, closeness, eigenvector, in-/out-degree) using Spearman rank correlation, to demonstrate that importance scores carry information distinct from network topology alone.

## When to use

Use when you have per-node scores (e.g., scBONITA node importance from rule inference on KEGG networks) plus graph-derived node statistics and need to test their association via Spearman rank correlation.

## When NOT to use

- When the goal is to validate that scores DO track network topology — this workflow is designed to demonstrate independence from centrality measures, not to predict topology from scores
- When node scores and network topology come from different, non-corresponding networks — the correlation must be computed per shared network
- When the data are paired per-cell measurements across two conditions; Pearson correlation of importance vectors (as done for cross-subpopulation comparison) is more appropriate than rank correlation

## Inputs

- scBONITA node importance scores per KEGG pathway network (rule-inference output from scRNA-seq data, e.g., SCP256 or HIV/AS GSE198339 datasets)
- KEGG pathway network topologies (GRAPHML format or downloaded KEGG pathways such as hsa00010, hsa00020)
- Cell subpopulation labels for cross-dataset pairing (e.g., cytotoxic T cells, CD8+ T cells)

## Outputs

- Table of Spearman correlations between node importance scores and six centrality measures per network (importance_centrality_correlations.csv)
- Table of cross-subpopulation Pearson correlations of node importance scores (importance_centrality_correlations.csv)

## How to apply

Load the scBONITA node importance scores learned from scRNA-seq data (rule-inference output) for each KEGG pathway network. Compute six node centrality measures (degree, betweenness, closeness, eigenvector, in-degree, out-degree) from each network topology using networkx in Python 3.6. Calculate Spearman rank correlations between importance scores and each centrality measure per network, using a significance threshold of p<0.01. Non-significant correlations indicate the importance score decouples from topology alone; pair this with cross-dataset Pearson correlations of importance scores for comparable cell subpopulations (e.g., cytotoxic T cells vs CD8+ T cells), which should fall between 0.71 and 0.91. Write both correlation tables to importance_centrality_correlations.csv.

## Related tools

- **scBONITA** (Produces the node importance scores (via rule inference and perturbation analysis) that are correlated against centrality measures) — https://github.com/Thakar-Lab/scBONITA
- **Python 3.6** (Runtime for computing centrality measures with networkx and Spearman/Pearson correlations; scBONITA requires Python 3.6 or newer)
- **KEGG** (Source of pathway topologies (e.g., hsa00010, hsa00020) for which node centrality is computed)
- **GRAPHML** (Format for non-KEGG network topologies that can be used as pathwayList input)
- **conda** (Environment setup for the scBONITA pipeline (conda create --name scBonita --file spec-file.txt))

## Examples

```
import networkx as nx; from scipy.stats import spearmanr; G = nx.read_graphml('hsa00010.graphml'); rho, p = spearmanr(imp['importance'], [d for _, d in G.degree()])
```

## Evaluation signals

- All Spearman correlations between node importance scores and the six centrality measures are non-significant (p-values > 0.01 at the p<0.01 threshold)
- Cross-dataset Pearson correlations of importance scores between comparable subpopulations (cytotoxic T cells vs CD8+ T cells) fall within 0.71–0.91
- Both correlation tables are written to importance_centrality_correlations.csv with one row per KEGG network and per centrality measure
- Centrality measures are computed on the same node sets as the importance scores for each KEGG network (no dropped/isolated nodes)

## Limitations

- scBONITA inference is expected to improve when pure cell populations are sequenced rather than mixed PBMC subpopulations, which may affect the importance scores being correlated
- Small sample size (four AS+ and four AS- subjects) can limit the power of the underlying dataset
- A non-significant Spearman correlation does not prove independence from all topological properties — only from the six centrality measures tested at p<0.01

## Evidence

- [results] scBONITA node importance scores for KEGG networks trained on the HIV/AS dataset are not correlated with any of six measures of node centrality, showing the importance score carries information distinct from network topology.: [evidence span withheld — anchor status: not_found]
- [methods] Compute six node centrality measures (e.g., degree, betweenness, closeness, eigenvector, in-/out-degree) from each KEGG network topology using networkx in Python 3.6.: [evidence span withheld — anchor status: not_found]
- [methods] Calculate Spearman rank correlations between scBONITA importance scores and each centrality measure per network, confirming no measure is significant at p<0.01: [evidence span withheld — anchor status: not_found]
- [methods] Verify the expected ranges: all importance-centrality Spearman p-values >0.01, and cross-dataset subpopulation Pearson correlations fall between 0.71 and 0.91.: [evidence span withheld — anchor status: not_found]
- [readme] scBonita prioritizes genes based on their impact on signaling: "prioritizes genes based on their impact on signaling"
