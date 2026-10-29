---
name: cell-type-drug-response-ranking
description: Use when you need to identify which cell type in an untreated disease
  scRNA-seq dataset is most responsive to a drug with a known target gene, without
  paired treated/untreated samples.
when_to_use_negative:
- Only paired pre/post-treatment data with no known drug target gene are available
  — DEG-count/separability methods like Augur, Wilcoxon, MAST, DESeq2 are more appropriate
  since scRank relies on prior target knowledge
- The drug has multiple targets — multi-target perturbation requires specifically
  tailored methodologies not yet developed in scRank
- You need validated agonist-drug modeling — scRank's agonist mode is a simplifying
  assumption (max-weight out-edges) with only preliminary testing due to limited agonist
  single-cell data
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: scRank
  role: Core R package implementing CreateScRank, Constr_net, rank_celltype, init_mod,
    plot_dim, plot_net, scRank_GSEA for tpGRN construction and drug-responsive cell-type
    ranking
  repo: https://github.com/ZJUFanLab/scRank
- name: rTensor
  role: Dependency installed for scRank's tensor-based GRN inference in Constr_net
  repo: https://github.com/rikenbit/rTensor
- name: devtools
  role: Used to install scRank from GitHub via devtools::install_github
  repo: https://github.com/ZJUFanLab/scRank
- name: Seurat
  role: Provides the input Seurat object with expression matrix and cell-type labels
    consumed by CreateScRank
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_007
  - task_002
  source_papers:
  - doi: 10.1016/j.xcrm.2024.101568
    title: scRank infers drug-responsive cell types from untreated scRNA-seq data
      using a target-perturbed gene regulatory network
  source_content_hash: sha256:34060f42014d3df3b3f55b0e21f038623f7e6dfad4474dc691ec97ddabfc44b0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ZJUFanLab/scRank
  tools:
  - Augur
  - R
  - Seurat
  - devtools
  - rTensor
  - scRank
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# cell-type-drug-response-ranking

## Summary

Rank drug-responsive cell types from untreated scRNA-seq data by building a target-perturbed gene regulatory network (tpGRN) with scRank and scoring in silico drug perturbation, combining global manifold-alignment distortion and local 2-hop network diffusion from the drug target gene.

## When to use

Use when you need to identify which cell type in an untreated disease scRNA-seq dataset is most responsive to a drug with a known target gene, without paired treated/untreated samples.

## When NOT to use

- Only paired pre/post-treatment data with no known drug target gene are available — DEG-count/separability methods like Augur, Wilcoxon, MAST, DESeq2 are more appropriate since scRank relies on prior target knowledge
- The drug has multiple targets — multi-target perturbation requires specifically tailored methodologies not yet developed in scRank
- You need validated agonist-drug modeling — scRank's agonist mode is a simplifying assumption (max-weight out-edges) with only preliminary testing due to limited agonist single-cell data

## Inputs

- Untreated-condition scRNA-seq count matrix or Seurat object with cell-type annotations
- Known drug target gene symbol (e.g. Brd4, SLC6A4) and drug type ('antagonist'/'agonist')
- Species identifier (e.g. 'human')
- Optional cell-type label column name for the Seurat object

## Outputs

- Cell-type perturbation-score ranking stored in obj@cell_type_rank (exportable as cell_type_rank.csv)
- Target-perturbed GRN per cell type (tpGRN)
- Plots: UMAP/dimension-reduction ranking (plot_dim), network/module heatmaps (plot_net), GSEA-based drug-function analysis (scRank_GSEA/plot_drug_function)

## How to apply

