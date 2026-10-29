---
name: diffusion-pseudotime-inference
description: Use when validating that simulated scRNA-seq data (e.g., from GRouNdGAN
  on the BoneMarrow dataset, Paul et al. 2015, GSE72857) preserves cell trajectories
  and pseudo-time ordering of a reference dataset, as in Figs. 4-5.
when_to_use_negative:
- GRouNdGAN only generates library-size normalized expression values across considered
  genes and TFs, so this validation approach should not assume unnormalized count
  data are available from the simulator
- The BoneMarrow dataset is small (~2.7k cells) and simulator performance dropped
  on the test set, so trajectory concordance on this dataset should not be generalized
  to larger datasets (e.g., Dahlin, ~45k cells) without re-running the full pipeline
- This is a validation/similarity skill, not GRN benchmarking — do not use it to evaluate
  GRN inference method performance; use GRouNdGAN's --benchmark_grn workflow instead
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: GRouNdGAN
  role: Generates the simulated scRNA-seq cells (--generate) whose trajectory and
    pseudotime preservation is being validated against the reference
  repo: https://github.com/Emad-COMBINE-lab/GRouNdGAN
- name: scanpy
  role: Jointly embeds experimental and simulated cells, runs PAGA trajectory inference
    and diffusion pseudotime computation, and plots marker-gene activation ordering
  repo: https://github.com/scverse/scanpy-tutorials
- name: GRNBoost2
  role: Infers the causal GRN (top k=15 regulating TFs per gene) from preprocessed
    reference data before GRouNdGAN training
  repo: https://github.com/Emad-COMBINE-lab/GRouNdGAN
- name: PyTorch
  role: Backend for training the GRouNdGAN GAN (maximum steps = 1000000, batch size
    = 1024, critic iterations = 5, gradient-penalty lambda = 10) on a CUDA GPU
  repo: https://github.com/Emad-COMBINE-lab/GRouNdGAN
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-48516-6
    title: 'GRouNdGAN: GRN-guided simulation of single-cell RNA-seq data using causal
      generative adversarial networks'
  source_content_hash: sha256:c0b89c07287afcb7b63e16ae70767d0c884c1ed6f98d54d35a05d71d881c37be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Emad-COMBINE-lab/GRouNdGAN
  tools:
  - GRNBoost2
  - GRouNdGAN
  - PyTorch
  - scanpy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# Diffusion pseudotime inference for validating simulated scRNA-seq trajectory preservation

## Summary

Jointly embed experimental and GRouNdGAN-simulated single-cell data in scanpy, run PAGA trajectory inference and diffusion pseudotime, then compare marker-gene activation ordering (e.g., Gata2→Gata1→Klf1→Hba-a2 along the erythroid trajectory) between reference and simulated cells to verify pseudo-time and trajectory preservation.

## When to use

Use when validating that simulated scRNA-seq data (e.g., from GRouNdGAN on the BoneMarrow dataset, Paul et al. 2015, GSE72857) preserves cell trajectories and pseudo-time ordering of a reference dataset, as in Figs. 4-5.

## When NOT to use

- GRouNdGAN only generates library-size normalized expression values across considered genes and TFs, so this validation approach should not assume unnormalized count data are available from the simulator
- The BoneMarrow dataset is small (~2.7k cells) and simulator performance dropped on the test set, so trajectory concordance on this dataset should not be generalized to larger datasets (e.g., Dahlin, ~45k cells) without re-running the full pipeline
- This is a validation/similarity skill, not GRN benchmarking — do not use it to evaluate GRN inference method performance; use GRouNdGAN's --benchmark_grn workflow instead

## Inputs

- Experimental reference scRNA-seq dataset (e.g., Mouse BoneMarrow, Paul et al., 2015, GEO GSE72857) as h5ad/AnnData
- GRouNdGAN-simulated.h5ad file with the same number of cells as specified at generation (e.g., 2230 for BoneMarrow)
- Lineage marker gene sets: erythroid (Gata2, Gata1, Klf1, Hba-a2), myeloid (Mpo, Ctsg), basophil (Mcpt8, Prss34), neutrophil (Elane, Cebpe), dendritic/monocyte (Irf8, Csf1r)

