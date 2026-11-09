---
name: mmd-loss-transfer-learning-model-training
description: Use when you need to align single-cell RNA-seq and spatial transcriptomics
  count data into a unified embedding space via deep transfer learning — e.g., before
  computing SC-ST mapping matrices or pseudo-spatial adjacency for single-cell spatial
  landscapes.
when_to_use_negative:
- Input matrices have not been normalize_total + log1p transformed — the encoder and
  dropout-rate correction assume this preprocessing
- ST and SC data come from different donors or different disease states — the article
  states applying STEM across donors in the same disease state remains untested future
  work
- The goal is deconvolving spots into a fixed number of cell types rather than single-cell
  mapping — the article notes such deconvolution cannot link SC and ST at the single-cell
  level
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: STEM
  role: Provides the SOmodel, netE encoder, train_wholedata, and MMD loss implementation
    used for transfer learning training
  repo: https://github.com/WhirlFirst/STEM
- name: scSTEM
  role: PyPI package distribution of STEM installable via pip install scSTEM
- name: PyTorch
  role: Deep learning backend for SOmodel training on cuda:0, tensor construction,
    and F.softmax embedding operations
  repo: https://github.com/WhirlFirst/STEM
- name: scanpy
  role: Performs normalize_total, log1p, and calculate_qc_metrics preprocessing needed
    to build training tensors and the dropout rate
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42003-023-05640-1
    title: STEM enables mapping of single-cell and spatial transcriptomics data with
      transfer learning
  source_content_hash: sha256:96ea6a32d6bbf7a70263d07e136952c942337c662eccdb50fcfd4a6a3ea73f76
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WhirlFirst/STEM
  tools:
  - PyTorch
  - STEM
  - scSTEM
  - scanpy
  - torchvision
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# mmd-loss-transfer-learning-model-training

## Summary

Train the STEM transfer learning model (SOmodel in PyTorch) that encodes SC and ST count matrices into a shared 128-d latent space by combining a spatial Gaussian loss with an MMD (maximum mean discrepancy) loss that eliminates the SC-ST domain gap. This produces the encoder (netE) whose embeddings are later used to build SC-ST mapping matrices.

## When to use

Use when you need to align single-cell RNA-seq and spatial transcriptomics count data into a unified embedding space via deep transfer learning — e.g., before computing SC-ST mapping matrices or pseudo-spatial adjacency for single-cell spatial landscapes.

## When NOT to use

- Input matrices have not been normalize_total + log1p transformed — the encoder and dropout-rate correction assume this preprocessing
- ST and SC data come from different donors or different disease states — the article states applying STEM across donors in the same disease state remains untested future work
- The goal is deconvolving spots into a fixed number of cell types rather than single-cell mapping — the article notes such deconvolution cannot link SC and ST at the single-cell level

## Inputs

- Normalized, log1p-transformed SC count matrix as a float32 torch tensor (e.g., SeqFISH E1z2 simu_sc_counts.csv, transposed, 351 genes)
- Normalized, log1p-transformed ST count matrix as a float32 torch tensor (simu_st_counts.csv)
- ST spot spatial coordinate matrix as a float32 tensor (simu_st_metadata.csv)
- Computed dropout rate dp (via scanpy calculate_qc_metrics medians)

## Outputs

- Trained SOmodel state (saved to outf folder 'log/test')
- SC embeddings from netE on GPU (one vector per cell in the shared latent space)
- ST embeddings from netE (one vector per spot in the shared latent space)
- Training loss curve (exported as loss.png)

## How to apply

First preprocess both inputs with scanpy: sc.pp.normalize_total then sc.pp.log1p on each AnnData, and compute the ST dropout rate as dp = 1 - median(n_genes_by_counts_SC)/median(n_genes_by_counts_ST) using sc.pp.calculate_qc_metrics (0.5837 for the E1z2 SeqFISH simulation). Instantiate a settings object with n_genes=351 (number of input genes), sigma=3 (spatial variance of the Gaussian, set empirically to half the ST spot adjacent distance), alpha=0.8 (MMD loss weight), lr=0.002, mmdbatch=1000, and device cuda:0, calling seed_all(2022) for reproducibility. Train the SOmodel end-to-end with train_wholedata for 400 epochs on the normalized, log-scaled SC and ST tensors plus spatial coordinates, recording the loss curve. After training, call modeleval() and compute SC and ST embeddings through the netE encoder on GPU; the MMD term minimizes the domain gap between the SC and ST embedding distributions while the Gaussian spatial term ties ST embeddings to coordinates.

## Related tools

- **STEM** (Provides the SOmodel, netE encoder, train_wholedata, and MMD loss implementation used for transfer learning training) — https://github.com/WhirlFirst/STEM
- **scSTEM** (PyPI package distribution of STEM installable via pip install scSTEM)
- **PyTorch** (Deep learning backend for SOmodel training on cuda:0, tensor construction, and F.softmax embedding operations) — https://github.com/WhirlFirst/STEM
- **scanpy** (Performs normalize_total, log1p, and calculate_qc_metrics preprocessing needed to build training tensors and the dropout rate)

## Examples

```
testmodel = SOmodel(opt); testmodel.togpu(); loss_curve = testmodel.train_wholedata(400, torch.tensor(scdata.values).float(), torch.tensor(stdata.values).float(), torch.tensor(spcoor.values).float())
```

## Evaluation signals

- Loss curve from train_wholedata decreases over 400 epochs and matches the expected loss.png shape from the README
- Dropout rate matches the expected median-ratio value (e.g., 0.5836734693877551 for E1z2)
- After modeleval(), SC and ST embeddings both come from the same netE encoder and lie in a shared latent space; cross dot-products followed by F.softmax(dim=1) yield rows summing to 1
- Reproducibility: seed_all(2022) is called before instantiating the model, so retraining should reproduce embeddings
- Downstream SC-ST mapping preserves the original topology structure of all single cells, the property the paper claims distinguishes STEM

## Limitations

- Applying STEM to SC and ST data from different donors in the same disease state remains untested future work
- sigma must be tuned empirically — the README states setting it as half the ST spot adjacent distance gives best performance
- alpha (MMD loss weight, default 0.8) is a fixed empirical hyperparameter; untested behavior outside the demonstrated settings

## Evidence

- [results] STEM's domain alignment module eliminates the domain gap between SC and ST embeddings by minimizing the maximum mean discrepancy (MMD): [evidence span withheld — anchor status: not_found]
- [readme] Empirically we found by setting the sigma as half of the ST spot adjacent distance, STEM achieves the best performance.: "Empirically we found by setting the sigma as half of the ST spot adjacent distance, STEM achieves the best performance."
- [readme] dp = 1- adata.obs['n_genes_by_counts'].median()/stadata.obs['n_genes_by_counts'].median(): "dp = 1- adata.obs['n_genes_by_counts'].median()/stadata.obs['n_genes_by_counts'].median()"
- [readme] loss_curve = testmodel.train_wholedata(400,torch.tensor(scdata.values).float(),torch.tensor(stdata.values).float(),torch.tensor(spcoor.values).float()): "loss_curve = testmodel.train_wholedata(400,torch.tensor(scdata.values).float(),torch.tensor(stdata.values).float(),torch.tensor(spcoor.values).float())"
- [abstract] It uses deep transfer learning to encode both ST and SC data into a unified spatially aware embedding space: "It uses deep transfer learning to encode both ST and SC data into a unified spatially aware embedding space"
