---
name: fitch-hartigan-small-parsimony-scoring
description: Use when you have a single-cell phylogeny (cells as leaves) plus cell-level
  categorical metadata and need a per-cell numeric score of plasticity/local state
  heterogeneity on the tree.
when_to_use_negative:
- The metadata of interest is continuous/numeric rather than categorical — Fitch-Hartigan
  parsimony requires discrete character states
- There is no cell-resolved phylogeny available (cells as leaves); the score is defined
  entirely by subtree structure
- You want relative fitness quantification from the phylogeny — the article explicitly
  notes such approaches 'are not directly supported within our software'
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0084
tools:
- name: PhyloVision
  role: Implements the single-cell plasticity analysis that quantifies categorical
    metadata for each leaf on the tree
  repo: https://github.com/YosefLab/VISION
- name: ape
  role: 'Preprocessing: collapse.singles removes unifurcations from the Cassiopeia-reconstructed
    tree before scoring'
- name: Cassiopeia
  role: Provides the input single-cell phylogenies (Cassiopeia-Hybrid reconstruction)
    that the plasticity scores are computed on
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100200
    title: Interactive, integrated analysis of single-cell transcriptomic and phylogenetic
      data with PhyloVision
  source_content_hash: sha256:9fd2753e950fe25af6cc0012fc37fa5eee77fab8b16cb50a8a4d34039ed9790a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/VISION
  tools:
  - Cassiopeia
  - PhyloVision
  - R
  - ape
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# fitch-hartigan-small-parsimony-scoring

## Summary

Reconstructs PhyloVision's single-cell plasticity analysis, which quantifies how heterogeneous the ancestral neighborhoods of each cell are for a user-provided categorical meta-data column by running Fitch-Hartigan small-parsimony on every internal-node subtree and averaging edge-normalized parsimony scores along root-to-leaf paths.

## When to use

Use when you have a single-cell phylogeny (cells as leaves) plus cell-level categorical metadata and need a per-cell numeric score of plasticity/local state heterogeneity on the tree.

## When NOT to use

- The metadata of interest is continuous/numeric rather than categorical — Fitch-Hartigan parsimony requires discrete character states
- There is no cell-resolved phylogeny available (cells as leaves); the score is defined entirely by subtree structure
- You want relative fitness quantification from the phylogeny — the article explicitly notes such approaches 'are not directly supported within our software'

## Inputs

- Rooted single-cell phylogeny with one leaf per cell (e.g., Newick tree from Cassiopeia-Hybrid, unifurcations collapsed with ape::collapse.singles)
- Cell-level categorical metadata table (e.g., metadata from GEO series GSE161363 / dataset GSM4905334), keyed by cell identifier such as sampleID

## Outputs

- plasticity_scores.csv: one row per cell, one column per categorical metadata variable, containing per-cell plasticity scores
- Per-cell numeric meta-data entries added to the phylogeny (e.g., a plasticity index stored on sampleID)

## How to apply

Load the reconstructed phylogeny (e.g., the CP004 tumor phylogeny inferred with Cassiopeia-Hybrid, with unifurcations collapsed via ape's collapse.singles) and the cell-level categorical metadata table (e.g., downloaded from GEO series GSE161363, dataset GSM4905334). For each categorical column, run the Fitch-Hartigan small-parsimony algorithm (Fitch 1971; Hartigan 1973) on the subtree defined by every internal node of the tree, then normalize each subtree's parsimony score by its number of edges. For each leaf, perform a depth-first traversal from the root and average the normalized subtree parsimony scores of all internal nodes along that path to yield the single-cell plasticity score. Assemble one plasticity vector per categorical variable (analogous to per-cell meta-data entries like scMetRate keyed on sampleID) and write them to plasticity_scores.csv with one row per cell and one column per categorical variable. The rationale is that cells whose ancestral path is internally homogeneous for a trait have low total parsimony, while cells whose lineage traverses mixed ancestral states accumulate higher normalized parsimony.

## Related tools

- **PhyloVision** (Implements the single-cell plasticity analysis that quantifies categorical metadata for each leaf on the tree) — https://github.com/YosefLab/VISION
- **ape** (Preprocessing: collapse.singles removes unifurcations from the Cassiopeia-reconstructed tree before scoring)
- **Cassiopeia** (Provides the input single-cell phylogenies (Cassiopeia-Hybrid reconstruction) that the plasticity scores are computed on)

## Examples

```
R> collapsed_tree <- ape::collapse.singles(tree)  # then run Fitch-Hartigan subtree parsimony per categorical column and write plasticity_scores.csv
```

## Evaluation signals

- plasticity_scores.csv exists and has exactly one row per cell in the phylogeny and one column per categorical metadata variable
- Cells whose ancestral neighborhoods are entirely homogeneous for a category receive near-minimal scores (low subtree parsimony)
- Cells spanning mixed ancestral states along their root-to-leaf path receive higher scores than homogeneous-lineage cells
- Each per-subtree parsimony score is normalized by the subtree's edge count, so per-node contributions are edge-count independent
- The plasticity column stored per cell (e.g., on sampleID) matches the recomputed averages of normalized Fitch-Hartigan subtree scores along each leaf's path

## Limitations

- PhyloVision is implemented in R, which the authors note 'may provide challenges for utilizing separate software available in Python, like Cassiopeia' when composing pipelines
- Analysis of the metastasis dataset 'still lacks experimental validation to support the generated hypotheses', so plasticity scores are exploratory
- Scores are only meaningful for categorical characters on a tree whose leaves map 1:1 to metadata rows (cell barcodes/sampleID must match)

## Evidence

- [methods] plasticity analysis enables per-leaf scoring of categorical metadata: "PhyloVision enables users to quantify the plasticity of a particular categorical metadata for each leaf on the tree"
- [methods] tree preprocessing with ape: "collapsed using the collapse.singles function in the R package ape"
- [methods] source of cell-level categorical metadata: "Meta data corresponding to each cell was downloaded from NCBI GEO, series GSE161363 and dataset GSM4905334"
- [discussion] R implementation limits Python interop: "our implementation of PhyloVision in the R programming language may provide challenges for utilizing separate software available in Python"
