---
name: lineage-marker-gene-activation-comparison
description: Use when validating simulated scRNA-seq data against a reference for
  trajectory preservation, e.g. hematopoietic differentiation (BoneMarrow, GSE72857),
  by checking that marker gene sets activate sequentially along PAGA-inferred branches
  and pseudo-time ordering is concordant.
when_to_use_negative:
- The simulated and reference data come from an isolated cell type without a differentiation
  trajectory (e.g. PBMC-CTL) — trajectory and pseudotime comparison is not meaningful
  there
- The goal is benchmarking GRN inference quality rather than validating simulation
  fidelity — use GRouNdGAN's --benchmark_grn against the exported ground truth GRN
  instead
- The imposed GRN's TF-gene relationships differ significantly from reference data
  patterns — resemblance to the reference deteriorates and the comparison would fail
  for reasons unrelated to trajectory preservation
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: GRouNdGAN
  role: Generates the simulated scRNA-seq data (--preprocess, --create_grn, --train,
    --generate) that is compared to the reference for trajectory and pseudotime preservation
  repo: https://github.com/Emad-COMBINE-lab/GRouNdGAN
- name: GRNBoost2
  role: Infers the causal GRN (top k=15 regulating TFs per gene) imposed during simulation
    via GRouNdGAN's --create_grn step
- name: scanpy
  role: Jointly embeds experimental and simulated cells, runs PAGA trajectory inference
    and diffusion pseudotime computation, and supports marker gene activation comparison
  repo: https://github.com/scverse/scanpy
- name: PyTorch
  role: Backend used to train GRouNdGAN on a CUDA GPU and to resume from .pth checkpoints
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-48516-6
    title: 'GRouNdGAN: GRN-guided simulation of single-cell RNA-seq data using causal
      generative adversarial networks'
  source_content_hash: sha256:52e02b98ded0a849918ad03f452d7db83c7e53de10b43324253c59281b3cfd4e
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# lineage-marker-gene-activation-comparison

## Summary

Validates that GRouNdGAN-simulated scRNA-seq data preserves the transitional states (trajectories) and pseudo-time ordering of an experimental reference by jointly embedding both datasets in scanpy, running PAGA and diffusion pseudotime, and comparing lineage marker gene activation across trajectories (reproducing the BoneMarrow Fig. 4-5 analyses).

## When to use

Use when validating simulated scRNA-seq data against a reference for trajectory preservation, e.g. hematopoietic differentiation (BoneMarrow, GSE72857), by checking that marker gene sets activate sequentially along PAGA-inferred branches and pseudo-time ordering is concordant.

## When NOT to use

- The simulated and reference data come from an isolated cell type without a differentiation trajectory (e.g. PBMC-CTL) — trajectory and pseudotime comparison is not meaningful there
- The goal is benchmarking GRN inference quality rather than validating simulation fidelity — use GRouNdGAN's --benchmark_grn against the exported ground truth GRN instead
- The imposed GRN's TF-gene relationships differ significantly from reference data patterns — resemblance to the reference deteriorates and the comparison would fail for reasons unrelated to trajectory preservation

## Inputs

- Experimental reference scRNA-seq dataset (BoneMarrow, Paul et al., 2015; GEO GSE72857) preprocessed for GAN training
- GRouNdGAN-trained model checkpoint (.pth state dictionary)
- simulated.h5ad file from GRouNdGAN --generate (2230 cells for BoneMarrow)
- Causal GRN inferred by GRNBoost2 (top k=15 regulating TFs per gene)
- Lineage marker gene sets: erythroid (Gata1, Klf1), myeloid (Mpo, Ctsg), basophil (Mcpt8, Prss34), neutrophil (Elane, Cebpe), dendritic/monocyte (Irf8, Csf1r), and erythroid sequence (Gata2, Gata1, Klf1, Hba-a2)

## Outputs

- Joint embedding of experimental and simulated cells with PAGA trajectory graph
- Diffusion pseudotime values per cell subgroup for both datasets
- Lineage marker gene activation comparisons along trajectories (Fig. 4-5 style plots)
- miLISI and random forest AUROC similarity metrics

## How to apply

