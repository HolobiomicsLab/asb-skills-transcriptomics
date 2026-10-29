---
name: sc-st-embedding-transfer-learning
description: Use when you have SC count matrix plus ST counts with spot spatial coordinates
  and cell-type labels, and need single-cell spatial mapping, pseudo-spatial adjacency,
  or colocalization of cell types (e.g., TSKs vs endothelial cells in hSCC).
when_to_use_negative:
- The goal is classic spot deconvolution where a spot is modeled as a mixture of a
  fixed number of cell types — STEM instead transfers spatial information to individual
  cells
- SC and ST data come from different donors (even in the same disease state) — the
  paper states this remains untested future work
- The ST data lack spot spatial coordinates, so the Gaussian spatial-variance term
  and coordinate mapping cannot be defined
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: STEM
  role: Provides the SOmodel, netE encoder, all_coord utility, and softmax mapping-matrix
    construction used to transfer spatial information from ST to SC data
  repo: https://github.com/WhirlFirst/STEM
- name: scSTEM
  role: PyPI package distribution of the same STEM model (pip install scSTEM) for
    building single-cell level spatial transcriptomic landscapes
- name: PyTorch
  role: Backend for training SOmodel on CUDA GPU, computing embeddings and softmax
    dot-product mapping matrices
- name: scanpy
  role: Preprocessing of SC and ST AnnData (sc.pp.normalize_total, sc.pp.log1p) and
    QC metrics for dropout-rate estimation
provenance:
  source_task_ids:
  - task_003
  - task_007
  source_papers:
  - doi: 10.1038/s42003-023-05640-1
    title: STEM enables mapping of single-cell and spatial transcriptomics data with
      transfer learning
  source_content_hash: sha256:fa28cbfc39aea62770ea3dbf4125ee6ba95986ed562547ab1ac92884c66da21d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WhirlFirst/STEM
  tools:
  - CellTrek
  - PyTorch
  - STEM
  - Seurat
  - Spaotsc
  - Tangram
  - scSTEM
  - scSpace
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# sc-st-embedding-transfer-learning

## Summary

Train STEM's SOmodel, a deep transfer learning model, to encode single-cell RNA-seq (SC) and spatial transcriptomics (ST) counts into a shared spatially aware embedding space, then derive SC–ST mapping matrices and single-cell pseudo-spatial coordinates. It eliminates the domain gap (batch/technical bias, dropout) between SC and ST data without requiring spot deconvolution.

## When to use

Use when you have SC count matrix plus ST counts with spot spatial coordinates and cell-type labels, and need single-cell spatial mapping, pseudo-spatial adjacency, or colocalization of cell types (e.g., TSKs vs endothelial cells in hSCC).

## When NOT to use

- The goal is classic spot deconvolution where a spot is modeled as a mixture of a fixed number of cell types — STEM instead transfers spatial information to individual cells
- SC and ST data come from different donors (even in the same disease state) — the paper states this remains untested future work
- The ST data lack spot spatial coordinates, so the Gaussian spatial-variance term and coordinate mapping cannot be defined

## Inputs

- Single-cell (SC) count matrix, e.g. simu_sc_counts.csv from raw SeqFISH data, transposed to cells × genes
- Spatial transcriptomics (ST) count matrix with spot coordinates, e.g. simu_st_counts.csv plus simu_st_metadata.csv
- ST ground-truth cell-type labels (e.g. simu_st_celltype.csv) or SC cell-type annotations for downstream colocalization analyses
- Processed data/trained models from Figshare deposit DOI 10.6084/m9.figshare.24452812 for reproduction

## Outputs

- SC and ST embeddings from netE in the unified spatially aware space
- Row-normalized SC–ST mapping matrix netsc2st and ST–SC mapping matrix netst2sc (softmax rows sum to 1)
- Per-single-cell pseudo-spatial coordinates stored in adata.obsm['spatialDA']
- Training loss curve from train_wholedata (400 epochs)

## How to apply

Preprocess both datasets with scanpy (sc.pp.normalize_total then sc.pp.log1p) and compute the dropout rate dp = 1 − median(SC n_genes_by_counts)/median(ST n_genes_by_counts) via sc.pp.calculate_qc_metrics (E1z2: dp ≈ 0.584). Set sigma to half the ST spot adjacent distance, alpha (MMD weight) = 0.8, lr = 0.002, mmdbatch = 1000, and train SOmodel on GPU for 400 epochs with seed_all(2022), passing SC counts, ST counts, and spot coordinates as float tensors. Extract SC and ST embeddings with netE and build the row-normalized softmax mapping matrices netst2sc = softmax(stE·scEᵀ, dim=1) and netsc2st = softmax(scE·stEᵀ, dim=1). Assign each single cell pseudo-spatial coordinates by applying all_coord(netsc2st, spot coordinates) and write the result into adata.obsm['spatialDA']; the rationale is that the Gaussian spatial-variance term (sigma) plus MMD alignment preserves the original topology of single cells while removing technical biases.

## Related tools

- **STEM** (Provides the SOmodel, netE encoder, all_coord utility, and softmax mapping-matrix construction used to transfer spatial information from ST to SC data) — https://github.com/WhirlFirst/STEM
- **scSTEM** (PyPI package distribution of the same STEM model (pip install scSTEM) for building single-cell level spatial transcriptomic landscapes)
- **PyTorch** (Backend for training SOmodel on CUDA GPU, computing embeddings and softmax dot-product mapping matrices)
- **scanpy** (Preprocessing of SC and ST AnnData (sc.pp.normalize_total, sc.pp.log1p) and QC metrics for dropout-rate estimation)

## Examples

```
netsc2st = F.softmax(scembedding.mm(stembedding.t()),dim=1).detach().cpu().numpy()
```

## Evaluation signals

- Dropout rate matches the expected value of the ratio of median n_genes_by_counts between SC and ST data (E1z2: dp ≈ 0.5837)
- Loss curve from train_wholedata decreases over 400 epochs and converges (compare with the README's reference loss.png shape)
- Sanity check that each row of netst2sc and netsc2st sums to 1 after the softmax
- On semi-simulation E1z2, mapped coordinates should yield the lowest MAE to ground truth and roughly two-fold higher hit-number@K (K=200) than the second-best baseline among CellTrek, scSpace, Seurat, Spaotsc, Tangram; per-cell-type PCC of cell-type proportions can also be tabulated
- Verify installation by running python test.py

## Limitations

- Empirical tuning rule: sigma must be set to half the ST spot adjacent distance; other values degrade performance
- STEM's transfer across donors (SC and ST from different individuals in the same disease state) is untested future work
- Using spatially dominant genes (SDGs) of cancer cells as drug-target insights is only proposed, not demonstrated
- The dropout-rate heuristic assumes median gene-detection differences capture the SC/ST domain gap; datasets with unusual depth distributions may need re-tuning

## Evidence

- [abstract] It uses deep transfer learning to encode both ST and SC data into a unified spatially aware embedding space: [evidence span withheld — anchor status: not_found]
- [methods] dropout rate from median gene counts: "Next we calculate the ratio between the median total counts value of SC and ST data as the dropout rate."
- [readme] softmax mapping matrices: "The matrix `netst2sc` and `netsc2st` are ST-SC and SC-ST mapping matrices, respectively."
- [readme] sigma tuning rule: "Empirically we found by setting the sigma as half of the ST spot adjacent distance, STEM achieves the best performance."
- [readme] topology preservation vs baselines: "Compared with other methods, STEM is the only method that preserves the original topology structure of all single cells."
- [discussion] transfer-learning framing: "It learns spatially-aware embeddings of transcriptomic data via transfer learning."
