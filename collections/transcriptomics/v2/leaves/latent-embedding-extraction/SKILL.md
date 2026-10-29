---
name: latent-embedding-extraction
description: Use when raw scRNA-seq count matrices must be compressed into a low-dimensional
  feature representation for downstream clustering, explicitly modeling zero inflation
  via a ZINB likelihood.
when_to_use_negative:
- The goal is only preprocessing or quality control of raw counts — that belongs to
  the first module (preprocess.py), not the autoencoder
- A consensus clustering of existing embeddings is already available and only the
  third module (BC.m / BGEC.m) is needed
- The data lack the excess zeros characteristic of scRNA-seq, making ZINB modeling
  inappropriate
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: SDDA.py
  role: End-to-end training script for the second module of scBGEDA; runs the dual
    denoising autoencoder and saves the latent embedding
  repo: https://github.com/wangyh082/scBGEDA
- name: network.py
  role: Defines the Single-cell Dual Denoising Autoencoder Network (noise-corrupted
    encoder, ZINB master decoder, MSE follower decoder)
  repo: https://github.com/wangyh082/scBGEDA
- name: loss.py
  role: Implements the ZINB negative-log reconstruction loss (L1) and denoising MSE
    loss (L2)
  repo: https://github.com/wangyh082/scBGEDA
- name: utils.py
  role: Utility functions supporting the network
  repo: https://github.com/wangyh082/scBGEDA
- name: preprocess.py
  role: First-module preprocessing that supplies the highly-variable-gene count matrix
    to the autoencoder
  repo: https://github.com/wangyh082/scBGEDA
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad075
    title: 'scBGEDA: deep single-cell clustering analysis via a dual denoising autoencoder
      with bipartite graph ensemble clustering'
  source_content_hash: sha256:5f89b5e2df835a4622d8d09de4c484520a14d250401f1ce8221b97ced41eaaf9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wangyh082/scBGEDA
  tools:
  - NumPy
  - Python
  - SDDA.py
  - SciPy
  - loss.py
  - network.py
  - preprocess.py
  - scBGEDA
  - utils.py
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5882352941176471
n_claims: 17
---

# latent-embedding-extraction

## Summary

Trains the single-cell Dual Denoising Autoencoder (SDDA) of scBGEDA — a Gaussian-noise-corrupted encoder with a ZINB master decoder and an MSE follower decoder — to produce a 32-dimensional latent embedding of scRNA-seq count data.

## When to use

Use when raw scRNA-seq count matrices must be compressed into a low-dimensional feature representation for downstream clustering, explicitly modeling zero inflation via a ZINB likelihood.

## When NOT to use

- The goal is only preprocessing or quality control of raw counts — that belongs to the first module (preprocess.py), not the autoencoder
- A consensus clustering of existing embeddings is already available and only the third module (BC.m / BGEC.m) is needed
- The data lack the excess zeros characteristic of scRNA-seq, making ZINB modeling inappropriate

## Inputs

- Preprocessed scRNA-seq count matrix (e.g. Adam dataset, 2000 highly variable genes)
- CLI configuration: dataname, highly_genes, random_seed, dims, alpha, learning_rate, batch_size, pretrain_epoch, noise_sd, gpu_option

## Outputs

- latent_embedding.csv: 32-dimensional latent embedding matrix with one vector per cell
- Training curves showing both the ZINB (L1) and denoising (L2) loss terms over epochs

## How to apply

Load the preprocessed scRNA-seq count matrix (e.g. the Adam dataset restricted to 2000 highly variable genes via preprocess.py) as input to the second module of scBGEDA. Build the network (network.py) in TensorFlow/Keras: corrupt the input with Gaussian noise (noise_sd, e.g. 1.5), encode to a compressed 32-dimensional latent Z (dims '256, 32'), then use a master ZINB decoder estimating dropout probability pi, mean mu, and dispersion theta optimized by negative-log ZINB loss (L1), plus a follower MSE decoder approximating mu (L2). Combine losses as L_h = lambda*L1 + gamma*L2 with the balance controlled by the --alpha hyperparameter (e.g. 0.001), and train end-to-end with the Adam optimizer (learning_rate 0.0001, batch_size 256, pretrain_epoch 1000) by running SDDA.py. Finally, extract the learned latent embedded space and save one 32-dimensional embedding vector per cell as latent_embedding.csv for the downstream bipartite graph ensemble clustering (BC.m) module.

## Related tools

- **SDDA.py** (End-to-end training script for the second module of scBGEDA; runs the dual denoising autoencoder and saves the latent embedding) — https://github.com/wangyh082/scBGEDA
- **network.py** (Defines the Single-cell Dual Denoising Autoencoder Network (noise-corrupted encoder, ZINB master decoder, MSE follower decoder)) — https://github.com/wangyh082/scBGEDA
- **loss.py** (Implements the ZINB negative-log reconstruction loss (L1) and denoising MSE loss (L2)) — https://github.com/wangyh082/scBGEDA
- **utils.py** (Utility functions supporting the network) — https://github.com/wangyh082/scBGEDA
- **preprocess.py** (First-module preprocessing that supplies the highly-variable-gene count matrix to the autoencoder) — https://github.com/wangyh082/scBGEDA

## Examples

```
python SDDA.py --dataname Adam --highly_genes 2000 --dims "256, 32" --alpha 0.001 --learning_rate 0.0001 --batch_size 256 --pretrain_epoch 1000 --noise_sd 1.5
```

## Evaluation signals

- SDDA.py executes without error on the deposited dataset (e.g. Adam with --highly_genes 2000)
- latent_embedding.csv contains exactly one 32-dimensional embedding vector per input cell (dims '256, 32')
- Both the ZINB negative-log likelihood term (L1) and the MSE denoising term (L2) decrease over training epochs until the combined loss L_h = lambda*L1 + gamma*L2 converges
- The learned latent space feeds BC.m and yields reported clustering performance with median ARI and NMI values across the ten random seeds (1111–10000)

## Limitations

- Layer dimensionality, alpha, noise_sd, learning rate, batch size, and pretrain epochs are dataset-dependent hyperparameters; defaults may not transfer without tuning
- Fairness of comparisons depends on the cell-sampling random seeds ('1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000'); single-seed runs may not reproduce reported median ARI/NMI
- Specific names of the fifteen benchmark scRNA-seq datasets and the compared state-of-the-art methods are not detailed in the extracted context, limiting direct benchmarking

## Evidence

- [abstract] learn feature representation via explicit modeling of synergistic optimization of the zero-inflated negative binomial reconstruction loss and denoising reconstruction loss: "learn feature representation via explicit modeling of synergistic optimization of the zero-inflated negative binomial reconstruction loss and denoising reconstruction loss"
- [intro] a single-cell dual denoising autoencoder network is proposed to project the data into a compressed low-dimensional space: "First, a single-cell dual denoising  autoencoder network is proposed to project the data into a compressed low-dimensional space"
- [supplementary] SDDA.py      | Function of the second module of scBGEDA: "SDDA.py      | Function of the second module of scBGEDA"
- [supplementary] python SDDA.py --dataname Adam --highly_genes 2000 --random_seed "1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000": "python SDDA.py --dataname Adam --highly_genes 2000 --random_seed "1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000""
- [readme] network.py   | Single-cell Dual Denoising Autoencoder Network: "network.py   | Single-cell Dual Denoising Autoencoder Network"