First generate the simulated dataset: preprocess the reference (Paul et al., 2015; GEO GSE72857) with GRouNdGAN's --preprocess step, infer the causal GRN with GRNBoost2 (top k=15 regulating TFs per gene) via --create_grn, train with --train (maximum steps = 1000000, batch size = 1024, critic iterations = 5, gradient-penalty lambda = 10) on a CUDA GPU, then populate the [EXPERIMENT]/checkpoint field with the final .pth state dictionary, set the number of cells to generate (2230 for BoneMarrow) and run --generate to write simulated.h5ad. In scanpy, jointly embed the experimental and simulated cells, run PAGA trajectory inference, and compute diffusion pseudotime. Compare activation of lineage marker gene sets across inferred trajectories: erythroid (Gata1, Klf1), myeloid (Mpo, Ctsg), basophil (Mcpt8, Prss34), neutrophil (Elane, Cebpe), and dendritic/monocyte (Irf8, Csf1r). Accept the simulation only if pseudotime ordering and cell-trajectory structure are preserved and simulated cells overlap experimental cells in the joint embedding (miLISI near 2 and RF AUROC near 0.5 as the paper's similarity criteria); for the erythroid trajectory specifically, check that Gata2, Gata1, Klf1, Hba-a2 activate in sequential order in both datasets.

## Related tools

- **GRouNdGAN** (Generates the simulated scRNA-seq data (--preprocess, --create_grn, --train, --generate) that is compared to the reference for trajectory and pseudotime preservation) — https://github.com/Emad-COMBINE-lab/GRouNdGAN
- **GRNBoost2** (Infers the causal GRN (top k=15 regulating TFs per gene) imposed during simulation via GRouNdGAN's --create_grn step)
- **scanpy** (Jointly embeds experimental and simulated cells, runs PAGA trajectory inference and diffusion pseudotime computation, and supports marker gene activation comparison) — https://github.com/scverse/scanpy
- **PyTorch** (Backend used to train GRouNdGAN on a CUDA GPU and to resume from .pth checkpoints)

## Evaluation signals

- Pseudo-time values assigned to cell subgroups in the simulated data show strong concordance with the reference data
- Erythroid markers Gata2, Gata1, Klf1, Hba-a2 activate in sequential order along the erythroid trajectory in both simulated and experimental datasets
- Simulated cells overlap experimental cells in the joint embedding, with miLISI near 2
- Random forest classifier AUROC distinguishing simulated from experimental cells is near 0.5 (indistinguishability)
- Lineage marker sets activate along the correct PAGA branches in both datasets: erythroid (Gata1, Klf1), myeloid (Mpo, Ctsg), basophil (Mcpt8, Prss34), neutrophil (Elane, Cebpe), dendritic/monocyte (Irf8, Csf1r)

## Limitations

- In the BoneMarrow dataset (only ~2.7k cells), the performance of all simulators dropped on the test set compared to the training set
- GRouNdGAN generates library-size normalized expression values across considered genes and TFs; unnormalized count generation is not yet supported
- Given a new dataset, the model needs to be retrained — GRouNdGAN is generative, not predictive; hyperparameters may need re-tuning via a validation set for datasets with significantly different numbers of genes and TFs
- The current version only allows imposing a bipartite GRN and does not support multi-layer GRNs

## Evidence

- [abstract] GRouNdGAN captures non-linear TF-gene dependencies and preserves gene identities, cell trajectories, pseudo-time ordering: "our model captures non-linear TF-gene dependencies and preserves gene identities, cell trajectories, pseudo-time ordering, and technical and biological noise"
- [methods] BoneMarrow reference dataset is mouse bone marrow hematopoiesis (GSE72857): "Reference dataset: Differentiation of hematopoietic stem cells to different lineages from mouse bone marrow (GEO accession number: GSE72857)."
- [methods] --generate simulates single-cell RNA-seq data in-silico: "--generate       Simulate single-cells RNA-seq data in-silico"
- [discussion] BoneMarrow is small (~2.7k cells) and all simulators' performance dropped on the test set: "In the BoneMarrow dataset (which had only ~2.7k cells), the performance of all simulators dropped on the test set"
- [readme] GRouNdGAN documentation names the GRN-guided causal GAN for scRNA-seq simulation: "GRN-guided in silic**o** sim**u**lation of single-cell R**N**A-seq **d**ata using Causal **G**enerative **A**dversarial **N**etworks"
