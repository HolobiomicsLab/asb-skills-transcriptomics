---
name: simulated-vs-experimental-trajectory-benchmarking
description: Use when you have a trained generative scRNA-seq simulator's output (.h5ad)
  plus the original reference dataset and need to verify that developmental trajectories,
  transitional states, and pseudotime ordering were preserved before downstream use.
when_to_use_negative:
- The model was trained on a completely new dataset or one with significantly different
  numbers of genes and TFs — hyperparameters may need re-tuning via a validation set
  before trajectory fidelity can be expected
- You want to evaluate numeric GRN inference benchmarking (AUROC of inferred edges
  against ground truth) rather than trajectory/trajectory-structure preservation —
  use the --benchmark_grn evaluation instead
- The simulation goal is unnormalized raw count generation — GRouNdGAN only generates
  library-size normalized expression values across the considered genes and TFs
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: GRouNdGAN
  role: Generates the simulated scRNA-seq cells (--generate, 2230 cells from the final
    .pth checkpoint) whose trajectories are benchmarked
  repo: https://github.com/Emad-COMBINE-lab/GRouNdGAN
- name: scanpy
  role: Jointly embeds experimental and simulated cells, runs PAGA trajectory inference
    and diffusion pseudotime computation, and visualizes marker gene activation order
    (scanpy tutorials referenced)
  repo: https://github.com/scverse/scanpy-tutorials
- name: GRNBoost2
  role: Infers the causal GRN (top k=15 regulating TFs per gene) via --create_grn,
    which guides the simulation being benchmarked
  repo: https://github.com/Emad-COMBINE-lab/GRouNdGAN
- name: PyTorch
  role: Backend for training GRouNdGAN on a CUDA GPU (maximum steps = 1000000, batch
    size = 1024, critic iterations = 5, gradient-penalty lambda = 10)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-48516-6
    title: 'GRouNdGAN: GRN-guided simulation of single-cell RNA-seq data using causal
      generative adversarial networks'
  source_content_hash: sha256:5f4e18a2d8556dc1acdffb0b93fe90adda485a186bef2f33c972ebce24fdbb39
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Emad-COMBINE-lab/GRouNdGAN
  tools:
  - GRNBoost2
  - GRouNdGAN
  - PyTorch
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# Simulated vs. experimental trajectory benchmarking (PAGA + diffusion pseudotime)

## Summary

Qualitative-quantitative benchmarking skill that checks whether GRouNdGAN-simulated single-cell RNA-seq data preserves the cell trajectories and pseudo-time ordering of an experimental reference (e.g., BoneMarrow hematopoiesis) by jointly embedding both datasets in scanpy and comparing PAGA structure, diffusion pseudotime, and lineage marker gene activation order.

## When to use

Use when you have a trained generative scRNA-seq simulator's output (.h5ad) plus the original reference dataset and need to verify that developmental trajectories, transitional states, and pseudotime ordering were preserved before downstream use.

## When NOT to use

- The model was trained on a completely new dataset or one with significantly different numbers of genes and TFs — hyperparameters may need re-tuning via a validation set before trajectory fidelity can be expected
- You want to evaluate numeric GRN inference benchmarking (AUROC of inferred edges against ground truth) rather than trajectory/trajectory-structure preservation — use the --benchmark_grn evaluation instead
- The simulation goal is unnormalized raw count generation — GRouNdGAN only generates library-size normalized expression values across the considered genes and TFs

## Inputs

- GRouNdGAN-simulated single-cell expression matrix (simulated.h5ad) generated with --generate from the trained .pth checkpoint
- Preprocessed experimental reference scRNA-seq dataset (e.g., Mouse BoneMarrow, Paul et al. 2015, GEO GSE72857)
- List of lineage marker gene sets (erythroid: Gata2, Gata1, Klf1, Hba-a2; myeloid: Mpo, Ctsg; basophil: Mcpt8, Prss34; neutrophil: Elane, Cebpe; dendritic/monocyte: Irf8, Csf1r)

## Outputs

