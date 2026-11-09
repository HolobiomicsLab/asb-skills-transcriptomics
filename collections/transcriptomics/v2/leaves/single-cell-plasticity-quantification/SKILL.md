---
name: single-cell-plasticity-quantification
description: Use when you have a reconstructed single-cell phylogeny (cells as leaves)
  plus per-cell categorical metadata (e.g., clone, site, or state labels from GEO
  metadata) and want a per-cell numeric score of how plastic (mixed) the ancestral
  context of each category is along the lineage.
when_to_use_negative:
- The metadata of interest is continuous/numeric rather than categorical — Fitch-Hartigan
  small parsimony requires discrete character states
- The goal is to quantify relative fitness from the phylogeny — the article explicitly
  notes such approaches 'are not directly supported within our software'
- The tree leaves do not correspond one-to-one with metadata rows (e.g., mixed species
  or unmatched cell barcodes), since scores are assigned per leaf via sampleID
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0084
tools:
- name: PhyloVision
  role: Implements the single-cell plasticity analysis in its phyloAnalyze step, attaching
    plasticity scores as per-cell numeric meta-data
  repo: https://github.com/YosefLab/VISION
- name: ape
  role: R package whose collapse.singles function collapses unifurcations in the Cassiopeia-Hybrid
    tree before plasticity computation
- name: Cassiopeia
  role: Reconstructs the single-cell phylogenies (e.g., the CP004 tumor phylogeny)
    that serve as input for plasticity scoring
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100200
    title: Interactive, integrated analysis of single-cell transcriptomic and phylogenetic
      data with PhyloVision
  source_content_hash: sha256:28e7b6b9185bfab4f390a6bf289de108c1edab46903165c787b66d3b4f08a8e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/VISION
  tools:
  - Cassiopeia
  - PhyloVision
  - R
  - ape
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# single-cell-plasticity-quantification

## Summary

Quantifies per-cell 'plasticity' of user-provided categorical meta-data on a single-cell phylogeny, as implemented in the PhyloVision phyloAnalyze pipeline: Fitch-Hartigan small-parsimony scores computed on subtrees, edge-normalized, and averaged along root-to-leaf paths.

## When to use

Use when you have a reconstructed single-cell phylogeny (cells as leaves) plus per-cell categorical metadata (e.g., clone, site, or state labels from GEO metadata) and want a per-cell numeric score of how plastic (mixed) the ancestral context of each category is along the lineage.

## When NOT to use

- The metadata of interest is continuous/numeric rather than categorical — Fitch-Hartigan small parsimony requires discrete character states
- The goal is to quantify relative fitness from the phylogeny — the article explicitly notes such approaches 'are not directly supported within our software'
- The tree leaves do not correspond one-to-one with metadata rows (e.g., mixed species or unmatched cell barcodes), since scores are assigned per leaf via sampleID

## Inputs

- Rooted single-cell phylogeny with cells as leaves (e.g., Newick tree reconstructed by Cassiopeia-Hybrid, unifurcations collapsed)
- Cell-level categorical metadata table (one row per cell barcode, one column per categorical variable)
- Mapping between tree leaf names and metadata row keys (e.g., sampleID)

## Outputs

- plasticity_scores.csv: one row per cell, one column per categorical metadata variable, containing per-cell plasticity scores
- Per-cell numeric meta-data entries added to the PhyloVision report (a plasticity index per categorical variable)

## How to apply

Load the phylogeny (e.g., CP004 tumor phylogeny reconstructed with Cassiopeia-Hybrid, unifurcations collapsed with ape's collapse.singles) and the cell-level categorical metadata table (e.g., downloaded from GEO series GSE161363, dataset GSM4905334). For each categorical metadata column, run the Fitch-Hartigan small-parsimony algorithm (Fitch 1971; Hartigan 1973) on every subtree defined by each internal node of the tree, then normalize each subtree's parsimony score by the number of edges in that subtree. For each leaf, perform a depth-first traversal from the root to the leaf and average the normalized parsimony scores of all internal nodes along that path to obtain the single-cell plasticity score. Assemble per-cell plasticity vectors (one per categorical variable) and write them out as plasticity_scores.csv with one row per cell and one column per categorical variable; in PhyloVision these are added as per-cell numeric meta-data entries (e.g., a plasticity index on sampleID) that can be displayed in the report.

## Related tools

- **PhyloVision** (Implements the single-cell plasticity analysis in its phyloAnalyze step, attaching plasticity scores as per-cell numeric meta-data) — https://github.com/YosefLab/VISION
- **ape** (R package whose collapse.singles function collapses unifurcations in the Cassiopeia-Hybrid tree before plasticity computation)
- **Cassiopeia** (Reconstructs the single-cell phylogenies (e.g., the CP004 tumor phylogeny) that serve as input for plasticity scoring)

## Evaluation signals

- plasticity_scores.csv exists and has exactly one row per cell in the phylogeny and one column per categorical metadata variable
- Cells whose ancestral neighborhoods are entirely homogeneous for a category receive near-minimal scores (low parsimony)
- Cells whose root-to-leaf path spans mixed ancestral states receive higher plasticity scores than homogeneous-path cells
- Each subtree's parsimony contribution is edge-normalized, so scores are comparable across subtrees of different sizes

## Limitations

- Scores are only meaningful for categorical metadata columns; continuous traits are not handled by this Fitch-Hartigan-based procedure
- PhyloVision is implemented in R, which may create challenges when interoperating with Python tools such as Cassiopeia
- Relative fitness and some other cellular characteristics quantifiable from phylogenies are not supported by the software

## Evidence

- [methods] PhyloVision enables users to quantify the plasticity of a particular categorical metadata for each leaf on the tree: "PhyloVision enables users to quantify the plasticity of a particular categorical metadata for each leaf on the tree"
- [methods] collapse.singles in ape: "collapsed using the collapse.singles function in the R package ape"
- [methods] GEO metadata provenance: "Meta data corresponding to each cell was downloaded from NCBI GEO, series GSE161363 and dataset GSM4905334"
- [readme] Hotspot lineage metric: [evidence span withheld — anchor status: external_doc]
