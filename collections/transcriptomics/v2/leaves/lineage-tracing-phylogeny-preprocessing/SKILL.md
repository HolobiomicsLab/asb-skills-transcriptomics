---
name: lineage-tracing-phylogeny-preprocessing
description: Use when you have a single-cell phylogeny reconstructed from CRISPR-Cas9
  lineage tracing (e.g., via Cassiopeia) plus cell-level categorical meta-data (e.
when_to_use_negative:
- You need relative fitness quantification from phylogenies — the article states such
  approaches are 'not directly supported within our software'.
- The tree is not derived from lineage tracing or lacks per-cell categorical meta-data
  aligned to leaf labels; plasticity scoring requires categorical states on the phylogeny
  leaves.
- You need phylogenetic autocorrelation of continuous signature scores (Geary's C)
  or Hotspot module identification — those are separate PhyloVision pipeline components,
  not this preprocessing/plasticity skill.
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0194
- http://edamontology.org/topic_3170
tools:
- name: PhyloVision
  role: Pipeline whose phyloAnalyze step computes per-cell plasticity scores for each
    categorical meta-data column and adds them as per-cell numeric meta-data
  repo: https://github.com/YosefLab/VISION
- name: ape
  role: R package providing collapse.singles to remove unifurcations from the reconstructed
    phylogeny before analysis
- name: Cassiopeia
  role: Used to infer the single-cell phylogenies that serve as input trees (e.g.,
    Cassiopeia-Hybrid for the CP004 tumor phylogeny)
- name: R
  role: Implementation language for the PhyloVision pipeline and the Fitch-Hartigan
    plasticity computation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100200
    title: Interactive, integrated analysis of single-cell transcriptomic and phylogenetic
      data with PhyloVision
  source_content_hash: sha256:606b3a1199b2ce5c23f9f07c90c65bf195d3c73d9a9178c66f748d65f9539926
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
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# lineage-tracing-phylogeny-preprocessing

## Summary

Preprocess single-cell lineage-tracing phylogenies for downstream analysis in PhyloVision by collapsing unifurcations (ape's collapse.singles) and quantifying cell-level plasticity of categorical meta-data via Fitch-Hartigan small-parsimony scoring over subtrees. This makes tree structure and categorical states directly usable as per-cell numeric meta-data.

## When to use

Use when you have a single-cell phylogeny reconstructed from CRISPR-Cas9 lineage tracing (e.g., via Cassiopeia) plus cell-level categorical meta-data (e.g., GEO sample annotations), and you need a clean, unifurcation-free tree and per-cell scores of how plastic each categorical variable is across ancestral neighborhoods.

## When NOT to use

- You need relative fitness quantification from phylogenies — the article states such approaches are 'not directly supported within our software'.
- The tree is not derived from lineage tracing or lacks per-cell categorical meta-data aligned to leaf labels; plasticity scoring requires categorical states on the phylogeny leaves.
- You need phylogenetic autocorrelation of continuous signature scores (Geary's C) or Hotspot module identification — those are separate PhyloVision pipeline components, not this preprocessing/plasticity skill.

## Inputs

- Single-cell phylogeny reconstructed with Cassiopeia (Newick tree; unifurcations collapsed with ape's collapse.singles)
- Cell-level categorical meta-data table (e.g., from GEO series GSE161363, dataset GSM4905334), with cell identifiers matching phylogeny leaves

## Outputs

- plasticity_scores.csv with one row per cell and one column per categorical meta-data variable
- Per-cell numeric meta-data entries (e.g., a plasticity index on sampleID corresponding to scMetRate) suitable for the PhyloVision report
- Unifurcation-collapsed phylogeny ready for PhyloVision analysis and visualization

## How to apply

Start with the phylogeny reconstructed with Cassiopeia (e.g., the CP004 tumor phylogeny from the 1,127-cell metastatic lung adenocarcinoma clone) and collapse unifurcations using ape's collapse.singles in R, as done in the PhyloVision pipeline. Load the categorical meta-data table (e.g., downloaded from GEO series GSE161363, dataset GSM4905334) and align its cell identifiers with the phylogeny's leaves. For each categorical column, run the Fitch-Hartigan small-parsimony algorithm (Fitch 1971; Hartigan 1973) on every subtree defined by each internal node, then normalize each subtree's parsimony score by its number of edges. For each leaf, perform a depth-first traversal from the root and average the normalized subtree parsimony scores of all internal nodes along the root-to-leaf path to obtain the single-cell plasticity score. Assemble one plasticity vector per categorical variable and write plasticity_scores.csv with one row per cell and one column per categorical variable; these per-cell numeric entries can then be added to the report as meta-data (e.g., a plasticity index on sampleID corresponding to scMetRate).

## Related tools

- **PhyloVision** (Pipeline whose phyloAnalyze step computes per-cell plasticity scores for each categorical meta-data column and adds them as per-cell numeric meta-data) — https://github.com/YosefLab/VISION
- **ape** (R package providing collapse.singles to remove unifurcations from the reconstructed phylogeny before analysis)
- **Cassiopeia** (Used to infer the single-cell phylogenies that serve as input trees (e.g., Cassiopeia-Hybrid for the CP004 tumor phylogeny))
- **R** (Implementation language for the PhyloVision pipeline and the Fitch-Hartigan plasticity computation)

## Examples

```
tree <- ape::collapse.singles(tree)  # preprocess Cassiopeia-reconstructed phylogeny before phyloAnalyze plasticity scoring
```

## Evaluation signals

- plasticity_scores.csv exists and has exactly one row per cell in the phylogeny and one column per categorical variable
- Cells whose ancestral neighborhoods are entirely homogeneous for a category receive near-minimal scores (low parsimony)
- Cells spanning mixed ancestral states along root-to-leaf paths receive higher plasticity scores
- The input tree has no unifurcations remaining after collapse.singles, matching the PhyloVision preprocessing convention

## Limitations

- Relative fitness quantification from phylogenies is not directly supported within the software
- PhyloVision's R implementation may pose interoperability challenges with Python software such as Cassiopeia
- PhyloVision's dynamic phylogeny viewer was only tested on trees of up to ~4,000 leaves, so much larger phylogenies are untested

## Evidence

- [methods] PhyloVision enables users to quantify the plasticity of a particular categorical metadata for each leaf on the tree: "PhyloVision enables users to quantify the plasticity of a particular categorical metadata for each leaf on the tree"
- [methods] collapse.singles preprocessing: "collapsed using the collapse.singles function in the R package ape"
- [results] Cassiopeia-reconstructed input phylogenies: "reconstructed single-cell phylogenies from the original study, which were inferred with the Cassiopeia package"
- [methods] meta-data source for plasticity scoring: "Meta data corresponding to each cell was downloaded from NCBI GEO, series GSE161363 and dataset GSM4905334"
- [readme] tree metric input requirement in Hotspot/lineage context: [evidence span withheld — anchor status: external_doc]
