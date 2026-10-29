---
name: scrnaseq-count-data-modeling
description: Use when raw scRNA-seq count matrices need a denoised, low-dimensional
  latent feature representation that explicitly models zero inflation (dropout) before
  downstream cell-type clustering.
when_to_use_negative:
- Input is already a normalized feature table or latent embedding rather than raw
  scRNA-seq count data with dropouts
- The goal is consensus clustering itself — that is handled by the third scBGEDA module
  (BC.m / bipartite graph ensemble clustering), not the SDDA network
- Data has not passed through the first preprocessing module (preprocess.py), so highly
  variable genes have not been selected
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: scBGEDA
  role: Parent pipeline whose second module implements the single-cell Dual Denoising
    Autoencoder
  repo: https://github.com/wangyh082/scBGEDA
- name: SDDA.py
  role: Entry point that trains the dual denoising autoencoder and saves the latent
    embedding
  repo: https://github.com/wangyh082/scBGEDA
- name: network.py
  role: Defines the Single-cell Dual Denoising Autoencoder network architecture
  repo: https://github.com/wangyh082/scBGEDA
- name: loss.py
  role: Implements the ZINB negative-log and MSE denoising loss functions
  repo: https://github.com/wangyh082/scBGEDA
- name: utils.py
  role: Utility functions supporting the network
  repo: https://github.com/wangyh082/scBGEDA
- name: TensorFlow
  role: Deep learning framework for building and training the autoencoder
- name: Keras
  role: Model API used for the encoder/decoder layers
- name: NumPy
  role: Array handling for count matrices and embeddings
- name: SciPy
  role: Scientific routines used in loss/utility code
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad075
    title: 'scBGEDA: deep single-cell clustering analysis via a dual denoising autoencoder
      with bipartite graph ensemble clustering'
  source_content_hash: sha256:39f0b8ee446fbca84cc15385fe80b4f9e06de2271e78ecd1070ccc5c2baeb899
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wangyh082/scBGEDA
  tools:
  - Keras
  - NumPy
  - Python
  - SDDA.py
  - SciPy
  - TensorFlow
  - loss.py
  - network.py
  - scBGEDA
  - utils.py
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6538461538461539
n_claims: 26
---

# scrnaseq-count-data-modeling

## Summary

Reconstruct and train the single-cell Dual Denoising Autoencoder (SDDA) from scBGEDA, which corrupts scRNA-seq counts with Gaussian noise and learns a 32-dimensional latent embedding by jointly optimizing a ZINB negative-log reconstruction loss and an MSE denoising loss.

## When to use

Use when raw scRNA-seq count matrices need a denoised, low-dimensional latent feature representation that explicitly models zero inflation (dropout) before downstream cell-type clustering.

## When NOT to use

- Input is already a normalized feature table or latent embedding rather than raw scRNA-seq count data with dropouts
- The goal is consensus clustering itself — that is handled by the third scBGEDA module (BC.m / bipartite graph ensemble clustering), not the SDDA network
- Data has not passed through the first preprocessing module (preprocess.py), so highly variable genes have not been selected

## Inputs

- Preprocessed scRNA-seq count matrix (e.g. Adam dataset, 2000 highly variable genes)
- Command-line parameters: dataname, highly_genes, random_seed, dims, alpha, learning_rate, batch_size, pretrain_epoch, noise_sd, gpu_option

## Outputs

- 32-dimensional latent embedding matrix for all cells (latent_embedding.csv)
- Training curves/logs of the ZINB (L1) and denoising (L2) loss terms

## How to apply

