---
name: causal-gan-scrna-seq-simulation
description: Use when benchmarking GRN inference algorithms (GRNBoost2, GENIE3, PIDC,
  PPCOR) needs realistic scRNA-seq simulations with a known imposed ground-truth GRN,
  or when performing in silico TF perturbation/knockout experiments on reference data
  such as PBMC-CTL.
when_to_use_negative:
- You need to generate unnormalized count data — the current model generates only
  library-size normalized expression values across the considered genes and TFs
- Your GRN is multi-layer or includes TF-gene-phenotype links — the current version
  only supports imposing a bipartite GRN
- You want a predictive model of expression outcomes for a new dataset without retraining
  — GRouNdGAN is generative, not predictive, and must be retrained per dataset
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: GRouNdGAN
  role: Causal GAN used to preprocess reference data, create/impose the GRN, train,
    simulate cells, and benchmark reconstructed GRNs
  repo: https://github.com/Emad-COMBINE-lab/GRouNdGAN
- name: GRNBoost2
  role: Infers the TF-gene ranking for GRN creation (--create_grn) and reconstructs
    GRNs from simulated data for benchmarking
- name: GENIE3
  role: Reconstructs a GRN from the simulated expression matrix as a benchmarked inference
    method
- name: PIDC
  role: Reconstructs a GRN from the simulated expression matrix as a benchmarked inference
    method
- name: PPCOR
  role: Reconstructs a GRN from the simulated expression matrix as a benchmarked inference
    method
- name: BEELINE
  role: Experimental-data benchmarking framework whose results the GRouNdGAN-based
    benchmark is compared against
  repo: https://github.com/Murali-group/Beeline
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-48516-6
    title: 'GRouNdGAN: GRN-guided simulation of single-cell RNA-seq data using causal
      generative adversarial networks'
  source_content_hash: sha256:b5824bb07eb6b1bb2ab43b48d1332931bb8a6fcd92309f43b898670890db2934
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Emad-COMBINE-lab/GRouNdGAN
  tools:
  - BEELINE
  - GENIE3
  - GRNBoost2
  - GRouNdGAN
  - PIDC
  - PPCOR
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# causal-gan-scrna-seq-simulation

## Summary

Simulate single-cell RNA-seq data with GRouNdGAN, a GRN-guided causal implicit generative model that imposes a user-provided ground-truth bipartite TF-gene GRN onto reference-based simulated cells, enabling gold-standard benchmarking of GRN inference methods.

## When to use

Use when benchmarking GRN inference algorithms (GRNBoost2, GENIE3, PIDC, PPCOR) needs realistic scRNA-seq simulations with a known imposed ground-truth GRN, or when performing in silico TF perturbation/knockout experiments on reference data such as PBMC-CTL.

## When NOT to use

- You need to generate unnormalized count data — the current model generates only library-size normalized expression values across the considered genes and TFs
- Your GRN is multi-layer or includes TF-gene-phenotype links — the current version only supports imposing a bipartite GRN
- You want a predictive model of expression outcomes for a new dataset without retraining — GRouNdGAN is generative, not predictive, and must be retrained per dataset

## Inputs

- Reference scRNA-seq expression matrix (e.g., PBMC-CTL from 10x Genomics fresh 68k PBMC Donor A; also BoneMarrow GSE72857, Dahlin GSE107727, Tumor-ALL)
- Homo_sapiens_TF.csv transcription factor list
- Ground-truth GRN as causal_graph.pkl (nested dictionary of gene-index to TF-index sets), either from --create_grn or user-imposed
- GRouNdGAN configuration file specifying [EXPERIMENT], [PREPROCESS], [TRAINING], and [GRN Benchmarking] sections

## Outputs

- simulated.h5ad simulated single-cell expression matrix (10,000 cells by default, library-size normalized values)
- Trained .pth model checkpoint
- Reconstructed GRN CSVs (TF/Gene/importance columns from GRNBoost2, GENIE3, PIDC, PPCOR)
- PR_curve.png with global Precision-Recall curve and AUPRC of each reconstructed GRN vs. the imposed ground truth
- Exported ground-truth GRN CSV via ground truth save path

## How to apply

Preprocess the reference expression matrix (e.g., PBMC-CTL, the CD8+ cytotoxic T-cell subset of PBMC-All from 10x Genomics donor A) with GRouNdGAN's --preprocess step, keeping 1000 highly variable genes and applying library-size normalization to 20,000. Obtain a TF-gene ranking with --create_grn (GRNBoost2, k = 15 top most important TFs per gene, Homo_sapiens_TF.csv TF list), or impose a custom GRN by skipping --create_grn and supplying a causal_graph.pkl nested dictionary of gene-index to TF-index sets (e.g., even/odd-parity positive/negative control GRNs built from the top ranked TFs). Train the causal GAN (latent dim = 128, batch size = 1024, 1,000,000 training steps) with the chosen GRN imposed via --train, then simulate cells with --generate to produce simulated.h5ad (10,000 cells by default). Reconstruct GRNs from the simulated matrix with GRNBoost2, GENIE3, PIDC, and PPCOR (formatted as three-column TF/Gene/importance CSVs) and score them with --benchmark_grn (compute_pr = True) against the imposed ground truth; the rationale is that GRouNdGAN's causal structure enforces the imposed regulatory relationships so recoverable edges provide a gold standard absent from purely observational benchmarks.

