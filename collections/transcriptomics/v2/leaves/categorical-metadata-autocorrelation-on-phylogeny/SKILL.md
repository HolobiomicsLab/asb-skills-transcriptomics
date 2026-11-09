---
name: categorical-metadata-autocorrelation-on-phylogeny
description: Use when you have a rooted single-cell phylogeny (e.g., a Cassiopeia-reconstructed
  Newick tree) plus cell-level categorical metadata (e.g., sampleID, tissue site)
  and want a per-cell numeric score of how homogeneous or mixed that category is along
  each cell's ancestral path.
when_to_use_negative:
- The metadata of interest is continuous/numeric rather than categorical — for continuous
  signature scores use PhyloVision's phylogenetic autocorrelation analysis with Geary's
  C instead
- The goal is quantifying relative fitness from the phylogeny — the paper states such
  approaches 'are not directly supported within our software'
- You need joint interpretation of gene expression variation rather than metadata
  plasticity — use Hotspot module identification and hypergeometric module-signature
  overlap instead
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0194
tools:
- name: PhyloVision
  role: 'Implements the plasticity analysis: runs Fitch-Hartigan small parsimony per
    categorical metadata column and adds per-cell plasticity scores to the interactive
    report'
  repo: https://github.com/YosefLab/VISION
- name: ape
  role: Provides the collapse.singles function used to collapse unifurcations in the
    input phylogeny before plasticity computation
- name: Cassiopeia
  role: Reconstructs the single-cell phylogenies that serve as input trees for the
    plasticity analysis
- name: R
  role: Language in which PhyloVision and the plasticity pipeline are implemented;
    note the R/Python interoperability caveat when combining with Python tools
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100200
    title: Interactive, integrated analysis of single-cell transcriptomic and phylogenetic
      data with PhyloVision
  source_content_hash: sha256:97ed204f43dd19d0dacc14d82ad29586cb3098e20401fbb3c0e020fd66c258d4
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
  - http://edamontology.org/topic_0194
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# categorical-metadata-autocorrelation-on-phylogeny

## Summary

PhyloVision's single-cell plasticity analysis quantifies, for each leaf of a cell phylogeny, how plastic the ancestral neighborhoods are with respect to a user-provided categorical metadata variable. It converts small-parsimony (Fitch-Hartigan) subtree scores into a per-cell numeric index, exemplified by the plasticity index on sampleID corresponding to scMetRate in the metastatic lung adenocarcinoma case study.

## When to use

Use when you have a rooted single-cell phylogeny (e.g., a Cassiopeia-reconstructed Newick tree) plus cell-level categorical metadata (e.g., sampleID, tissue site) and want a per-cell numeric score of how homogeneous or mixed that category is along each cell's ancestral path.

## When NOT to use

- The metadata of interest is continuous/numeric rather than categorical — for continuous signature scores use PhyloVision's phylogenetic autocorrelation analysis with Geary's C instead
- The goal is quantifying relative fitness from the phylogeny — the paper states such approaches 'are not directly supported within our software'
- You need joint interpretation of gene expression variation rather than metadata plasticity — use Hotspot module identification and hypergeometric module-signature overlap instead

## Inputs

- Rooted single-cell phylogeny in Newick format (e.g., Cassiopeia-Hybrid CP004 tree with unifurcations collapsed via ape::collapse.singles), leaves named by cell identifiers
- Cell-level categorical metadata table (e.g., downloaded from GEO series GSE161363, dataset GSM4905334), one row per cell with categorical columns such as sampleID

## Outputs

- plasticity_scores.csv with one row per cell in the phylogeny and one column per categorical metadata variable
- Per-cell numeric meta-data entries added to the PhyloVision report (e.g., a plasticity index on sampleID corresponding to scMetRate)

## How to apply

Load the reconstructed tumor phylogeny — in the reference case, the CP004 phylogeny inferred with Cassiopeia-Hybrid and collapsed with ape's collapse.singles to remove unifurcations — together with the cell-level categorical metadata table (GEO series GSE161363, dataset GSM4905334). For each categorical metadata column, run the Fitch-Hartigan small-parsimony algorithm (Fitch 1971; Hartigan 1973) on every subtree defined by each internal node of the tree, then normalize each subtree's parsimony score by the number of edges in that subtree. For each leaf, perform a depth-first traversal from the root and average the normalized parsimony scores of all internal nodes along that path to obtain that cell's plasticity score for the category. Assemble the per-cell plasticity vectors (one per categorical variable) and write them to plasticity_scores.csv with one row per cell and one column per categorical variable; these numeric entries are then added as per-cell meta-data (e.g., a plasticity index on sampleID) for downstream analysis such as scMetRate comparison.

## Related tools

- **PhyloVision** (Implements the plasticity analysis: runs Fitch-Hartigan small parsimony per categorical metadata column and adds per-cell plasticity scores to the interactive report) — https://github.com/YosefLab/VISION
- **ape** (Provides the collapse.singles function used to collapse unifurcations in the input phylogeny before plasticity computation)
- **Cassiopeia** (Reconstructs the single-cell phylogenies that serve as input trees for the plasticity analysis)
- **R** (Language in which PhyloVision and the plasticity pipeline are implemented; note the R/Python interoperability caveat when combining with Python tools)

## Examples

```
collapsed_tree <- ape::collapse.singles(tree)
```

## Evaluation signals

- plasticity_scores.csv exists with exactly one row per cell present in the phylogeny and one column per categorical metadata variable
- Cells whose ancestral neighborhoods are entirely homogeneous for a category receive near-minimal scores (low normalized parsimony)
- Cells spanning mixed ancestral states along their root-to-leaf path receive higher plasticity scores than homogeneous-path cells
- Each subtree parsimony score is edge-normalized, so scores are comparable across subtrees of different sizes (e.g., deep vs. shallow internal nodes)
- The added per-cell plasticity entry (e.g., plasticity index on sampleID) aligns cell-for-cell with the phylogeny's leaf labels

## Limitations

- PhyloVision is implemented in R, which the authors note 'may provide challenges for utilizing separate software available in Python, like Cassiopeia'
- Plasticity scores depend on the quality of the reconstructed phylogeny; unifurcations must first be collapsed (ape's collapse.singles) before computation
- The tool does not directly support quantifying other phylogeny-derived cellular characteristics such as relative fitness
- Hypotheses generated by the metastasis analysis (including plasticity-related observations) lack experimental validation, per the discussion

## Evidence

- [methods] PhyloVision enables users to quantify the plasticity of a particular categorical metadata for each leaf on the tree: "PhyloVision enables users to quantify the plasticity of a particular categorical metadata for each leaf on the tree"
- [methods] Unifurcation collapsing with ape: "collapsed using the collapse.singles function in the R package ape"
- [methods] Source of the cell-level metadata table: "Meta data corresponding to each cell was downloaded from NCBI GEO, series GSE161363 and dataset GSM4905334"
- [results] PhyloVision as the analysis pipeline containing this step: "PhyloVision is an integrated pipeline for interactive analysis of single-cell expression and lineage profiles"
