---
name: grn-guided-gan-training
description: Use when you need reference-based simulation of scRNA-seq data that retains
  the regulatory dynamics of a user-provided causal GRN, e.g. for benchmarking GRN
  inference or pseudo-time preservation studies.
when_to_use_negative:
- You need unnormalized count data — GRouNdGAN currently only generates expression
  values library-size normalized across the considered genes and TFs
- Your GRN is multi-layer or includes TF-gene-phenotype edges — the current version
  only supports imposing a bipartite GRN
- You want to predict outcomes on a new dataset without retraining — GRouNdGAN is
  generative, not predictive, and must be retrained for a new dataset
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: GRouNdGAN
  role: The GRN-guided causal GAN being trained (--preprocess, --create_grn, --train,
    --generate CLI steps)
  repo: https://github.com/Emad-COMBINE-lab/GRouNdGAN
- name: GRNBoost2
  role: Infers the GRN from the preprocessed training set (top k=15 regulating TFs
    per gene) that is formatted as the causal graph imposed during training
  repo: https://github.com/Murali-group/Beeline
- name: PyTorch
  role: Deep-learning backend for training the GAN on a CUDA GPU and loading .pth
    checkpoints
- name: scanpy
  role: Joint embedding and downstream quality checks (PAGA, diffusion pseudotime,
    miLISI/AUROC) of generated cells against the reference
  repo: https://github.com/scverse/scanpy-tutorials
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-48516-6
    title: 'GRouNdGAN: GRN-guided simulation of single-cell RNA-seq data using causal
      generative adversarial networks'
  source_content_hash: sha256:52009d4e377eba8fb4aa375bbd1bad0e538ab8182e9cfb773af07e71fecd4f9d
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# grn-guided-gan-training

## Summary

Train GRouNdGAN, a GRN-guided causal implicit GAN, on a preprocessed scRNA-seq reference dataset with a GRNBoost2-inferred causal graph, then generate in-silico single cells (simulated.h5ad) that preserve gene identities, trajectories, and pseudo-time ordering.

## When to use

Use when you need reference-based simulation of scRNA-seq data that retains the regulatory dynamics of a user-provided causal GRN, e.g. for benchmarking GRN inference or pseudo-time preservation studies.

## When NOT to use

- You need unnormalized count data — GRouNdGAN currently only generates expression values library-size normalized across the considered genes and TFs
- Your GRN is multi-layer or includes TF-gene-phenotype edges — the current version only supports imposing a bipartite GRN
- You want to predict outcomes on a new dataset without retraining — GRouNdGAN is generative, not predictive, and must be retrained for a new dataset

## Inputs

- Raw scRNA-seq reference dataset (e.g. BoneMarrow from Paul et al., 2015; GEO GSE72857)
- Preprocessed training set (output of --preprocess)
- Causal GRN inferred by GRNBoost2 or a user-imposed custom bipartite GRN
- Trained model checkpoint (.pth state dictionary)
- Config file with [EXPERIMENT] fields including checkpoint path and number of cells to generate

## Outputs

- Formatted causal graph of TF-gene regulatory edges (from --create_grn)
- Trained GRouNdGAN model weights (.pth checkpoint)
- Simulated single-cell RNA-seq expression matrix in simulated.h5ad (library-size normalized)

## How to apply

Run GRouNdGAN's --preprocess step on the raw reference dataset (e.g. BoneMarrow, Paul et al. 2015, GEO GSE72857), then --create_grn to infer a GRN with GRNBoost2 (top k=15 regulating TFs per gene) formatted as a causal graph; alternatively impose a custom GRN and skip --create_grn. Train with --train under default settings (maximum steps = 1000000, batch size = 1024, critic iterations = 5, gradient-penalty lambda = 10) on a CUDA GPU; training can be resumed from a .pth checkpoint. After training, populate the [EXPERIMENT]/checkpoint field with the final .pth state dictionary, set the number of cells to generate (e.g. 2230 for BoneMarrow), and run --generate to write simulated.h5ad. Correctness is judged by downstream similarity checks: joint scanpy embedding of simulated and experimental cells with miLISI near 2 and a random-forest AUROC near 0.5, plus preservation of cell trajectories and pseudo-time ordering (PAGA, diffusion pseudotime) as in the paper's Fig. 4-5.

## Related tools

- **GRouNdGAN** (The GRN-guided causal GAN being trained (--preprocess, --create_grn, --train, --generate CLI steps)) — https://github.com/Emad-COMBINE-lab/GRouNdGAN
- **GRNBoost2** (Infers the GRN from the preprocessed training set (top k=15 regulating TFs per gene) that is formatted as the causal graph imposed during training) — https://github.com/Murali-group/Beeline
- **PyTorch** (Deep-learning backend for training the GAN on a CUDA GPU and loading .pth checkpoints)
- **scanpy** (Joint embedding and downstream quality checks (PAGA, diffusion pseudotime, miLISI/AUROC) of generated cells against the reference) — https://github.com/scverse/scanpy-tutorials

## Examples

```
docker pull yazdanz/groundgan:4b98686
```

## Evaluation signals

- miLISI near 2 when simulated and experimental cells are jointly embedded, indicating cell-level mixing
- Random-forest classifier AUROC near 0.5 (indistinguishable simulated vs. experimental cells), the paper's similarity criterion
- PAGA trajectory structure and diffusion pseudotime ordering of simulated BoneMarrow cells match the reference (Fig. 4-5 comparisons)
- Lineage marker genes activate in sequential order along trajectories, e.g. erythroid Gata2, Gata1, Klf1, Hba-a2, in both simulated and experimental data
- Simulated cells overlap experimental cells in the joint embedding for each lineage (erythroid, myeloid, basophil, neutrophil, dendritic/monocyte)

## Limitations

- Performance of all simulators, including GRouNdGAN, dropped on the test set for the small (~2.7k cell) BoneMarrow dataset
- Hyperparameters may need re-tuning via a validation set when training on a completely new dataset or datasets with significantly different numbers of genes and TFs
- Resemblance to the reference dataset deteriorates if imposed TF-gene relationships differ significantly from patterns in the reference data
- The underlying causal graph of an observational dataset is not necessarily unique, and verifying the imposed GRN is the most probable one is beyond the study's scope

## Evidence

- [methods] GRN creation via GRNBoost2: "--create_grn     Infer a GRN from preprocessed data using GRNBoost2 and appropriately format as causal graph"
- [methods] Training step: "--train          Start or resume model training"
- [methods] Checkpoint requirement before generation: "One training is done, populate the ``[EXPERIMENT]/checkpoint`` field with the path of the ``.pth`` checkpoint"
- [discussion] Degradation with mismatched GRN: "the resemblance of the data to the reference dataset deteriorates if the imposed TF-gene relationships are significantly different"
- [abstract] Core capability claim: "our model captures non-linear TF-gene dependencies and preserves gene identities, cell trajectories, pseudo-time ordering, and technical and biological noise"
