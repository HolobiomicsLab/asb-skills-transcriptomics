---
name: network-node-centrality-computation
description: Use when you have directed network topologies (e.g., KEGG GRAPHML pathways)
  plus a per-node data-driven importance score and need to test whether that score
  is decoupled from — or redundant with — network topology.
when_to_use_negative:
- When you only need pathway-level dysregulation calls rather than per-node topology-vs-importance
  comparisons — use scBONITA pathway analysis instead
- When the networks are not available as topologies (no GRAPHML/KEGG structure), since
  centralities cannot be computed
- When per-node importance scores are absent — centrality computation alone does not
  address the topology-independence question
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_0092
tools:
- name: scBONITA
  role: Produces the per-node importance scores (via perturbation analysis of learned
    Boolean rule models) that are correlated against centrality measures
  repo: https://github.com/Thakar-Lab/scBONITA
- name: Python 3.6
  role: Runtime environment for computing centrality measures with networkx and Spearman/Pearson
    correlations
- name: KEGG
  role: Source of the pathway network topologies (e.g., hsa00010, hsa00020) on which
    centrality is computed
- name: GRAPHML
  role: Format for non-KEGG networks used as pathwayList inputs to scBONITA analysis
- name: conda
  role: Environment management for the scBonita pipeline (spec-file or yml based)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41540-022-00246-5
    title: Executable models of immune signaling pathways in HIV-associated atherosclerosis
  source_content_hash: sha256:a19424ec80b4de42bbb5a337ddc1ce2793eb1ecb00e6bbb7dc0abee9f74a8bd5
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
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# network-node-centrality-computation

## Summary

Compute multiple node centrality measures (e.g., degree, betweenness, closeness, eigenvector, in-/out-degree) from signaling network topologies with networkx in Python 3.6, then test correlations against data-driven node importance scores (e.g., scBONITA perturbation-based importance) to determine whether importance captures information beyond topology.

## When to use

Use when you have directed network topologies (e.g., KEGG GRAPHML pathways) plus a per-node data-driven importance score and need to test whether that score is decoupled from — or redundant with — network topology.

## When NOT to use

- When you only need pathway-level dysregulation calls rather than per-node topology-vs-importance comparisons — use scBONITA pathway analysis instead
- When the networks are not available as topologies (no GRAPHML/KEGG structure), since centralities cannot be computed
- When per-node importance scores are absent — centrality computation alone does not address the topology-independence question

## Inputs

- scBONITA node importance scores for KEGG pathway networks learned from scRNA-seq data (rule-inference output)
- KEGG pathway network topologies in GRAPHML format (e.g., hsa00010, hsa00020 or other KEGG hsa pathways)
- Node importance score tables for comparable cell subpopulations across two datasets (e.g., SCP256 cytotoxic T cells and HIV/AS CD8+ T cells)

## Outputs

- Spearman correlation table of node importance scores vs six centrality measures per network
- Pearson correlation table of node importance scores across matched cell subpopulations for each shared KEGG network
- importance_centrality_correlations.csv containing both correlation tables

## How to apply

Load the per-node importance scores (here, scBONITA node importance scores from rule-inference output on KEGG pathway networks) and the corresponding network topologies. Compute six centrality measures (degree, betweenness, closeness, eigenvector, in-degree, out-degree) per network using networkx in Python 3.6. Calculate Spearman rank correlations between importance scores and each centrality measure per network; the article treats non-significance at p<0.01 as evidence that the importance score carries information distinct from topology alone. Optionally pair comparable cell subpopulations across datasets (cytotoxic T cells in SCP256 vs CD8+ T cells in HIV/AS) and compute Pearson correlations of importance scores across nodes of each shared KEGG network, expecting values in the 0.71–0.91 range. Write both correlation tables (importance-vs-centrality and cross-subpopulation) to importance_centrality_correlations.csv.

## Related tools

- **scBONITA** (Produces the per-node importance scores (via perturbation analysis of learned Boolean rule models) that are correlated against centrality measures) — https://github.com/Thakar-Lab/scBONITA
- **Python 3.6** (Runtime environment for computing centrality measures with networkx and Spearman/Pearson correlations)
- **KEGG** (Source of the pathway network topologies (e.g., hsa00010, hsa00020) on which centrality is computed)
- **GRAPHML** (Format for non-KEGG networks used as pathwayList inputs to scBONITA analysis)
- **conda** (Environment management for the scBonita pipeline (spec-file or yml based))

## Examples

```
python3.6 pathwayAnalysis.py --dataFile "data/trainingData.csv" --conditions "data/conditions.txt" --contrast "data/contrast.txt" --conditions_separator "comma" --contrast_separator "comma" --pathwayList "hsa00010"
```

## Evaluation signals

- All Spearman correlations between scBONITA importance scores and the six centrality measures are non-significant at the p<0.01 threshold in every KEGG network
- Cross-dataset subpopulation Pearson correlations of node importance scores (cytotoxic T cells vs CD8+ T cells) fall within 0.71–0.91
- The output file importance_centrality_correlations.csv exists and contains both the importance-vs-centrality and cross-subpopulation correlation tables
- Six distinct centrality measures (degree, betweenness, closeness, eigenvector, in-degree, out-degree) are computed for each network without errors

## Limitations

- Centrality measures are purely topological and cannot capture context-dependent signaling flow; lack of correlation does not indicate the importance score is more correct, only that it encodes data-dependent information
- scRNA-seq technical limitations cause nonspecific distortions of expression and fewer detected differences, which can propagate into the importance scores being compared
- Comparing subpopulations across datasets requires pre-classified, comparable cell types (e.g., cytotoxic T cells vs CD8+ T cells) to keep cross-dataset correlations interpretable

## Evidence

- [other] scBONITA node importance scores for KEGG networks trained on the HIV/AS dataset are not correlated with any of six measures of node centrality, showing the importance score carries information distinct from network topology.: [evidence span withheld — anchor status: not_found]
- [readme] README notes scBONITA prioritizes genes based on their impact on signaling and infers Boolean rules for topologically characterized networks.: "uses scRNA-seq data to infer Boolean regulatory rules for topologically characterized networks"
- [readme] README specifies network input format and Python version.: "Usually networks from non-KEGG sources, saved in GRAPHML format."
- [methods] Methods require Python 3.6 or newer for scBONITA.: "scBONITA requires Python 3.6 or newer."
