---
name: tensorflow-keras-network-construction
description: Use when constructing a denoising autoencoder in TensorFlow/Keras to
  learn latent embeddings from raw scRNA-seq count matrices, explicitly modeling zero
  inflation (dropout) via a zero-inflated negative binomial reconstruction loss.
when_to_use_negative:
- The input is already a low-dimensional feature/embedding table rather than a raw
  scRNA-seq count matrix requiring dropout-aware denoising
- The goal is consensus clustering of existing embeddings — that is handled by the
  separate bipartite graph ensemble clustering module (BC.m/BGEC.m), not the autoencoder
- A non-TensorFlow/Keras deep learning stack is mandated, since network.py, loss.py
  and utils.py are implemented in TensorFlow 2.6.0/Keras 2.6.0
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: scBGEDA
  role: Pipeline whose second module (SDDA.py) trains the dual denoising autoencoder
    network to produce the latent embedding
  repo: https://github.com/wangyh082/scBGEDA
- name: SDDA.py
  role: Entry point that trains the network end-to-end with Adam and saves the latent
    embedding; second module of scBGEDA
  repo: https://github.com/wangyh082/scBGEDA
- name: network.py
  role: Defines the Single-cell Dual Denoising Autoencoder network architecture (noisy
    encoder, ZINB master decoder, MSE follower decoder)
  repo: https://github.com/wangyh082/scBGEDA
- name: loss.py
  role: 'Implements the loss functions: negative-log ZINB reconstruction loss (L1)
    and squared-error denoising reconstruction loss (L2)'
  repo: https://github.com/wangyh082/scBGEDA
- name: utils.py
  role: Utility functions supporting the network (data handling, evaluation, saving
    the embedding)
  repo: https://github.com/wangyh082/scBGEDA
- name: TensorFlow
  role: Deep learning framework (version 2.6.0) in which the autoencoder network and
    losses are implemented
- name: Keras
  role: Keras 2.6.0 API used to build and train the encoder and dual decoder layers
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad075
    title: 'scBGEDA: deep single-cell clustering analysis via a dual denoising autoencoder
      with bipartite graph ensemble clustering'
  source_content_hash: sha256:62a06b8c3a7cac8ccd01faa757ef0a919c345d765268760dd2a12c47661ca6b0
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
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# tensorflow-keras-network-construction

## Summary

Build and train the single-cell Dual Denoising Autoencoder network of scBGEDA in TensorFlow/Keras: a Gaussian-noise-corrupted encoder projects scRNA-seq counts into a 32-dimensional latent space Z, with a ZINB master decoder and an MSE follower decoder trained by the combined loss L_h = lambda*L1 + gamma*L2.

## When to use

Use when constructing a denoising autoencoder in TensorFlow/Keras to learn latent embeddings from raw scRNA-seq count matrices, explicitly modeling zero inflation (dropout) via a zero-inflated negative binomial reconstruction loss.

## When NOT to use

- The input is already a low-dimensional feature/embedding table rather than a raw scRNA-seq count matrix requiring dropout-aware denoising
- The goal is consensus clustering of existing embeddings — that is handled by the separate bipartite graph ensemble clustering module (BC.m/BGEC.m), not the autoencoder
- A non-TensorFlow/Keras deep learning stack is mandated, since network.py, loss.py and utils.py are implemented in TensorFlow 2.6.0/Keras 2.6.0

## Inputs

- Preprocessed scRNA-seq count matrix (e.g. Adam dataset) reduced to highly variable genes (e.g. 2000 via --highly_genes)
- Hyperparameters: dims (hidden layer sizes, e.g. '256, 32'), alpha, learning_rate, batch_size, pretrain_epoch, noise_sd, random_seed, gpu_option

## Outputs

- 32-dimensional latent embedding matrix (latent_embedding.csv) with one embedding vector per input cell
- Trained ZINB master decoder parameters (pi, mu, theta) and trained network weights in TensorFlow/Keras format

## How to apply

Load the preprocessed count matrix (e.g. the Adam dataset reduced to 2000 highly variable genes) as input to scBGEDA's second module, then define the network in network.py: add Gaussian noise to the input, encode it through hidden layers configured via --dims '256, 32' to a compressed 32-dimensional latent Z, and attach two decoders. Implement the losses in loss.py: the master decoder estimates ZINB parameters (dropout probability pi, mean mu, dispersion theta) and is optimized by a negative-log ZINB loss (L1), while the follower decoder approximates mu via a squared Frobenius-norm denoising loss (L2). Combine them as L_h = lambda*(-log(ZINB(X|pi,mu,theta))) + gamma*(||mu - f_W2'(Z)||_F^2), where the balance is controlled by the --alpha hyperparameter. Train end-to-end by running SDDA.py with the Adam optimizer (--learning_rate 0.0001, --batch_size 256, --pretrain_epoch 1000, --noise_sd 1.5) until the combined loss converges, then extract and save the 32-dimensional latent embedding for all cells as latent_embedding.csv.

