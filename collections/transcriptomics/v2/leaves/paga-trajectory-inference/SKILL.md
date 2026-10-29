---
name: paga-trajectory-inference
description: Use when validating that simulated scRNA-seq data (e.g., GRouNdGAN .h5ad
  output) reproduce the trajectory structure and pseudo-time ordering of an experimental
  reference such as mouse BoneMarrow hematopoiesis (Paul et al., 2015; GSE72857).
when_to_use_negative:
- The reference and simulated data cannot be jointly embedded (e.g., different gene
  spaces or unnormalized counts — GRouNdGAN outputs library-size normalized expression
  values only)
- The biological system has no expected transitional structure — PAGA/diffusion pseudotime
  is designed for trajectories such as hematopoietic differentiation, not for unrelated
  discrete cell types
- The GRouNdGAN model has not been retrained on the dataset at hand — it is a generative,
  not predictive, model, so simulated data from a different reference will not match
  the new trajectory
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scanpy
  role: Jointly embed experimental and simulated cells, run PAGA trajectory inference,
    diffusion pseudotime computation, and marker-gene plotting
  repo: scverse/scanpy-tutorials
- name: GRouNdGAN
  role: Produces the simulated single-cell dataset (--generate -> simulated.h5ad)
    whose trajectory preservation is being validated
  repo: github.com/Emad-COMBINE-lab/GRouNdGAN
- name: GRNBoost2
  role: Infers the causal GRN (top k=15 regulating TFs per gene) imposed during GRouNdGAN
    training, upstream of simulation
  repo: github.com/Emad-COMBINE-lab/GRouNdGAN
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-48516-6
    title: 'GRouNdGAN: GRN-guided simulation of single-cell RNA-seq data using causal
      generative adversarial networks'
  source_content_hash: sha256:14c2f0a617f5f0e15205a4c4bb55fd4b663be95332c182eda236c1a7b54436fc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scverse/scanpy-tutorials
  tools:
  - GRNBoost2
  - GRouNdGAN
  - PyTorch
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# PAGA trajectory inference and diffusion pseudotime validation

## Summary

Jointly embeds experimental and simulated scRNA-seq cells in scanpy, then runs PAGA graph abstraction and diffusion pseudotime to verify that GRouNdGAN-simulated data preserve cell trajectories and pseudo-time ordering of the reference dataset (e.g., BoneMarrow, Fig. 4-5 of the GRouNdGAN paper).

## When to use

Use when validating that simulated scRNA-seq data (e.g., GRouNdGAN .h5ad output) reproduce the trajectory structure and pseudo-time ordering of an experimental reference such as mouse BoneMarrow hematopoiesis (Paul et al., 2015; GSE72857).

## When NOT to use

- The reference and simulated data cannot be jointly embedded (e.g., different gene spaces or unnormalized counts — GRouNdGAN outputs library-size normalized expression values only)
- The biological system has no expected transitional structure — PAGA/diffusion pseudotime is designed for trajectories such as hematopoietic differentiation, not for unrelated discrete cell types
- The GRouNdGAN model has not been retrained on the dataset at hand — it is a generative, not predictive, model, so simulated data from a different reference will not match the new trajectory

## Inputs

- GRouNdGAN-simulated scRNA-seq data (simulated.h5ad) generated from a trained .pth checkpoint
- Experimental reference scRNA-seq dataset preprocessed with GRouNdGAN --preprocess (e.g., BoneMarrow, Paul et al. 2015, GEO GSE72857, ~2230 cells)
- Cell type/cluster labels and lineage marker gene sets (e.g., Gata1, Klf1; Mpo, Ctsg; Elane, Cebpe)

## Outputs

- PAGA graph abstraction of the joint experimental + simulated embedding
- Diffusion pseudotime values per cell in both simulated and reference datasets
- Trajectory/pseudo-time concordance assessment (accept/reject simulation) reproducing the paper's Fig. 4-5 comparisons

## How to apply

Generate simulated cells with GRouNdGAN (e.g., 2230 BoneMarrow cells from the final .pth checkpoint) and load the experimental reference into an AnnData object. Jointly embed experimental and simulated cells in scanpy (shared neighbors/UMAP), then run PAGA trajectory inference on cluster labels followed by diffusion pseudotime computation on both datasets. Compare pseudo-time values assigned to matched cell subgroups between simulated and reference data, and check that lineage marker genes activate in sequential order along trajectories: erythroid (Gata2, Gata1, Klf1, Hba-a2), myeloid (Mpo, Ctsg), basophil (Mcpt8, Prss34), neutrophil (Elane, Cebpe), and dendritic/monocyte (Irf8, Csf1r). Accept the simulation if pseudotime ordering and cell-trajectory structure are preserved and simulated cells overlap experimental cells in the joint embedding.

## Related tools

- **scanpy** (Jointly embed experimental and simulated cells, run PAGA trajectory inference, diffusion pseudotime computation, and marker-gene plotting) — scverse/scanpy-tutorials
- **GRouNdGAN** (Produces the simulated single-cell dataset (--generate -> simulated.h5ad) whose trajectory preservation is being validated) — github.com/Emad-COMBINE-lab/GRouNdGAN
- **GRNBoost2** (Infers the causal GRN (top k=15 regulating TFs per gene) imposed during GRouNdGAN training, upstream of simulation) — github.com/Emad-COMBINE-lab/GRouNdGAN

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('joint_experimental_simulated.h5ad'); sc.pp.neighbors(adata); sc.tl.paga(adata, groups='clusters'); sc.tl.dpt(adata)
```

## Evaluation signals

- Pseudo-time values assigned to cell subgroups are strongly concordant between simulated and reference BoneMarrow data
- Erythroid marker genes (Gata2, Gata1, Klf1, Hba-a2) activate in sequential order along the erythroid trajectory in both simulated and experimental data
- Simulated cells overlap experimental cells in the joint embedding (miLISI near 2 and RF AUROC near 0.5, the paper's similarity criteria)
- PAGA connectivity of simulated data reproduces the reference cell-trajectory structure (Fig. 4-5 comparison)
- Lineage marker sets beyond erythroid (myeloid Mpo/Ctsg, basophil Mcpt8/Prss34, neutrophil Elane/Cebpe, dendritic/monocyte Irf8/Csf1r) show consistent activation patterns across both datasets

## Limitations

- GRouNdGAN only generates library-size normalized expression values across the considered genes and TFs; unnormalized count generation is planned for future iterations
- On the small (~2.7k cell) BoneMarrow dataset, simulator performance dropped on the test set compared to the training set
- The resemblance of simulated data to the reference deteriorates if the imposed TF-gene relationships differ significantly from reference data patterns

## Evidence

- [abstract] trajectory preservation via PAGA + diffusion pseudotime: "our model captures non-linear TF-gene dependencies and preserves gene identities, cell trajectories, pseudo-time ordering, and technical and biological noise"
- [methods] scanpy joint embedding, PAGA, and diffusion pseudotime workflow: [evidence span withheld — anchor status: not_found]
- [methods] acceptance criteria for trajectory and pseudotime preservation: [evidence span withheld — anchor status: not_found]
- [methods] BoneMarrow reference dataset: "Reference dataset: Differentiation of hematopoietic stem cells to different lineages from mouse bone marrow (GEO accession number: GSE72857)."