## Outputs

- PAGA trajectory graph over the joint experimental+simulated embedding
- Diffusion pseudotime values assigned to cell subgroups in both reference and simulated data
- Marker-gene activation ordering plots along the inferred trajectory (Fig. 4-5 style comparisons)
- Similarity metrics: miLISI (near 2) and RF classifier AUROC (near 0.5)

## How to apply

First generate simulated cells with GRouNdGAN (--generate, e.g., 2230 cells for BoneMarrow from the final .pth checkpoint) and load the reference dataset. In scanpy, jointly embed experimental and simulated cells (batch mixing assessed via miLISI near 2 and a random-forest classifier AUROC near 0.5). Run PAGA to infer the trajectory graph and compute diffusion pseudotime over the joint embedding. Then compare activation of lineage marker gene sets across the inferred trajectories in both datasets: erythroid (Gata1, Klf1), myeloid (Mpo, Ctsg), basophil (Mcpt8, Prss34), neutrophil (Elane, Cebpe), and dendritic/monocyte (Irf8, Csf1r). Accept the simulation if pseudotime ordering and cell-trajectory structure are preserved and simulated cells overlap experimental cells in the joint embedding.

## Related tools

- **GRouNdGAN** (Generates the simulated scRNA-seq cells (--generate) whose trajectory and pseudotime preservation is being validated against the reference) — https://github.com/Emad-COMBINE-lab/GRouNdGAN
- **scanpy** (Jointly embeds experimental and simulated cells, runs PAGA trajectory inference and diffusion pseudotime computation, and plots marker-gene activation ordering) — https://github.com/scverse/scanpy-tutorials
- **GRNBoost2** (Infers the causal GRN (top k=15 regulating TFs per gene) from preprocessed reference data before GRouNdGAN training) — https://github.com/Emad-COMBINE-lab/GRouNdGAN
- **PyTorch** (Backend for training the GRouNdGAN GAN (maximum steps = 1000000, batch size = 1024, critic iterations = 5, gradient-penalty lambda = 10) on a CUDA GPU) — https://github.com/Emad-COMBINE-lab/GRouNdGAN

## Examples

```
python run.py --preprocess --create_grn --train && python run.py --generate && python -c "import scanpy as sc; adata=sc.read_h5ad('simulated.h5ad'); sc.tl.paga(adata); sc.tl.dpt(adata)"
```

## Evaluation signals

- Pseudotime ordering of cell subgroups in simulated data is concordant with the reference data
- Erythroid markers (Gata2, Gata1, Klf1, Hba-a2) activate in the same sequential order along the erythroid trajectory in both simulated and experimental data
- Simulated cells overlap experimental cells in the joint embedding, with miLISI near 2
- A random-forest classifier trained to distinguish real vs. simulated cells achieves AUROC near 0.5
- Lineage marker sets for myeloid (Mpo, Ctsg), basophil (Mcpt8, Prss34), neutrophil (Elane, Cebpe), and dendritic/monocyte (Irf8, Csf1r) show consistent activation patterns across both datasets' trajectories

## Limitations

- GRouNdGAN is generative, not predictive: for a new dataset the model must be retrained before trajectory/pseudotime validation is meaningful
- Pseudotime concordance depends on the imposed GRN; resemblance to the reference deteriorates if imposed TF-gene relationships differ significantly from reference data patterns
- On the small (~2.7k cell) BoneMarrow dataset, simulator performance dropped on the test set compared to the training set
- Hyperparameters may need re-tuning via a validation set for datasets with significantly different numbers of genes and TFs

## Evidence

- [methods] PAGA trajectory and pseudotime preservation workflow: [evidence span withheld — anchor status: not_found]
- [results] Marker gene ordering along erythroid trajectory: [evidence span withheld — anchor status: not_found]
- [results] Similarity criteria: [evidence span withheld — anchor status: not_found]
- [abstract] Abstract claim of trajectory preservation: "our model captures non-linear TF-gene dependencies and preserves gene identities, cell trajectories, pseudo-time ordering, and technical and biological noise"
- [discussion] Normalization limitation: "the model generates expression values library-size normalized across the considered genes and TFs"
