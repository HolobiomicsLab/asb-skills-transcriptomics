---
name: loss-ablation-study-design
description: Use when a model's training objective combines multiple loss terms (e.g.
  scBGEDA's zero-inflated negative binomial reconstruction loss plus an MSE denoising
  reconstruction loss) and you need to isolate whether a specific term improves downstream
  performance such as cell-type clustering ARI.
when_to_use_negative:
- When the model's objective has only a single loss term — there is no component to
  ablate and the comparison degenerates to the full model against itself.
- When the question is whether the overall method beats baselines (state-of-the-art
  comparison experiments) rather than attributing performance to a specific loss component.
- When computational budget does not permit retraining all configurations across the
  ten seeds with identical hyperparameters — an uncontrolled ablation with differing
  seeds or parameters confounds the loss-term effect.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: scBGEDA
  role: Full pipeline whose dual-loss autoencoder is the subject of the ablation (full
    Eq. 8 loss vs ZINB-only variant)
  repo: https://github.com/wangyh082/scBGEDA
- name: SDDA.py
  role: Trains the Single-cell Dual Denoising Autoencoder for both configurations,
    with the MSE follower-decoder term removed in the ablated variant
  repo: https://github.com/wangyh082/scBGEDA
- name: loss.py
  role: Contains the loss functions of the network; the ZINB and MSE denoising terms
    are edited here for the ablation
  repo: https://github.com/wangyh082/scBGEDA
- name: preprocess.py
  role: Preprocesses each dataset with the top 2000 highly variable genes and seeds
    1111–10000 before training either configuration
  repo: https://github.com/wangyh082/scBGEDA
- name: BC.m
  role: Runs bipartite graph ensemble clustering on the latent embeddings of both
    loss configurations under MATLAB
  repo: https://github.com/wangyh082/scBGEDA
- name: rand_index.m
  role: Computes ARI values after clustering against ground-truth labels for each
    dataset, configuration, and seed
  repo: https://github.com/wangyh082/scBGEDA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad075
    title: 'scBGEDA: deep single-cell clustering analysis via a dual denoising autoencoder
      with bipartite graph ensemble clustering'
  source_content_hash: sha256:d1d162da627884339a618f986e147dd1df27b11e1b5129cfdce32a113f066040
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wangyh082/scBGEDA
  tools:
  - BC.m
  - MATLAB
  - Python
  - SDDA.py
  - loss.py
  - preprocess.py
  - rand_index.m
  - scBGEDA
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# loss-ablation-study-design

## Summary

Design and execute a loss-term ablation for a multi-component training objective — here, removing the MSE follower-decoder denoising term from scBGEDA's dual ZINB+MSE loss — and compare clustering performance (ARI) across datasets and seeds to quantify each loss term's contribution.

## When to use

Use when a model's training objective combines multiple loss terms (e.g. scBGEDA's zero-inflated negative binomial reconstruction loss plus an MSE denoising reconstruction loss) and you need to isolate whether a specific term improves downstream performance such as cell-type clustering ARI.

## When NOT to use

- When the model's objective has only a single loss term — there is no component to ablate and the comparison degenerates to the full model against itself.
- When the question is whether the overall method beats baselines (state-of-the-art comparison experiments) rather than attributing performance to a specific loss component.
- When computational budget does not permit retraining all configurations across the ten seeds with identical hyperparameters — an uncontrolled ablation with differing seeds or parameters confounds the loss-term effect.

## Inputs

- Real scRNA-seq datasets (20 datasets) downloaded from the figshare deposit https://figshare.com/articles/software/scBGEDA/19657911
- Preprocessed expression matrices with top 2000 highly variable genes (preprocess.py output)
- Ground-truth cell-type labels per dataset for ARI computation
- Model source files: SDDA.py, network.py, loss.py (loss definitions to be modified for ablation)

## Outputs

- Latent embeddings from both loss configurations (full dual loss vs ZINB-only) across ten seeds per dataset
- ARI values per dataset, configuration, and seed (rand_index.m)
- Ablation comparison table of mean ARI per dataset (ablation_ari_comparison.csv)

## How to apply

