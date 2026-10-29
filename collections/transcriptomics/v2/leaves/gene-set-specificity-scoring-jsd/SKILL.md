---
name: gene-set-specificity-scoring-jsd
description: Use when you already have per-cell pathway activity weights (e.g. from
  run.rwr on an MCA co-embedding) and spot/cell group annotations, and need a numeric
  JSD-based score of how specific each gene set is to a cluster or anatomical annotation.
when_to_use_negative:
- Data are targeted gene panels with limited numbers of genes — the article states
  GSDensity is 'not suitable for targeted gene data having limited numbers of genes'
- Gene sets have not passed the coordination test (compute.kld adjusted p < 0.05);
  the article recommends downstream analyses only on coordinated gene sets
- No cell/cluster or spatial annotations exist to define cell_group, making a JSD
  specificity score undefined
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: GSDensity
  role: Provides compute.spec for JSD-based cluster-wise specificity scoring and the
    surrounding workflow functions (compute.mca, compute.kld, run.rwr.list, compute.spatial.kld.df)
  repo: https://github.com/qingnanl/gsdensity
- name: CelliD
  role: Underlying method whose MCA implementation GSDensity uses to co-embed cells
    and genes before activity scoring
  repo: https://bioconductor.org/packages/devel/bioc/html/CelliD.html
- name: Seurat
  role: Hosts the stxBrain anterior1 Visium spatial object and its spot metadata/annotations
    used as cell_group input to compute.spec
- name: msigdbr
  role: Supplies the GO Biological Process gene set collection (e.g. 727 GO terms
    in the mouse forebrain analysis) evaluated for specificity
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-44206-x
    title: Pathway centric analysis for single-cell RNA-seq and spatial transcriptomics
      data with GSDensity
  source_content_hash: sha256:9409117044e3905b2fd75c1dd0611984b9872b2795f25ac8980efe15be0e163d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qingnanl/gsdensity
  tools:
  - CelliD
  - GSDensity
  - Seurat
  - SeuratData
  - msigdbr
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# Gene-set-specificity-scoring-jsd

## Summary

Quantify how cluster-restricted each coordinated gene set is using GSDensity's compute.spec, a Jensen-Shannon divergence based specificity score computed on pathway activity labels per cell group; used alongside spatial relevance to find high-order spatial organization that cluster-centric analysis misses.

## When to use

Use when you already have per-cell pathway activity weights (e.g. from run.rwr on an MCA co-embedding) and spot/cell group annotations, and need a numeric JSD-based score of how specific each gene set is to a cluster or anatomical annotation.

## When NOT to use

- Data are targeted gene panels with limited numbers of genes — the article states GSDensity is 'not suitable for targeted gene data having limited numbers of genes'
- Gene sets have not passed the coordination test (compute.kld adjusted p < 0.05); the article recommends downstream analyses only on coordinated gene sets
- No cell/cluster or spatial annotations exist to define cell_group, making a JSD specificity score undefined

## Inputs

- Per-cell pathway activity scores (label propagation probabilities from run.rwr/run.rwr.list) for coordinated gene sets passing compute.kld adjusted p < 0.05
- Cell or spot metadata with group/cluster annotations (e.g. Visium spot labels from the stxBrain anterior1 SeuratData object)
- MCA co-embedding and nearest-neighbor cell-gene graph edges from compute.mca and compute.nn.edges

## Outputs

- Per-gene-set Jensen-Shannon divergence specificity scores (one value per GO BP term per annotation group)
- Joint scatter plot of spatial relevance vs cluster-wise specificity highlighting high-specificity and high-relevance/low-specificity terms (e.g. visium_anterior1_joint_plot.png)

## How to apply

First obtain pathway activity propagation probabilities for each coordinated gene set by running run.rwr.list on the nearest-neighbor cell-gene graph (compute.nn.edges with nn.use = 300, default restart = 0.75) built from the compute.mca co-embedding. Assemble a cell-by-gene-set data frame of activity values plus a metadata column of cell/spot labels (e.g. Visium spot annotations), then call compute.spec(cell_df, metadata, cell_group) which scores each gene set's specificity to groups using Jensen-Shannon divergence. In the stxBrain anterior1 Visium workflow, these specificity scores are jointly plotted against spatial relevance from compute.spatial.kld.df (pathway-activity-weighted vs equal-weighted 2D KDE with permutation-based nulls, adjusted p < 0.05) to highlight high-specificity terms versus high-relevance/low-specificity terms such as 'positive regulation of cellular response to insulin stimulus' and 'fatty acid oxidation'. The joint scatter reveals GO BP terms whose spatial organization spans multiple transcriptomic clusters and is therefore undetectable by cluster-centric analysis alone.

## Related tools

- **GSDensity** (Provides compute.spec for JSD-based cluster-wise specificity scoring and the surrounding workflow functions (compute.mca, compute.kld, run.rwr.list, compute.spatial.kld.df)) — https://github.com/qingnanl/gsdensity
- **CelliD** (Underlying method whose MCA implementation GSDensity uses to co-embed cells and genes before activity scoring) — https://bioconductor.org/packages/devel/bioc/html/CelliD.html
- **Seurat** (Hosts the stxBrain anterior1 Visium spatial object and its spot metadata/annotations used as cell_group input to compute.spec)
- **msigdbr** (Supplies the GO Biological Process gene set collection (e.g. 727 GO terms in the mouse forebrain analysis) evaluated for specificity)

## Examples

```
cl <- compute.spec(cell_df, metadata, cell_group)  # after cv_list <- run.rwr.list(el, gene_set_list, cells, restart = 0.75)
```

## Evaluation signals

- Joint plot reproduces the published spatial vignette structure: coordinated GO BP terms plotted as spatial relevance vs JSD specificity with high-specificity and high-relevance/low-specificity terms highlighted
- All scored gene sets previously passed the coordination filter (compute.kld adjusted p < 0.05 with n.times permutation replicates)
- Spatial relevance of highlighted terms is supported by significant permutation-based KLD p-values (adjusted p < 0.05) from compute.spatial.kld.df
- High-relevance/low-specificity terms (e.g. 'positive regulation of cellular response to insulin stimulus', 'fatty acid oxidation') map to caudal/ventral regions as in the published finding
- Specificity scores are finite, non-negative JSD values, one per gene set per annotation group in cell_df

## Limitations

- MCA embedding step is memory-hungry: about 25 Gb for a 40,000-cell dataset (hcabm40k example)
- GSDensity (and hence this specificity scoring) applies to spatial whole-transcriptomic data but not targeted gene data; imputation of unmeasured genes is suggested but not implemented
- Neighborhood size (nn.use) is an important parameter when constructing the cell-gene graph and influences downstream activity and specificity scores

## Evidence

- [methods] compute.spec is a JSD-based method for cell.type specificity analysis.: [evidence span withheld — anchor status: not_found]
- [methods] compute.spec call signature on cell_df, metadata, cell_group: "compute.spec(cell_df, metadata, cell_group)"
- [readme] README motivation of co-embedding and density-based gene set evaluation: "The core idea of gsdensity is that we use multiple correspondence analsysis (MCA) to co-embed cells and genes (thanks to the CelliD package) and then investigate the 'density' of the gene sets of"
- [abstract] Article statement that spatially relevant pathways can follow high-order organizational patterns: "GSDensity can identify spatially relevant pathways in mouse brains and human tumors including those following high-order organizational patterns in the ST data"
