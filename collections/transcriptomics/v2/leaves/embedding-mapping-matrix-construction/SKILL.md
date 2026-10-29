---
name: embedding-mapping-matrix-construction
description: Use when you have aligned SC and ST embeddings in a shared latent space
  (e.g. from a trained STEM netE encoder) and need per-cell spatial coordinates or
  spot-to-cell correspondence probabilities.
when_to_use_negative:
- You only need spot-level deconvolution into a fixed number of cell types — the article
  notes deconvolution cannot link SC and ST at the single-cell level, but if that
  coarse representation suffices, a mapping matrix is unnecessary.
- The SC and ST embeddings have not been trained/aligned in a shared latent space
  (e.g. no MMD-based domain alignment was performed) — raw dot-products of unaligned
  embeddings will not yield meaningful mappings.
- SC and ST data are from different donors or different disease states — the article
  flags applying STEM to data from different donors in the same disease state as untested
  future work.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3516
- http://edamontology.org/topic_3170
tools:
- name: STEM
  role: Provides the SOmodel, netE encoder, and all_coord function used to compute
    embeddings and build the mapping matrices
  repo: https://github.com/WhirlFirst/STEM
- name: PyTorch
  role: Provides torch.mm for the cross dot-product of embeddings and F.softmax(dim=1)
    for row-normalized probability mapping
- name: scanpy
  role: AnnData container whose obsm['spatialDA'] slot stores the per-cell spatial
    coordinates produced from the SC-ST mapping matrix
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42003-023-05640-1
    title: STEM enables mapping of single-cell and spatial transcriptomics data with
      transfer learning
  source_content_hash: sha256:8ace128217238b843d7c03dcdb62af2bb5119fc363780568caae89eaf161b91d
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
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3516
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# embedding-mapping-matrix-construction

## Summary

Constructs ST-SC and SC-ST mapping matrices by applying F.softmax over the cross dot-product of SC and ST embeddings produced by the STEM encoder, so that probabilities per spot/cell sum to 1. These matrices transfer spatial information from spatial transcriptomics spots onto individual single cells.

## When to use

Use when you have aligned SC and ST embeddings in a shared latent space (e.g. from a trained STEM netE encoder) and need per-cell spatial coordinates or spot-to-cell correspondence probabilities.

## When NOT to use

- You only need spot-level deconvolution into a fixed number of cell types — the article notes deconvolution cannot link SC and ST at the single-cell level, but if that coarse representation suffices, a mapping matrix is unnecessary.
- The SC and ST embeddings have not been trained/aligned in a shared latent space (e.g. no MMD-based domain alignment was performed) — raw dot-products of unaligned embeddings will not yield meaningful mappings.
- SC and ST data are from different donors or different disease states — the article flags applying STEM to data from different donors in the same disease state as untested future work.

## Inputs

- scembedding: PyTorch tensor of SC cell embeddings from the trained netE encoder (n_cells x latent_dim)
- stembedding: PyTorch tensor of ST spot embeddings from the trained netE encoder (n_spots x latent_dim)
- ST spatial coordinates (spcoor, e.g. simu_st_metadata.csv with spot x/y positions)
- AnnData object for SC cells (adata) to receive the inferred spatial coordinates

## Outputs

- netst2sc: ST-SC mapping matrix (softmax over dot-products; each spot row sums to 1)
- netsc2st: SC-ST mapping matrix (softmax over dot-products; each cell row sums to 1)
- adata.obsm['spatialDA']: per-single-cell spatial coordinates derived via all_coord from netsc2st and spcoor

## How to apply

After training the STEM SOmodel and computing embeddings, set the model to evaluation mode with modeleval() and pass the normalized, log-scaled count tensors through netE on GPU to get scembedding and stembedding. Compute the cross dot-product between the two embedding sets (stembedding.mm(scembedding.t()) and scembedding.mm(stembedding.t())) and apply F.softmax(dim=1) to each so that each row sums to 1, yielding the ST-SC matrix (netst2sc, probability of one spot to other cells sums to 1) and the SC-ST matrix (netsc2st, probability of one cell to other spots sums to 1). Then call all_coord on the SC-ST matrix together with the ST spatial coordinates (spcoor) to assign every single cell a spatial coordinate stored in adata.obsm['spatialDA']. The softmax normalization is what makes rows interpretable as probabilistic mappings rather than raw similarities.

## Related tools

- **STEM** (Provides the SOmodel, netE encoder, and all_coord function used to compute embeddings and build the mapping matrices) — https://github.com/WhirlFirst/STEM
- **PyTorch** (Provides torch.mm for the cross dot-product of embeddings and F.softmax(dim=1) for row-normalized probability mapping)
- **scanpy** (AnnData container whose obsm['spatialDA'] slot stores the per-cell spatial coordinates produced from the SC-ST mapping matrix)

## Examples

```
netst2sc = F.softmax(stembedding.mm(scembedding.t()),dim=1).detach().cpu().numpy(); netsc2st = F.softmax(scembedding.mm(stembedding.t()),dim=1).detach().cpu().numpy()
```

## Evaluation signals

- Row-stochasticity invariant: in netst2sc each spot's probabilities over cells sum to 1; in netsc2st each cell's probabilities over spots sum to 1 (guaranteed by F.softmax(dim=1)).
- Matrix shape checks: netst2sc is (n_spots x n_cells) and netsc2st is (n_cells x n_spots), matching the embedding dimensions.
- adata.obsm['spatialDA'] has one coordinate per SC cell (n_cells x 2), each falling within the convex hull of the ST spot coordinates.
- Qualitative check from the README: STEM is the only method that preserves the original topology structure of all single cells — compare the reconstructed single-cell spatial layout against the reference (e.g. e1z2 figure).
- The underlying embeddings should show the loss curve from train_wholedata (400 epochs) converging before matrices are computed; matrices built from a non-converged encoder are suspect.

## Limitations

- The mapping quality depends on the shared latent space quality; the domain gap is addressed upstream by minimizing MMD (alpha=0.8 loss weight), so softmax over raw dot-products alone does not correct batch effects.
- Applying STEM (and hence this mapping) to SC and ST data from different donors in the same disease state remains untested future work.
- Coordinates assigned to cells are pseudo-spatial: each cell's location is a probability-weighted combination of ST spot coordinates, not a measured position.

## Evidence

- [methods] Compute embeddings and softmax mapping matrices: "netst2sc = F.softmax(stembedding.mm(scembedding.t()),dim=1).detach().cpu().numpy()"
- [readme] Mapping matrix semantics: "In the ST-SC mapping matrix, the probability of one spot to other cells summarizes to 1. In the SC-ST mapping matrix, the probability of one cell to other spots summarizes to 1."
- [readme] Assign per-cell spatial coordinates: "adata.obsm['spatialDA'] = all_coord(pd.DataFrame(netsc2st,index=adata.obs_names,columns=stadata.obs_names),spcoor)"
- [abstract] Purpose of embeddings for mapping: "and then uses the embeddings to infer SC-ST mapping"
- [intro] Single-cell level linkage: "Such representation cannot link the SC and ST at the single cell level and limits the potential to flexibly"
