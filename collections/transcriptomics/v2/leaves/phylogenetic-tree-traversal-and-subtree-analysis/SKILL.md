---
name: phylogenetic-tree-traversal-and-subtree-analysis
description: Use when a rooted phylogeny with cells as leaves must be traversed subtree-by-subtree
  (e.g., to compute per-cell plasticity of categorical meta-data such as GEO GSM4905334
  annotations), or when per-leaf summaries require averaging node-level statistics
  along root-to-leaf paths.
when_to_use_negative:
- The metadata of interest is continuous gene-expression scores rather than categorical
  annotations — PhyloVision's plasticity mechanism is defined for categorical metadata
  columns
- You want relative fitness quantification from the phylogeny — the authors note such
  approaches 'are not directly supported within our software'
- The tree contains uncollapsed unifurcations — the pipeline expects unifurcations
  collapsed with ape's collapse.singles first
edam_topics:
- http://edamontology.org/topic_0194
tools:
- name: PhyloVision
  role: Provides the pipeline phyloAnalyze step that computes plasticity scores for
    each categorical meta-data column and adds them as per-cell numeric meta-data;
    also clusters cells on the phylogeny via breadth-first search over internal nodes
  repo: https://github.com/YosefLab/VISION
- name: ape
  role: R package whose collapse.singles function collapses unifurcations in the reconstructed
    tree before subtree traversal and parsimony computation
- name: Cassiopeia
  role: Source of the reconstructed single-cell phylogenies (Cassiopeia-Hybrid) on
    which the traversal and plasticity analysis is run
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100200
    title: Interactive, integrated analysis of single-cell transcriptomic and phylogenetic
      data with PhyloVision
  source_content_hash: sha256:1136254f852ef535852b50937d26662d64e1b171272560673812d09bb657b2bd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/VISION
  tools:
  - Cassiopeia
  - PhyloVision
  - R
  - ape
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0194
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# phylogenetic-tree-traversal-and-subtree-analysis

## Summary

Traverses a single-cell phylogeny (e.g., a Cassiopeia-reconstructed CP004 tumor tree) over its internal nodes and subtrees to quantify plasticity of categorical metadata per leaf, using Fitch-Hartigan small-parsimony scores normalized by subtree edge count.

## When to use

Use when a rooted phylogeny with cells as leaves must be traversed subtree-by-subtree (e.g., to compute per-cell plasticity of categorical meta-data such as GEO GSM4905334 annotations), or when per-leaf summaries require averaging node-level statistics along root-to-leaf paths.

## When NOT to use

- The metadata of interest is continuous gene-expression scores rather than categorical annotations — PhyloVision's plasticity mechanism is defined for categorical metadata columns
- You want relative fitness quantification from the phylogeny — the authors note such approaches 'are not directly supported within our software'
- The tree contains uncollapsed unifurcations — the pipeline expects unifurcations collapsed with ape's collapse.singles first

## Inputs

- Rooted single-cell phylogeny (Newick/tree object, e.g., Cassiopeia-Hybrid output with unifurcations collapsed via ape::collapse.singles)
- Cell-level categorical metadata table (e.g., downloaded from NCBI GEO series GSE161363, dataset GSM4905334), keyed by cell barcode matching tree leaves

## Outputs

- plasticity_scores.csv: one row per cell in the phylogeny and one column per categorical metadata variable, containing per-cell plasticity scores
- Per-cell numeric meta-data entries (e.g., a plasticity index on sampleID corresponding to scMetRate) added to the analysis object

## How to apply

Load the phylogeny reconstructed with Cassiopeia-Hybrid and collapse unifurcations with ape's collapse.singles so internal nodes define meaningful subtrees. For each categorical metadata column, run the Fitch-Hartigan small-parsimony algorithm (Fitch 1971; Hartigan 1973) on every subtree defined by each internal node, then normalize each subtree's parsimony score by the number of edges in that subtree. For each leaf, perform a depth-first traversal from the root to the leaf and average the normalized parsimony scores of all internal nodes along that path to obtain the single-cell plasticity score. Assemble the per-cell plasticity vectors (one per categorical variable) and write plasticity_scores.csv with one row per cell and one column per categorical variable. The same breadth-first traversal over internal nodes is the mechanism PhyloVision uses to cluster cells on the phylogeny, so the traversal utilities generalize beyond plasticity scoring.

## Related tools

- **PhyloVision** (Provides the pipeline phyloAnalyze step that computes plasticity scores for each categorical meta-data column and adds them as per-cell numeric meta-data; also clusters cells on the phylogeny via breadth-first search over internal nodes) — https://github.com/YosefLab/VISION
- **ape** (R package whose collapse.singles function collapses unifurcations in the reconstructed tree before subtree traversal and parsimony computation)
- **Cassiopeia** (Source of the reconstructed single-cell phylogenies (Cassiopeia-Hybrid) on which the traversal and plasticity analysis is run)

## Examples

```
library(ape); tree <- collapse.singles(tree)
```

## Evaluation signals

- plasticity_scores.csv exists and has exactly one row per cell (leaf) in the phylogeny
- Each categorical variable contributes one column, matching the metadata table's categorical columns one-to-one
- Cells whose ancestral neighborhoods are entirely homogeneous for a category receive near-minimal scores (low parsimony)
- Cells spanning mixed ancestral states along their root-to-leaf path receive higher scores
- Every subtree parsimony score was normalized by the number of edges in that subtree before path averaging

## Limitations

- PhyloVision's R implementation may pose interoperability challenges with Python software such as Cassiopeia
- Plasticity scoring is defined per leaf for categorical metadata only; continuous traits are handled by different analyses (e.g., Geary's C autocorrelation)
- Useful phylogeny-derived quantities such as relative fitness are not directly supported within the software

## Evidence

- [methods] plasticity analysis is defined for each leaf: "PhyloVision enables users to quantify the plasticity of a particular categorical metadata for each leaf on the tree"
- [methods] breadth-first traversal over internal nodes: "PhyloVision clusters cells on a phylogeny by performing a breadth-first search over internal nodes"
- [methods] tree pre-processing with ape: "collapsed using the collapse.singles function in the R package ape"
- [readme] tree structure requirement for lineage metrics: [evidence span withheld — anchor status: external_doc]