Load the preprocessed scRNA-seq count matrix (e.g. the Adam dataset restricted to 2000 highly variable genes) as input to the second module of scBGEDA. Build the network (network.py) in TensorFlow/Keras: a Gaussian-noise-corrupted encoder (noise_sd, e.g. 1.5) projects cells into a compressed 32-dimensional latent space Z via hidden dims such as '256, 32'. Implement loss.py with a ZINB master decoder estimating zero-inflation probability pi, mean mu, and dispersion theta, optimized by negative-log ZINB likelihood (L1), plus an MSE follower decoder approximating the denoised mu (L2), combined as L_h = lambda*L1 + gamma*L2 with alpha controlling the decoders' relative impact. Train end-to-end by running SDDA.py with the Adam optimizer (learning_rate 0.0001, batch_size 256, pretrain_epoch 1000) until both loss terms decrease and converge. Extract and save the learned 32-dimensional latent embedding matrix for all cells (latent_embedding.csv) for the subsequent bipartite graph ensemble clustering stage.

## Related tools

- **scBGEDA** (Parent pipeline whose second module implements the single-cell Dual Denoising Autoencoder) — https://github.com/wangyh082/scBGEDA
- **SDDA.py** (Entry point that trains the dual denoising autoencoder and saves the latent embedding) — https://github.com/wangyh082/scBGEDA
- **network.py** (Defines the Single-cell Dual Denoising Autoencoder network architecture) — https://github.com/wangyh082/scBGEDA
- **loss.py** (Implements the ZINB negative-log and MSE denoising loss functions) — https://github.com/wangyh082/scBGEDA
- **utils.py** (Utility functions supporting the network) — https://github.com/wangyh082/scBGEDA
- **TensorFlow** (Deep learning framework for building and training the autoencoder)
- **Keras** (Model API used for the encoder/decoder layers)
- **NumPy** (Array handling for count matrices and embeddings)
- **SciPy** (Scientific routines used in loss/utility code)

## Examples

```
python SDDA.py --dataname Adam --highly_genes 2000 --random_seed "1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000" --dims "256, 32" --alpha 0.001 --learning_rate 0.0001 --batch_size 256 --pretrain_epoch 1000 --noise_sd 1.5
```

## Evaluation signals

- SDDA.py executes without error on the deposited dataset (e.g. python SDDA.py --dataname Adam)
- latent_embedding.csv contains one 32-dimensional vector per input cell, matching the encoder output dimension
- Both the ZINB reconstruction loss (L1) and the denoising MSE loss (L2) decrease monotonically (with noise) over pretrain epochs toward convergence
- The learned latent space, when fed to the bipartite graph ensemble clustering module, yields clustering performance reported via ARI and NMI median values
- Network hyperparameters (dims '256, 32', noise_sd 1.5, alpha 0.001) match the documented defaults/usage

## Limitations

- Requires selecting highly variable genes (e.g. 2000) via the preprocessing module first; raw unfiltered matrices are not the intended input
- Sensitivity to hyperparameters: noise_sd, alpha, learning_rate, batch_size, and pretrain_epoch must be tuned per dataset; random seeds across 1111–10000 are used to ensure fairness
- Results depend on the third-module MATLAB bipartite graph ensemble clustering for final consensus partitions; the autoencoder alone does not output cluster labels

## Evidence

- [abstract] learn feature representation via explicit modeling of synergistic optimization of the zero-inflated negative binomial reconstruction loss and denoising reconstruction loss: "learn feature representation via explicit modeling of synergistic optimization of the zero-inflated negative binomial reconstruction loss and denoising reconstruction loss"
- [intro] a single-cell dual denoising autoencoder network is proposed to project the data into a compressed low-dimensional space: "a single-cell dual denoising  autoencoder network is proposed to project the data into a compressed low-dimensional space"
- [supplementary] network.py | Single-cell Dual Denoising Autoencoder Network: "network.py   | Single-cell Dual Denoising Autoencoder Network"
- [readme] python SDDA.py --dataname Adam --highly_genes 2000 --random_seed ... --dims "256, 32" --alpha 0.001 --learning_rate 0.0001 --batch_size 256 --pretrain_epoch 1000 --noise_sd 1.5: "python SDDA.py --dataname Adam --highly_genes 2000 --random_seed "1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000" --dims "256, 32" --alpha 0.001"