Start by creating an scRank S4 object with CreateScRank, supplying the Seurat/expression object, species, cell-type label column, the drug target gene via the `target` argument, and `type` (e.g. 'antagonist' for inhibitors; agonists are modeled by setting out-edge weights of the target gene to their maximum value). Reconstruct cell-type-specific gene regulatory networks from expression profiles with Constr_net (which integrates the Python module 'tensorly'). Call rank_celltype to form the tpGRN by deleting the target gene node's outgoing edges, quantify global distortion via manifold-alignment node distances, add a local effect via 2-hop neighborhood network diffusion from the target gene, and combine both into a per-cell-type perturbation score. Read the ranking from obj@cell_type_rank. Optionally modularize the target-gene subnetwork with init_mod and visualize with plot_net (heatmap of module activity per cell type) or plot_dim (ranking in dimension-reduction space); use scRank_GSEA()/plot_drug_function() to assess drug functional impact in the top-ranked cell type.

## Related tools

- **scRank** (Core R package implementing CreateScRank, Constr_net, rank_celltype, init_mod, plot_dim, plot_net, scRank_GSEA for tpGRN construction and drug-responsive cell-type ranking) — https://github.com/ZJUFanLab/scRank
- **rTensor** (Dependency installed for scRank's tensor-based GRN inference in Constr_net) — https://github.com/rikenbit/rTensor
- **devtools** (Used to install scRank from GitHub via devtools::install_github) — https://github.com/ZJUFanLab/scRank
- **Seurat** (Provides the input Seurat object with expression matrix and cell-type labels consumed by CreateScRank)

## Examples

```
devtools::install_github("ZJUFanLab/scRank"); obj <- CreateScRank(input=seurat_obj, species='human', cell_type='label', target='SLC6A4', type='antagonist'); obj <- Constr_net(obj); obj <- rank_celltype(obj); obj@cell_type_rank
```

## Evaluation signals

- On SERGIO-simulated ground-truth benchmarks (gradient, binary, and cell-type-specific module scenarios with 4-6 cell types and 100 genes), the designated target cell type is ranked first in all three scenarios via obj@cell_type_rank
- On the five real drug-perturbation datasets (nine known target cell types), scRank run on untreated data only ranks all nine known target cell types at the top, outperforming paired-data baselines (Augur, Wilcoxon, MAST, bimod, DESeq2) in averaged rank
- The top-ranked cell type shows abnormally high activity in the drug-target-gene module (e.g. module 2 containing SLC6A4 in Ex_9 vs Inhib_5 in the MDD fluoxetine case) when compared with init_mod + plot_net heatmaps
- obj@cell_type_rank is populated and ordered by combined global + local perturbation score after rank_celltype completes
- Application to known cases reproduces literature-consistent targets: BET inhibitor target in GSE110894 leukaemic cells, Ex_9 excitatory neurons for fluoxetine (SLC6A4) in MDD DLPFC

## Limitations

- scRank primarily focuses on inhibitor drugs; agonist modeling is a simplifying assumption (setting out-edge weights of the target gene to maximum value) and only preliminarily tested
- Multi-target drugs require specifically tailored methodologies not yet developed; only preliminary results exist (Figure S17)
- Performance of alternative tools (scDEAL, beyondcell) and scRank comparisons can be influenced by the quality and imbalance of bulk reference datasets
- Validation of tpGRN simulation against GRNs from treated cells appears only in supplementary material (Figure S14)

## Evidence

- [abstract] tpGRN ranking workflow: "employs a target-perturbed gene regulatory network to rank drug-responsive cell populations via in silico drug perturbations using untreated single-cell transcriptomic data"
- [methods] pipeline functions: "the second step is to estimate the extent of the in silico drug perturbation for GRNs in each cell type using `rank_celltype` function"
- [methods] installation: "install dependent packages `devtools` and [`rTensor`](https://github.com/rikenbit/rTensor)"
- [discussion] global and local effects: "we model drug perturbation in the tpGRN as having both global and local effects. Specifically, the perturbed target gene can cause global distortion in each gene node"
- [discussion] agonist assumption: "we set the weight on edges emanating from the target gene node to their maximum possible value, resulting in the tpGRN"
- [readme] README install command: "devtools::install_github("ZJUFanLab/scRank")"
