---
name: cortical-depth-laminar-distribution-analysis
description: Use when single-cell coordinates have been mapped onto layered tissue
  (e.g., human MTG cortex with L1–L6 annotations) and you must verify that reconstructed
  cell positions reproduce laminar organization rather than layer compression or offset.
when_to_use_negative:
- Tissue is not laminar/layered or lacks dissection-group layer annotations — cortical-depth
  distributions against layer boundaries cannot be defined
- No reconstructed single-cell coordinates exist yet; deconvolution methods that cannot
  transfer spatial information to single cells cannot be evaluated this way
- Input is raw count data that has not been normalized (sc.pp.normalize_total) and
  log1p-transformed or passed through the STEM embedding step
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: STEM
  role: Produces the spatially aware embeddings and netsc2st mapping matrix whose
    reconstructed single-cell coordinates are evaluated by cortical-depth distributions
  repo: https://github.com/WhirlFirst/STEM
- name: scanpy
  role: Normalization (sc.pp.normalize_total, sc.pp.log1p) and QC metrics used to
    prepare SC/ST inputs and compute the dropout rate before reconstruction
- name: seaborn
  role: Visualization of cortical-depth distribution plots and neighborhood-enrichment
    heatmaps
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-023-05640-1
    title: STEM enables mapping of single-cell and spatial transcriptomics data with
      transfer learning
  source_content_hash: sha256:73385ba5a439b31bfd5277a7d3bf230d9cf42d3f3ef93eccdc3e425c64d499ac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WhirlFirst/STEM
  tools:
  - STEM
  - scSTEM
  - scanpy
  - seaborn
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# cortical-depth-laminar-distribution-analysis

## Summary

Evaluates single-cell spatial reconstruction in laminar brain tissue by comparing the cortical-depth distributions of L1–L6 dissection groups against true layer boundaries from ST data, plus neighborhood-enrichment and region-specific gene checks.

## When to use

Use when single-cell coordinates have been mapped onto layered tissue (e.g., human MTG cortex with L1–L6 annotations) and you must verify that reconstructed cell positions reproduce laminar organization rather than layer compression or offset.

## When NOT to use

- Tissue is not laminar/layered or lacks dissection-group layer annotations — cortical-depth distributions against layer boundaries cannot be defined
- No reconstructed single-cell coordinates exist yet; deconvolution methods that cannot transfer spatial information to single cells cannot be evaluated this way
- Input is raw count data that has not been normalized (sc.pp.normalize_total) and log1p-transformed or passed through the STEM embedding step

## Inputs

- Reconstructed single-cell spatial coordinates (adata.obsm['spatialDA']) from STEM's netsc2st matrix
- MERFISH ST spatial metadata with cortical layer boundaries for MTG donor H18
- Single-cell dissection-group / cell-type annotations (L1–L6, excitatory neuron subtypes)
- Normalized, log1p-transformed SC (SMART-seq) and ST (MERFISH) count matrices

## Outputs

- Cortical-depth distribution plots of L1–L6 dissection groups overlaid on true layer boundaries
- Neighborhood-enrichment score matrix for excitatory neurons (CSV)
- Region-specific SC-unique gene list (CXCL14, CUX2, RORB, NFIA, APOD)

## How to apply

After reconstructing single-cell coordinates via STEM (netsc2st softmax mapping matrix passed through all_coord into adata.obsm['spatialDA']), assign each cell its dissection-group label (L1–L6) and plot the density of cells along the cortical-depth axis against the true layer boundaries identified from the MERFISH ST data. A correct reconstruction keeps each dissection group's depth distribution within its true laminar region; mis-mappings appear as compression of cells into L1/L2/3 or offset toward deeper layers. Complement the depth check with a neighborhood-enrichment score matrix for excitatory neurons, which should show diagonal enrichment if spatial adjacency was preserved, and verify that region-specific SC-unique genes (CXCL14, CUX2, RORB, NFIA, APOD) are recovered. STEM was the only method evaluated whose cortical-depth distributions fit the true layer regions in the human MTG application.

## Related tools

- **STEM** (Produces the spatially aware embeddings and netsc2st mapping matrix whose reconstructed single-cell coordinates are evaluated by cortical-depth distributions) — https://github.com/WhirlFirst/STEM
- **scanpy** (Normalization (sc.pp.normalize_total, sc.pp.log1p) and QC metrics used to prepare SC/ST inputs and compute the dropout rate before reconstruction)
- **seaborn** (Visualization of cortical-depth distribution plots and neighborhood-enrichment heatmaps)

## Examples

```
adata.obsm['spatialDA'] = all_coord(pd.DataFrame(netsc2st,index=adata.obs_names,columns=stadata.obs_names),spcoor)
```

## Evaluation signals

- L1–L6 dissection-group cortical-depth distributions align with the true cortical layer boundaries identified from the ST data (no compression into L1/L2/3 and no offset to deeper regions)
- Excitatory-neuron neighborhood-enrichment heatmap shows a diagonal enrichment pattern
- CXCL14, CUX2, RORB, NFIA and APOD are recovered as region-specific SC-unique genes
- STEM training loss curve converges over the 400 train_wholedata epochs (lr=0.002, alpha=0.8, mmdbatch=1000)
- Reconstructed coordinates occupy valid ST spot space (all_coord output from a softmax netsc2st matrix where each cell's probabilities over spots sum to 1)

## Limitations

- Only STEM's cortical-depth distributions fitted the true layer regions; competing methods mis-located cells (compressed into L1/L2/3 or offset to deeper regions), so failure of this check may indicate the reconstruction method, not the tissue
- Applying STEM to SC and ST data from different donors in the same disease state remains untested
- The mouse liver demonstration required ST and SC data from the same healthy region with similar cell-type proportions; transfer across mismatched regions may fail

## Evidence

- [results] of all methods evaluated, only the cortical-depth distribution of single cells estimated by STEM fitted the true layer regions: [evidence span withheld — anchor status: not_found]
- [methods] adata.obsm['spatialDA'] = all_coord(pd.DataFrame(netsc2st,index=adata.obs_names,columns=stadata.obs_names),spcoor): "adata.obsm['spatialDA'] = all_coord(pd.DataFrame(netsc2st,index=adata.obs_names,columns=stadata.obs_names),spcoor)"
- [readme] the probability of one cell to other spots summarizes to 1: "In the SC-ST mapping matrix, the probability of one cell to other spots summarizes to 1."
- [readme] The processed data and trained models used for reproducing the results are deposited in [Figshare](https://doi.org/10.6084/m9.figshare.24452812): "The processed data and trained models used for reproducing the results are deposited in [Figshare](https://doi.org/10.6084/m9.figshare.24452812)"