## Related tools

- **GRouNdGAN** (Causal GAN used to preprocess reference data, create/impose the GRN, train, simulate cells, and benchmark reconstructed GRNs) — https://github.com/Emad-COMBINE-lab/GRouNdGAN
- **GRNBoost2** (Infers the TF-gene ranking for GRN creation (--create_grn) and reconstructs GRNs from simulated data for benchmarking)
- **GENIE3** (Reconstructs a GRN from the simulated expression matrix as a benchmarked inference method)
- **PIDC** (Reconstructs a GRN from the simulated expression matrix as a benchmarked inference method)
- **PPCOR** (Reconstructs a GRN from the simulated expression matrix as a benchmarked inference method)
- **BEELINE** (Experimental-data benchmarking framework whose results the GRouNdGAN-based benchmark is compared against) — https://github.com/Murali-group/Beeline

## Examples

```
git clone https://github.com/Emad-COMBINE-lab/GRouNdGAN.git && docker pull yazdanz/groundgan:4b98686 && python groundgan.py --preprocess --create_grn --train --generate --benchmark_grn
```

## Evaluation signals

- Reconstructed GRNs (GRNBoost2, GENIE3, PIDC, PPCOR) achieve AUPRC on imposed positive-control edges well above the random-predictor baseline AUPRC on GRouNdGAN-simulated PBMC-CTL data
- Edges belonging to the unimposed negative-control GRN score near random AUPRC, reproducing the Fig. 3 / Supplementary Data 3 pattern
- simulated.h5ad passes --evaluate data-quality checks: preservation of gene identities, cell trajectories, pseudo-time ordering, and technical/biological noise
- Causal graph pickle is a valid nested dictionary mapping gene indices to TF-index sets, with positive and negative control GRNs of equal density
- Training completes the configured 1,000,000 steps and a .pth checkpoint is produced that --generate can consume from the [EXPERIMENT]/checkpoint path

## Limitations

- Resemblance to the reference dataset deteriorates if imposed TF-gene relationships differ significantly from patterns in the reference data
- Hyperparameters may need re-tuning via a validation set when training on new datasets or datasets with very different gene/TF counts
- The underlying causal graph of an observational dataset is not necessarily unique; verifying the imposed GRN is the most probable one is beyond the study's scope
- Only bipartite GRNs are supported; multi-layer GRNs and TF-gene-phenotype causal graphs are planned for future versions
- Simulator performance dropped on the small (~2.7k cell) BoneMarrow test set; benchmarking on PBMC-All was not conclusive and requires follow-up study of distinct cell types

## Evidence

- [abstract] GRouNdGAN, a gene regulatory network (GRN)-guided reference-based causal implicit generative model for simulating single-cell RNA-seq data: "We introduce GRouNdGAN, a gene regulatory network (GRN)-guided reference-based causal implicit generative model for simulating single-cell RNA-seq data"
- [methods] This command uses GRNBoost2 to infer a GRN on the preprocessed train set.: "This command uses GRNBoost2 (Moerman et al., 2018) to infer a GRN on the preprocessed train set."
- [methods] It is possible to instead impose your own GRN onto GRouNdGAN. If you're opting for this option, skip the ``--create_grn`` command.: "It is possible to instead impose your own GRN onto GRouNdGAN. If you're opting for this option, skip the ``--create_grn`` command."
- [methods] This command evaluates the reconstructed GRN (from GRouNdGAN-simulate data) against the ground truth GRN (used for simulation).: "This command evaluates the reconstructed GRN (from GRouNdGAN-simulate data) against the ground truth GRN (used for simulation)."
- [discussion] we recommend first finding a set of regulatory edges from the reference dataset (e.g., using GRNBoost2 or even other data modalities) and imposing a selected subset of those in the simulated data: "we recommend first finding a set of regulatory edges from the reference dataset (e.g., using GRNBoost2 or even other data modalities) and imposing a selected subset of those in the simulated data"
- [discussion] The current version of GRouNdGAN only allows imposing a bipartite GRN and does not support a multi-layer GRN: "The current version of GRouNdGAN only allows imposing a bipartite GRN and does not support a multi-layer GRN, an assumption that we plan to relax with the future versions of the model."
- [abstract] GRouNdGAN effectively bridges the existing gap between simulated and biological data benchmarks of GRN inference algorithms, providing gold standard ground truth GRNs: "GRouNdGAN effectively bridges the existing gap between simulated and biological data benchmarks of GRN inference algorithms, providing gold standard ground truth GRNs"
