---
name: snrna-seq-data-loading-and-object-construction
description: Use when starting a scRank analysis from raw snRNA-seq/scRNA-seq count
  matrices plus cell-type metadata, and you need a ScRank S4 object encoding species,
  cell-type labels, and a known drug-target gene before network reconstruction.
when_to_use_negative:
- Data is bulk RNA-seq or lacks per-cell cell-type labels — scRank needs cell-type-resolved
  single-cell expression to build cell-type-specific GRNs
- The drug target gene is unknown or multi-target without tailored methodology — scRank
  models perturbation of a single target gene node, and multi-target extension is
  only preliminary (Figure S17)
- A ranked drug-responsive cell type is already available and the task is downstream
  modularization or visualization (use init_mod/plot_net/plot_dim instead)
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scRank
  role: Provides the CreateScRank and subsetScRank functions that wrap the Seurat
    object into the scRank S4 object used throughout the pipeline
  repo: https://github.com/ZJUFanLab/scRank
- name: Seurat
  role: Holds the count matrix and cell-type metadata as the input container passed
    to CreateScRank
- name: rTensor
  role: Dependent package that must be installed before scRank object construction
  repo: https://github.com/rikenbit/rTensor
- name: devtools
  role: Used to install scRank and rTensor from GitHub prior to object construction
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.xcrm.2024.101568
    title: scRank infers drug-responsive cell types from untreated scRNA-seq data
      using a target-perturbed gene regulatory network
  source_content_hash: sha256:129fac229fc55fe5f16d21f890c11166f001a035576757895bae975b6a12425d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ZJUFanLab/scRank
  tools:
  - R
  - Seurat
  - devtools
  - rTensor
  - scRank
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# snrna-seq-data-loading-and-object-construction

## Summary

Load single-nucleus RNA-seq count matrices with cell-type metadata (e.g., the untreated MDD dorsolateral prefrontal cortex dataset GSE144136) into a Seurat object and wrap it in an scRank S4 object via CreateScRank, specifying species, cell-type label column, drug target gene, and perturbation type. This is the mandatory entry point for the scRank target-perturbed GRN pipeline.

## When to use

Use when starting a scRank analysis from raw snRNA-seq/scRNA-seq count matrices plus cell-type metadata, and you need a ScRank S4 object encoding species, cell-type labels, and a known drug-target gene before network reconstruction.

## When NOT to use

- Data is bulk RNA-seq or lacks per-cell cell-type labels — scRank needs cell-type-resolved single-cell expression to build cell-type-specific GRNs
- The drug target gene is unknown or multi-target without tailored methodology — scRank models perturbation of a single target gene node, and multi-target extension is only preliminary (Figure S17)
- A ranked drug-responsive cell type is already available and the task is downstream modularization or visualization (use init_mod/plot_net/plot_dim instead)

## Inputs

- GEO-deposited snRNA-seq count matrix (genes x barcodes), e.g. GSE144136 untreated MDD dorsolateral prefrontal cortex
- Cell-type metadata table with a label column (e.g. excitatory cluster Ex_9, Inhib_5)
- Seurat object with cell-type annotations
- Drug-target gene symbol (e.g. SLC6A4) and perturbation type ('antagonist'/'agonist')

## Outputs

- scRank S4 object created by CreateScRank, ready for Constr_net and rank_celltype
- Optionally a subsetScRank()-derived subset object

## How to apply

Download the count matrix and cell-type metadata (e.g., from GEO accession GSE144136 for the untreated MDD dorsolateral prefrontal cortex) and load them into a Seurat object with the cell-type label column intact. Install scRank's dependencies devtools and rTensor, then install scRank from GitHub. Call CreateScRank(input=<Seurat object>, species='human', cell_type=<label column>, target=<drug-target gene>, type='antagonist'); for fluoxetine the target is SLC6A4 and type is 'antagonist' since the drug acts on the serotonin transporter. The resulting S4 object is the required input for downstream Constr_net network reconstruction and rank_celltype scoring; it is not yet a ranking — verify the object holds expression and cell-type annotations before proceeding. Use subsetScRank() (per the README issue #14 note) if a subset of the object is needed.

## Related tools

- **scRank** (Provides the CreateScRank and subsetScRank functions that wrap the Seurat object into the scRank S4 object used throughout the pipeline) — https://github.com/ZJUFanLab/scRank
- **Seurat** (Holds the count matrix and cell-type metadata as the input container passed to CreateScRank)
- **rTensor** (Dependent package that must be installed before scRank object construction) — https://github.com/rikenbit/rTensor
- **devtools** (Used to install scRank and rTensor from GitHub prior to object construction)

## Examples

```
devtools::install_github("rikenbit/rTensor"); devtools::install_github("ZJUFanLab/scRank"); obj <- CreateScRank(input=seurat_obj, species='human', cell_type='celltype', target='SLC6A4', type='antagonist')
```

## Evaluation signals

- CreateScRank runs without error and returns an S4 object whose slot structure matches the scRank class (subsequent Constr_net calls succeed on it)
- The target gene symbol (e.g. SLC6A4) is present in the object's gene/row names; mismatched gene IDs fail downstream network reconstruction
- species and cell_type arguments match the actual organism and metadata column of the loaded dataset (human vs. mouse label columns differ across datasets like GSE110894 vs. GSE144136)
- type='antagonist' is set for inhibitor-like drugs such as fluoxetine acting on SLC6A4; agonist vs. antagonist choice changes the tpGRN construction direction
- After pipeline completion, Ex_9 should be the top-ranked cell type in obj@cell_type_rank, confirming the object was built from the correct untreated MDD data

## Limitations

- scRank primarily focuses on drugs acting as inhibitors; agonist modeling uses a simplifying assumption (setting out-edge weights of the target gene to their maximum value) and is preliminary
- Multi-target perturbation requires specifically tailored methodologies not yet developed
- The S4 object alone does not produce any ranking — it is only the entry point before Constr_net and rank_celltype

## Evidence