Train the full model with the complete dual loss — scBGEDA's Single-cell Dual Denoising Autoencoder (SDDA.py, network.py, loss.py) jointly optimizing the ZINB reconstruction loss and the MSE denoising reconstruction loss — then retrain with the target term (the MSE follower-decoder loss, i.e. Eq. 8's denoising component) removed while holding all other hyperparameters identical: 2000 highly variable genes, dims '256, 32', alpha 0.001, learning_rate 0.0001, batch_size 256, pretrain_epoch 1000, noise_sd 1.5, and the ten seeds '1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000'. Run the downstream clustering stage unchanged (bipartite graph ensemble clustering via BC.m with ensemble size 100, k range 2–40) on both configurations' latent embeddings, then compute ARI against ground-truth labels with rand_index.m per dataset, configuration, and seed. Aggregate mean ARI per dataset into a comparison table (e.g. ablation_ari_comparison.csv) and judge the ablated term's contribution by whether the full-loss configuration wins on most datasets — the validation trend matching Fig. 3B and Supplementary Table S20. Controlling seeds and hyperparameters across configurations is the rationale: it isolates the loss term as the only variable, and the multi-seed design guards against randomness in cell selection during training.

## Related tools

- **scBGEDA** (Full pipeline whose dual-loss autoencoder is the subject of the ablation (full Eq. 8 loss vs ZINB-only variant)) — https://github.com/wangyh082/scBGEDA
- **SDDA.py** (Trains the Single-cell Dual Denoising Autoencoder for both configurations, with the MSE follower-decoder term removed in the ablated variant) — https://github.com/wangyh082/scBGEDA
- **loss.py** (Contains the loss functions of the network; the ZINB and MSE denoising terms are edited here for the ablation) — https://github.com/wangyh082/scBGEDA
- **preprocess.py** (Preprocesses each dataset with the top 2000 highly variable genes and seeds 1111–10000 before training either configuration) — https://github.com/wangyh082/scBGEDA
- **BC.m** (Runs bipartite graph ensemble clustering on the latent embeddings of both loss configurations under MATLAB) — https://github.com/wangyh082/scBGEDA
- **rand_index.m** (Computes ARI values after clustering against ground-truth labels for each dataset, configuration, and seed) — https://github.com/wangyh082/scBGEDA

## Examples

```
python SDDA.py --dataname Adam --highly_genes 2000 --random_seed "1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000" --dims "256, 32" --alpha 0.001 --learning_rate 0.0001 --batch_size 256 --pretrain_epoch 1000 --noise_sd 1.5 --gpu_option "2"
```

## Evaluation signals

- The full dual-loss configuration (ZINB + MSE) attains higher ARI than the ZINB-only configuration on most of the 20 datasets, matching the trend of Fig. 3B and Supplementary Table S20.
- Both configurations were trained with identical hyperparameters (dims '256, 32', alpha 0.001, learning_rate 0.0001, batch_size 256, pretrain_epoch 1000, noise_sd 1.5) and the same ten seeds 1111–10000; any differing run is invalid.
- The comparison table (ablation_ari_comparison.csv) contains a mean ARI entry for every dataset × configuration cell, with per-seed ARI values reproducible via rand_index.m.
- Downstream clustering for both configurations uses the same BC.m settings (ensemble size 100, bcsLowK 2, bcsUpK 40) so only the loss term differs.
- ARI is computed against ground-truth cell labels, not against the other configuration's cluster assignments.

## Limitations

- The finding is a majority trend across the 20 real scRNA-seq datasets, not a per-dataset guarantee — the MSE term may not improve ARI on every dataset.
- Conclusions depend on the specific hyperparameters and seeds used (Adam-style example settings); the ablation outcome may shift under different alpha, noise_sd, or architecture dims.
- ARI is the evaluation metric; other clustering metrics (e.g. NMI via computeNMI.m) may rank the two configurations differently.

## Evidence

- [abstract] Dual loss synergy claim: "learn feature representation via explicit modeling of synergistic optimization of the zero-inflated negative binomial reconstruction loss and denoising reconstruction loss"
- [readme] Seeds and preprocessing rationale: "--random_seed RANDOM_SEED The random seeds which are used to select the cells randomly to ensure the fairness"
- [readme] Concrete example invocation with hyperparameters: "python SDDA.py --dataname Adam --highly_genes 2000 --random_seed "1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000" --dims "256, 32" --alpha 0.001 --learning_rate 0.0001 --batch_size 256"
- [readme] Downstream ensemble clustering invocation: "matlab -r "BC 'Adam' 100 2 40 [1111, 2222, 3333, 4444, 5555, 6666, 7777, 8888, 9999, 10000]""
- [supplementary] ARI metric definition: "rand_index.m | Computing ARI values after clustering"
