---
name: tsne-visualization-single-cell
description: Use when a normalized c-CSN-derived CNDM (m genes x n cells, e.g. >20,000
  Tabula Muris droplet cells) needs dimension reduction and 2D visualization to confirm
  that cells segregate by known origin or cluster structure.
when_to_use_negative:
- Input is a raw gene expression matrix (GEM) rather than a c-CSN-transformed CNDM;
  use standard scRNA-seq visualization workflows instead
- Goal is to construct the conditional cell-specific network itself rather than visualize
  an already-computed CNDM
- Goal is trajectory directionality or differentiation potency estimation, which requires
  NFE computation rather than plain t-SNE embedding
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0092
tools:
- name: c-CSN
  role: Produces the CNDM input that t-SNE is applied to; CNDM must be normalized
    per the c-CSN methods for compatibility with downstream pipelines
  repo: https://github.com/LinLi-0909/c-CSN
- name: Seurat
  role: Used to run dimension reduction (t-SNE) on the normalized CNDM and to combine
    visualization with clustering for ARI evaluation
- name: t-SNE
  role: Nonlinear dimension reduction method applied to the normalized CNDM to produce
    the 2D cell map
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2020.05.005
    title: 'c-CSN: Single-Cell RNA Sequencing Data Analysis by Conditional Cell-Specific
      Network'
  source_content_hash: sha256:dae8222eff5bebcfa2730e87cca9f8bd8dbd077fed43e1422102b5e0a45dac8e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LinLi-0909/c-CSN
  tools:
  - Seurat
  - c-CSN
  - t-SNE
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# tsne-visualization-single-cell

## Summary

Apply t-SNE (optionally after PCA) to a c-CSN-derived conditional network degree matrix (CNDM) to visualize single-cell structure, enabling assessment of tissue- or cell-type separation compatible with standard scRNA-seq pipelines.

## When to use

Use when a normalized c-CSN-derived CNDM (m genes x n cells, e.g. >20,000 Tabula Muris droplet cells) needs dimension reduction and 2D visualization to confirm that cells segregate by known origin or cluster structure.

## When NOT to use

- Input is a raw gene expression matrix (GEM) rather than a c-CSN-transformed CNDM; use standard scRNA-seq visualization workflows instead
- Goal is to construct the conditional cell-specific network itself rather than visualize an already-computed CNDM
- Goal is trajectory directionality or differentiation potency estimation, which requires NFE computation rather than plain t-SNE embedding

## Inputs

- c-CSN-derived conditional network degree matrix (CNDM), m genes x n cells
- Cell-origin annotation labels (e.g. tissue of origin for Tabula Muris droplet1 cells)

## Outputs

- 2D t-SNE embedding of cells (coordinates per cell)
- t-SNE scatter plot colored by cell annotations showing cluster/tissue separation

## How to apply

First normalize the CNDM with the c-CSN methods so it is compatible with standard scRNA-seq downstream pipelines. Run PCA and/or t-SNE on the normalized CNDM, as described in the article's Network Analysis Of C-Csn > Dimension Reduction step, to obtain a 2D embedding of all cells. Visualize the embedding colored by cell-origin annotations (e.g. bladder, trachea, and spleen from the Tabula Muris droplet1 dataset, GSE109774) and inspect that the three tissue groups separate clearly. Optionally use the Seurat package to combine dimension reduction with subsequent clustering, as was done to reproduce ARI = 0.73 against ground-truth labels. Rationale: the article shows that t-SNE on the CNDM behaves like t-SNE on a gene expression matrix because most scRNA-seq methods designed for gene expression matrices are also applicable to c-CSN-transformed degree matrices.

## Related tools

- **c-CSN** (Produces the CNDM input that t-SNE is applied to; CNDM must be normalized per the c-CSN methods for compatibility with downstream pipelines) — https://github.com/LinLi-0909/c-CSN
- **Seurat** (Used to run dimension reduction (t-SNE) on the normalized CNDM and to combine visualization with clustering for ARI evaluation)
- **t-SNE** (Nonlinear dimension reduction method applied to the normalized CNDM to produce the 2D cell map)

## Examples

```
R: library(Seurat); se <- CreateSeuratObject(cndm); se <- RunPCA(se); se <- RunTSNE(se, dims=1:2); DimPlot(se, group.by='tissue')
```

## Evaluation signals

- t-SNE map shows clear separation of the three dominant cell groups corresponding to bladder, trachea, and spleen origins on the Tabula Muris droplet1 dataset
- Seurat clustering on the dimension-reduced CNDM reproduces the reported ARI = 0.73 against ground-truth cell origins (tolerance ~0.65-0.80)
- CNDM-based clustering/dimension reduction yields higher ARI than the same analysis on the original gene expression matrix (GEM), per the article's finding that CNDM performs better than GEM on all datasets
- CNDM dimensions are consistent: m genes x n cells, with n > 20,000 for the Tabula Muris droplet1 dataset before embedding

## Limitations

- t-SNE on the CNDM assumes the c-CSN transformation and its G-fold computational cost increase have already been performed; c-CSN provides no parallel computation scheme, so CNDM construction time can be large
- t-SNE visualization alone does not resolve differentiation trajectory direction; that requires CCSN-based network flow entropy (NFE) analysis
- t-SNE embeddings are stochastic and non-deterministic, so exact point positions should not be expected to reproduce across runs; separation quality and downstream ARI are the meaningful checks

## Evidence

- [methods] t-SNE applied to normalized CNDM as described dimension reduction step: "We used PCA [35] and t-SNE [36] which respectively represent linear and nonlinear methods, to perform dimension reduction"
- [abstract] Compatibility of c-CSN-transformed degree matrices with standard scRNA-seq analyses: "Most existing scRNA-seq methods designed for gene expression matrices are also applicable to c-CSN-transformed degree matrices."
- [methods] Tabula Muris droplet1 dataset used for Seurat-based t-SNE/clustering and ARI evaluation: "The Tabula Muris droplet1 dataset [47] comprising more than 20,000 cells from three tissues (bladder, trachea, and spleen)"
- [other] t-SNE + Seurat pipeline reproduces ARI = 0.73, confirming correct application: [evidence span withheld — anchor status: not_found]
- [methods] CNDM outperforms gene expression matrix in downstream clustering, supporting the choice of CNDM as t-SNE input: "CNDM performs obviously better than GEM on all datasets"