- Joint embedding of experimental and simulated cells (scanpy object)
- PAGA trajectory graph and diffusion pseudotime values for both datasets
- Marker gene activation-order comparison along each lineage trajectory (Fig. 4-5 style plots)
- Trajectory-preservation verdict: pass if pseudotime ordering is preserved, trajectories match, miLISI ≈ 2, and RF AUROC ≈ 0.5

## How to apply

Take the simulated.h5ad produced by GRouNdGAN --generate (e.g., 2230 cells for the BoneMarrow dataset, Paul et al. 2015, GEO GSE72857) and the preprocessed experimental reference, jointly embed experimental and simulated cells in scanpy, then run PAGA trajectory inference followed by diffusion pseudotime computation on the joint embedding. Compare pseudo-time values assigned to matched cell subgroups (erythroid, myeloid, basophil, neutrophil, dendritic/monocyte) between simulated and experimental cells, and verify that lineage marker genes activate in the same sequential order along inferred trajectories — for the erythroid branch: Gata2, Gata1, Klf1, Hba-a2; myeloid: Mpo, Ctsg; basophil: Mcpt8, Prss34; neutrophil: Elane, Cebpe; dendritic/monocyte: Irf8, Csf1r. Accept the simulation if pseudotime ordering and cell-trajectory structure are preserved and simulated cells overlap experimental cells in the joint embedding, using the paper's similarity criteria of miLISI near 2 and RF AUROC near 0.5.

## Related tools

- **GRouNdGAN** (Generates the simulated scRNA-seq cells (--generate, 2230 cells from the final .pth checkpoint) whose trajectories are benchmarked) — https://github.com/Emad-COMBINE-lab/GRouNdGAN
- **scanpy** (Jointly embeds experimental and simulated cells, runs PAGA trajectory inference and diffusion pseudotime computation, and visualizes marker gene activation order (scanpy tutorials referenced)) — https://github.com/scverse/scanpy-tutorials
- **GRNBoost2** (Infers the causal GRN (top k=15 regulating TFs per gene) via --create_grn, which guides the simulation being benchmarked) — https://github.com/Emad-COMBINE-lab/GRouNdGAN
- **PyTorch** (Backend for training GRouNdGAN on a CUDA GPU (maximum steps = 1000000, batch size = 1024, critic iterations = 5, gradient-penalty lambda = 10))

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('joint_ref_sim.h5ad'); sc.pp.neighbors(adata); sc.tl.paga(adata); sc.tl.dpt(adata)
```

## Evaluation signals

- miLISI near 2 (mixed cell-type label entropy) for simulated vs. experimental cells in the joint embedding
- RF AUROC near 0.5 for the random-forest classifier distinguishing simulated from experimental cells
- Pseudo-time values assigned to matched cell subgroups show strong concordance between simulated and reference data
- Erythroid markers activate in sequential order Gata2 → Gata1 → Klf1 → Hba-a2 along the erythroid trajectory in both datasets, matching previous findings (Paul et al., 2015)
- PAGA graph structure: same lineage branches (erythroid, myeloid, basophil, neutrophil, dendritic/monocyte) recovered for simulated and experimental cells

## Limitations

- In the BoneMarrow dataset (only ~2.7k cells), the performance of all simulators dropped on the test set compared to the training set — small references degrade simulation quality
- GRouNdGAN is generative, not predictive; given a new dataset, the model must be retrained — trajectory fidelity conclusions do not transfer across datasets
- Resemblance to the reference dataset deteriorates if the imposed TF-gene relationships differ significantly from patterns in the reference data
- The model generates expression values library-size normalized across the considered genes and TFs, not unnormalized counts

## Evidence

- [full_text] PAGA + pseudotime workflow: [evidence span withheld — anchor status: not_found]
- [full_text] similarity criteria: [evidence span withheld — anchor status: not_found]
- [abstract] preservation finding: "our model captures non-linear TF-gene dependencies and preserves gene identities, cell trajectories, pseudo-time ordering, and technical and biological noise"
- [methods] reference dataset identity: "Reference dataset: Differentiation of hematopoietic stem cells to different lineages from mouse bone marrow (GEO accession number: GSE72857)."
- [methods] evaluation step exists in tool: "--evaluate       Evaluate the data quality of the simulated dataset"
