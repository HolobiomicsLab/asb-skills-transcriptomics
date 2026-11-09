---
name: single-cell-to-spot-mapping-matrix-inference
description: Use when spot-level spatial transcriptomics data (e.g., MERFISH) must
  be deconvolved to single-cell resolution by mapping each SC profile onto ST spots,
  after both datasets share a trained embedding space.
when_to_use_negative:
- ST data is already at single-cell resolution, so no SC-to-spot assignment is needed
- SC and ST data come from different donors or different disease states — the article
  states applying STEM across donors in the same disease state remains untested future
  work
- Only spot-level deconvolution into a fixed number of cell types is required (deconvolution
  methods handle this; STEM's mapping matrix targets single-cell level assignment)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: STEM
  role: Provides SOmodel/netE encoder and all_coord utility used to compute embeddings,
    the softmax mapping matrices, and single-cell coordinates
  repo: https://github.com/WhirlFirst/STEM
- name: PyTorch
  role: Executes the embedding inner product and F.softmax mapping matrix computation
    on CUDA
- name: scanpy
  role: Holds the AnnData objects whose obsm['spatialDA'] stores the reconstructed
    coordinates and whose normalization precedes mapping
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-023-05640-1
    title: STEM enables mapping of single-cell and spatial transcriptomics data with
      transfer learning
  source_content_hash: sha256:d31617edd2af04726c08a7aad2d1eddebe6fc1d4238a5899609388d9a8ae181b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WhirlFirst/STEM
  tools:
  - PyTorch
  - STEM
  - scSTEM
  - scanpy
  - seaborn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# single-cell-to-spot mapping matrix inference

## Summary

Infers the cell-to-spot (SC-ST) mapping matrix `netsc2st` from STEM's spatially aware embeddings by applying softmax over the inner product of SC and ST embeddings, assigning each single cell a probability distribution over ST spots. This is the step that converts spot-level spatial information into single-cell spatial coordinates.

## When to use

Use when spot-level spatial transcriptomics data (e.g., MERFISH) must be deconvolved to single-cell resolution by mapping each SC profile onto ST spots, after both datasets share a trained embedding space.

## When NOT to use

- ST data is already at single-cell resolution, so no SC-to-spot assignment is needed
- SC and ST data come from different donors or different disease states — the article states applying STEM across donors in the same disease state remains untested future work
- Only spot-level deconvolution into a fixed number of cell types is required (deconvolution methods handle this; STEM's mapping matrix targets single-cell level assignment)

## Inputs

- Trained STEM SOmodel (netE encoder) with CUDA device set
- Normalized, log1p-transformed SC count matrix (e.g., SMART-seq MTG donor H18) as float torch tensor
- Normalized, log1p-transformed ST count matrix (e.g., MERFISH MTG) as float torch tensor
- ST spot spatial coordinates (spcoor, e.g., simu_st_metadata.csv with x_global/y_global columns)

## Outputs

- netsc2st: row-normalized (softmax, rows sum to 1) SC-to-ST mapping matrix (n_cells x n_spots)
- netst2sc: paired ST-to-SC mapping matrix (n_spots x n_cells, rows sum to 1)
- adata.obsm['spatialDA']: reconstructed single-cell spatial coordinates from all_coord(netsc2st, spcoor)

## How to apply

After training the STEM SOmodel (n_genes set to the ST gene count, sigma equal to half the ST spot adjacent distance, alpha = 0.8 MMD loss weight, lr = 0.002, mmdbatch = 1000, and dp set to the SC-to-ST domain-gap dropout rate computed as 1 - median(n_genes_by_counts SC)/median(n_genes_by_counts ST)) for 400 epochs with train_wholedata, switch the model to evaluation mode with modeleval. Compute the SC and ST embeddings with netE, then build netsc2st = F.softmax(scembedding.mm(stembedding.t()), dim=1).detach().cpu().numpy(), so each row (one cell) sums to 1 over spots. Convert the mapping to single-cell coordinates via all_coord(pd.DataFrame(netsc2st, index=adata.obs_names, columns=stadata.obs_names), spcoor) and store in adata.obsm['spatialDA']; the paired spot-to-cell matrix netst2sc = F.softmax(stembedding.mm(scembedding.t()), dim=1) is built the same way with dimensions transposed.

## Related tools

- **STEM** (Provides SOmodel/netE encoder and all_coord utility used to compute embeddings, the softmax mapping matrices, and single-cell coordinates) — https://github.com/WhirlFirst/STEM
- **PyTorch** (Executes the embedding inner product and F.softmax mapping matrix computation on CUDA)
- **scanpy** (Holds the AnnData objects whose obsm['spatialDA'] stores the reconstructed coordinates and whose normalization precedes mapping)

## Examples

```
netsc2st = F.softmax(scembedding.mm(stembedding.t()),dim=1).detach().cpu().numpy(); adata.obsm['spatialDA'] = all_coord(pd.DataFrame(netsc2st,index=adata.obs_names,columns=stadata.obs_names),spcoor)
```

## Evaluation signals

- Each row of netsc2st sums to 1 (softmax over spots); each row of netst2sc sums to 1 over cells
- Reconstructed cortical-depth distributions of the L1-L6 dissection groups align with the true cortical layer boundaries identified from the ST data
- The excitatory-neuron neighborhood-enrichment heatmap computed from the reconstructed coordinates shows a diagonal enrichment pattern
- Region-specific SC-unique genes (CXCL14, CUX2, RORB, NFIA, APOD) are recovered from the reconstructed spatial landscape
- Downstream comparison: STEM's reconstructed topology preserves the original structure of single cells while competitor methods compress cells into L1/L2/3 or offset them to deeper regions

## Limitations

- Mapping quality depends on correct sigma (set to half the ST spot adjacent distance) and the domain-gap dropout rate dp; mis-set parameters degrade embedding alignment
- Existing deconvolution-style representations cannot link SC and ST at the single-cell level, but conversely the mapping assumes SC and ST share cell types from the same or comparable tissue region
- Application to SC and ST data from different donors in the same disease state is explicitly listed as untested future work

## Evidence

- [readme] The matrix `netst2sc` and `netsc2st` are ST-SC and SC-ST mapping matrices, respectively.: "The matrix `netst2sc` and `netsc2st` are ST-SC and SC-ST mapping matrices, respectively. In the ST-SC mapping matrix, the probability of one spot to other cells summarizes to 1."
- [abstract] It uses deep transfer learning to encode both ST and SC data into a unified spatially aware embedding space: [evidence span withheld — anchor status: not_found]
- [methods] all_coord applied to netsc2st to obtain single-cell spatial coordinates: "adata.obsm['spatialDA'] = all_coord(pd.DataFrame(netsc2st,index=adata.obs_names,columns=stadata.obs_names),spcoor)"
- [results] only the cortical-depth distribution of single cells estimated by STEM fitted the true layer regions: [evidence span withheld — anchor status: not_found]