## Related tools

- **scBGEDA** (Pipeline whose second module (SDDA.py) trains the dual denoising autoencoder network to produce the latent embedding) — https://github.com/wangyh082/scBGEDA
- **SDDA.py** (Entry point that trains the network end-to-end with Adam and saves the latent embedding; second module of scBGEDA) — https://github.com/wangyh082/scBGEDA
- **network.py** (Defines the Single-cell Dual Denoising Autoencoder network architecture (noisy encoder, ZINB master decoder, MSE follower decoder)) — https://github.com/wangyh082/scBGEDA
- **loss.py** (Implements the loss functions: negative-log ZINB reconstruction loss (L1) and squared-error denoising reconstruction loss (L2)) — https://github.com/wangyh082/scBGEDA
- **utils.py** (Utility functions supporting the network (data handling, evaluation, saving the embedding)) — https://github.com/wangyh082/scBGEDA
- **TensorFlow** (Deep learning framework (version 2.6.0) in which the autoencoder network and losses are implemented)
- **Keras** (Keras 2.6.0 API used to build and train the encoder and dual decoder layers)

## Examples

```
python SDDA.py --dataname Adam --highly_genes 2000 --dims "256, 32" --alpha 0.001 --learning_rate 0.0001 --batch_size 256 --pretrain_epoch 1000 --noise_sd 1.5 --gpu_option "2"
```

## Evaluation signals

- SDDA.py executes without error on the deposited dataset (e.g. the Adam dataset with 2000 highly variable genes)
- latent_embedding.csv contains exactly one 32-dimensional embedding vector per input cell
- Both the ZINB reconstruction loss (L1) and the denoising reconstruction loss (L2) decrease monotonically (or near-monotonically) over training epochs toward convergence
- The latent dimensionality matches the last entry of the --dims argument (e.g. 32 from '256, 32')
- The Gaussian noise corruption is applied with the specified --noise_sd (e.g. 1.5), so reconstructions recover denoised estimates of the raw counts

## Limitations

- The ZINB loss assumes scRNA-seq counts follow a zero-inflated negative binomial distribution; the approach targets dropout-heavy count data and raw counts rather than already-normalized feature tables
- The network hyperparameters (dims, alpha, learning_rate 0.0001, batch_size 256, pretrain_epoch 1000, noise_sd 1.5) have defaults tuned per dataset; the README states parameters 'can be set as you like', so defaults may need retuning for new datasets
- Training requires a conda environment with pinned versions (python 3.6+, tensorflow 2.6.0, keras 2.6.0, numpy 1.19.5, scipy 1.5.4, scanpy 1.7.2); version mismatches are a failure mode
- The 32-dimensional latent space is a specific design choice; other compressed dimensionalities are not validated by the article

## Evidence

- [intro] First, a single-cell dual denoising autoencoder network is proposed to project the data into a compressed low-dimensional space and that can learn feature representation via explicit modeling of synergistic (interactions).: "First, a single-cell dual denoising autoencoder network is proposed to project the data into a compressed low-dimensional space and that can learn feature representation via explicit modeling of"
- [abstract] learn feature representation via explicit modeling of synergistic optimization of the zero-inflated negative binomial reconstruction loss and denoising reconstruction loss: "learn feature representation via explicit modeling of synergistic optimization of the zero-inflated negative binomial reconstruction loss and denoising reconstruction loss"
- [supplementary] python SDDA.py --dataname Adam --highly_genes 2000 --random_seed "1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000": "python SDDA.py --dataname Adam --highly_genes 2000 --random_seed "1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000""
- [readme] network.py   | Single-cell Dual Denoising Autoencoder Network: "network.py   | Single-cell Dual Denoising Autoencoder Network"
- [supplementary] The real single-cell RNA-seq datasets and source codes for our analyses are freely available at (https://figshare.com/articles/software/scBGEDA/19657911).: "The real single-cell RNA-seq datasets and source codes for our analyses are freely available at (https://figshare.com/articles/software/scBGEDA/19657911)."
