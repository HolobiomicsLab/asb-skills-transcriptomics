---
name: denoising-autoencoder-training
description: Use when you need a denoised, low-dimensional latent embedding of a scRNA-seq
  count matrix for downstream cell clustering, especially with dropout-heavy zero-inflated
  counts.
when_to_use_negative:
- Input is not a raw scRNA-seq count matrix (e.g. already-normalized feature tables
  or bulk RNA-seq), since the ZINB likelihood models zero-inflated single-cell counts.
- The goal is consensus clustering itself rather than latent feature learning — use
  the bipartite graph ensemble clustering module (BC.m/BGEC.m) on the learned embedding
  instead.
- Only small numbers of genes are of interest and dimensionality reduction to a compressed
  latent space is unnecessary.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: scBGEDA
  role: Overall pipeline whose second module (SDDA) trains the dual denoising autoencoder
  repo: https://github.com/wangyh082/scBGEDA
- name: SDDA.py
  role: Entry script that trains the single-cell Dual Denoising Autoencoder end-to-end
  repo: https://github.com/wangyh082/scBGEDA
- name: network.py
  role: Defines the noise-corrupted encoder and dual decoder architecture
  repo: https://github.com/wangyh082/scBGEDA
- name: loss.py
  role: Implements the ZINB negative-log likelihood and MSE denoising loss functions
  repo: https://github.com/wangyh082/scBGEDA
- name: preprocess.py
  role: First-module preprocessing selecting highly variable genes before training
  repo: https://github.com/wangyh082/scBGEDA
- name: TensorFlow/Keras
  role: Framework used to build and train the autoencoder network
provenance:
  source_task_ids:
  - task_007
  - task_002
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad075
    title: 'scBGEDA: deep single-cell clustering analysis via a dual denoising autoencoder
      with bipartite graph ensemble clustering'
  source_content_hash: sha256:79ede9be5038374585386d54a29e7512493bed3e5e79cb1ba04767bfdf4f1a16
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wangyh082/scBGEDA
  tools:
  - BC.m
  - MATLAB
  - NumPy
  - Python
  - SDDA.py
  - SciPy
  - TensorFlow/Keras
  - computeNMI.m
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
yes_rate: 0.5
n_claims: 20
---

# denoising-autoencoder-training

## Summary

Train scBGEDA's single-cell dual denoising autoencoder (SDDA), which corrupts scRNA-seq counts with Gaussian noise and learns a compressed latent representation via a ZINB master decoder (estimating pi, mu, theta) plus an MSE follower decoder, jointly optimized as lambda*(-log ZINB) + gamma*MSE.

## When to use

Use when you need a denoised, low-dimensional latent embedding of a scRNA-seq count matrix for downstream cell clustering, especially with dropout-heavy zero-inflated counts.

## When NOT to use

- Input is not a raw scRNA-seq count matrix (e.g. already-normalized feature tables or bulk RNA-seq), since the ZINB likelihood models zero-inflated single-cell counts.
- The goal is consensus clustering itself rather than latent feature learning — use the bipartite graph ensemble clustering module (BC.m/BGEC.m) on the learned embedding instead.
- Only small numbers of genes are of interest and dimensionality reduction to a compressed latent space is unnecessary.

## Inputs

- preprocessed scRNA-seq count matrix restricted to top 2000 highly variable genes (e.g. Adam dataset from the figshare deposit)
- hyperparameters: dataname, highly_genes, random_seed, dims, alpha, learning_rate, batch_size, pretrain_epoch, noise_sd, gpu_option

## Outputs

- 32-dimensional latent embedding matrix (one vector per cell) saved as latent_embedding.csv
- training loss curves for the ZINB reconstruction loss and MSE denoising reconstruction loss

## How to apply

Preprocess the raw scRNA-seq count matrix (e.g. the Adam dataset) with preprocess.py, selecting the top 2000 highly variable genes. Build the network in network.py: a Gaussian-noise-corrupted encoder (noise_sd ~1.5) projecting cells into a 32-dimensional latent space Z (dims '256, 32'). Implement loss.py so a ZINB master decoder minimizes the negative-log ZINB likelihood on raw counts while an MSE follower decoder approximates the denoised mean mu; combine as L_h = lambda*L1 + gamma*L2 (alpha controls relative decoder impact, default 0.001). Train end-to-end with SDDA.py using the Adam optimizer (learning_rate 0.0001, batch_size 256, pretrain_epoch 1000) until both loss terms converge, then save the 32-dimensional latent embedding matrix per cell for the subsequent bipartite graph ensemble clustering module (BC.m).

## Related tools

- **scBGEDA** (Overall pipeline whose second module (SDDA) trains the dual denoising autoencoder) — https://github.com/wangyh082/scBGEDA
- **SDDA.py** (Entry script that trains the single-cell Dual Denoising Autoencoder end-to-end) — https://github.com/wangyh082/scBGEDA
- **network.py** (Defines the noise-corrupted encoder and dual decoder architecture) — https://github.com/wangyh082/scBGEDA
- **loss.py** (Implements the ZINB negative-log likelihood and MSE denoising loss functions) — https://github.com/wangyh082/scBGEDA
- **preprocess.py** (First-module preprocessing selecting highly variable genes before training) — https://github.com/wangyh082/scBGEDA
- **TensorFlow/Keras** (Framework used to build and train the autoencoder network)

## Examples

```
python SDDA.py --dataname Adam --highly_genes 2000 --random_seed "1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000" --dims "256, 32" --alpha 0.001 --learning_rate 0.0001 --batch_size 256 --pretrain_epoch 1000 --noise_sd 1.5
```

## Evaluation signals

- SDDA.py executes without error on the deposited dataset with the chosen random seeds (1111–10000).
- latent_embedding.csv contains exactly one 32-dimensional embedding vector per input cell.
- Both the ZINB (L1) and MSE denoising (L2) loss terms decrease monotonically over training epochs.
- Downstream bipartite graph ensemble clustering (BC.m) yields higher ARI/NMI than state-of-the-art baselines, and the full dual loss outperforms the ZINB-only ablation on most of the 20 datasets.
- Changing alpha (decoder weighting) or removing the MSE term produces a measurable ARI difference, confirming both decoders contribute.

## Limitations

- Ablation shows replacing ZINB with a plain NB loss yields comparable (slightly higher on average) NMI/ARI on 11 of the 20 datasets, so ZINB modeling is not strictly necessary for clustering performance.
- The architecture and default hyperparameters (dims '256, 32', alpha 0.001, noise_sd 1.5, pretrain_epoch 1000) were tuned for scRNA-seq datasets; transfer to other data types is untested in the article.
- Result quality depends on the preprocessing choices (2000 highly variable genes, cell-subset random seeds for fairness).

## Evidence

- [supplementary] Gaussian noise, ZINB master decoder, MSE follower decoder: [evidence span withheld — anchor status: not_found]
- [results] MSE loss contributes to clustering: [evidence span withheld — anchor status: not_found]
- [abstract] synergistic dual loss design: "learn feature representation via explicit modeling of synergistic optimization of the zero-inflated negative binomial reconstruction loss and denoising reconstruction loss"
- [readme] default hyperparameters from README: "--dims "256, 32" --alpha 0.001 --learning_rate 0.0001 --batch_size 256 --pretrain_epoch 1000 --noise_sd 1.5"
