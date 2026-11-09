---
name: mclust-automated-scribble-generation
description: Use when running the ScribbleDom pipeline without expert-drawn manual_scribble.csv
  input, or when reproducing AutoScribbleDom baselines (e.g., melanoma ARI 0.74, breast
  cancer bcdc_ffpe ARI 0.83) on Visium or ST data.
when_to_use_negative:
- A human expert has already produced manual_scribble.csv via Loupe browser and you
  want the ScribbleDom (expert-scribble) result rather than the unsupervised AutoScribbleDom
  result.
- The data are not Visium or ST spatial transcriptomics samples processed through
  the ScribbleDom preprocessing scripts (e.g., already-clustered label tables with
  no raw counts).
edam_operation: http://edamontology.org/operation_3432
tools:
- name: ScribbleDom
  role: downstream pipeline that consumes the Mclust automated scribble to produce
    spatial domain predictions
  repo: https://github.com/1alnoman/ScribbleDom
- name: Mclust
  role: model-based clustering algorithm run on spot principal components to create
    the automated scribble backbone
- name: R
  role: runs the preprocessing scripts get_genex_data_from_10x_h5.R / get_genex_data_from_rds_ST_data.R
    that invoke Mclust
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad594
    title: 'ScribbleDom: using scribble-annotated histology images to identify domains
      in spatial transcriptomics data'
  source_content_hash: sha256:77b464f78659f9af27c63428dac6c48061f4b800da8eccad5579ce2d7030f5f0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1alnoman/ScribbleDom
  tools:
  - Mclust
  - Python
  - R
  - ScribbleDom
  - Space Ranger
  - conda
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# mclust-automated-scribble-generation

## Summary

Generates an automated scribble (mclust_backbone.csv) by running R's Mclust model-based clustering on the principal components of the top 2000 highly variable genes from a spatial transcriptomics sample, enabling ScribbleDom to run in fully unsupervised AutoScribbleDom mode when no expert scribbles exist.

## When to use

Use when running the ScribbleDom pipeline without expert-drawn manual_scribble.csv input, or when reproducing AutoScribbleDom baselines (e.g., melanoma ARI 0.74, breast cancer bcdc_ffpe ARI 0.83) on Visium or ST data.

## When NOT to use

- A human expert has already produced manual_scribble.csv via Loupe browser and you want the ScribbleDom (expert-scribble) result rather than the unsupervised AutoScribbleDom result.
- The data are not Visium or ST spatial transcriptomics samples processed through the ScribbleDom preprocessing scripts (e.g., already-clustered label tables with no raw counts).

## Inputs

- config_mclust.json (schema='mclust', technology='visium' or 'st', n_pcs, n_cluster_for_auto_scribble, space_ranger_output_directory)
- Space Ranger output directory with {sample}_filtered_feature_bc_matrix.h5 and spatial/ files (Visium) or {sample}.rds (ST)
- Raw count matrix of the spatial transcriptomics sample

## Outputs

- mclust_backbone.csv (automated scribble) in preprocessed_data/{dataset}/{sample}/
- Preprocessed data with principal components of spots and spot coordinates in preprocessed_data/{dataset}/{sample}/

## How to apply

Write a config_mclust.json with 'schema' set to 'mclust', plus technology ('visium' or 'st'), n_pcs (e.g., 15), and n_cluster_for_auto_scribble (number of Mclust clusters, e.g., 4 for the Thrane et al. melanoma ST sample or 2 for tumor/non-tumor bcdc_ffpe). For Visium data run 'Rscript get_genex_data_from_10x_h5.R config_mclust.json' (for ST data use get_genex_data_from_rds_ST_data.R); this selects the top 2000 highly variable genes, computes principal components from the raw gene expression data, and runs Mclust on those PCs. The script writes mclust_backbone.csv as the Automated Scribble into preprocessed_data/{dataset}/{sample}/ alongside spot coordinates and PCs. The Mclust backbone then plays the same role as a human annotator's scribbles on the histology image, feeding the ScribbleDom CNN loss function; if manual_scribble.csv is absent, the pipeline yields AutoScribbleDom results only.

## Related tools

- **ScribbleDom** (downstream pipeline that consumes the Mclust automated scribble to produce spatial domain predictions) — https://github.com/1alnoman/ScribbleDom
- **Mclust** (model-based clustering algorithm run on spot principal components to create the automated scribble backbone)
- **R** (runs the preprocessing scripts get_genex_data_from_10x_h5.R / get_genex_data_from_rds_ST_data.R that invoke Mclust)

## Examples

```
Rscript get_genex_data_from_10x_h5.R config_mclust.json
```

## Evaluation signals

- preprocessed_data/{dataset}/{sample}/ contains mclust_backbone.csv after the Rscript preprocessing step completes
- mclust_backbone.csv contains a cluster assignment per spot using n_cluster_for_auto_scribble clusters (e.g., 4 for melanoma, 2 for bcdc_ffpe tumor/non-tumor)
- Preprocessing used the top 2000 highly variable genes and the configured n_pcs principal components
- Running the pipeline with only the mclust config yields AutoScribbleDom output in final_outputs/{dataset}/{sample}/ (no expert-scribble result)
- Downstream ARI is competitive with published AutoScribbleDom values (e.g., ~0.74 melanoma, ~0.83 bcdc_ffpe) against manual annotations

## Limitations

- Without manual_scribble.csv, only AutoScribbleDom results are produced; expert-scribble ScribbleDom results (e.g., melanoma ARI 0.87 vs AutoScribbleDom 0.74) are typically higher, so the automated scribble is the weaker supervision source.
- n_cluster_for_auto_scribble must be chosen by the user per dataset; a wrong cluster count for the Mclust initialization degrades the backbone.
- The R environment needs libcairo2-dev and libxt-dev installed for the R libraries to run.

## Evidence

- [methods] Mclust preprocessing step: "Finds the top 2000 highly variable genes and calculates principal components from raw Gene expression data. Also runs ```Mclust``` algorithm to create a automated scribble."
- [readme] README invocation: "For visium data:
```
Rscript get_genex_data_from_10x_h5.R config_mclust.json"
- [readme] output description: "This step will produce preprocessed data with principal components of spots, coordinates and mclust_backbone.csv as Automated Scribble"
- [readme] fallback behavior: "If you don't generate ```manual_scribble.csv``` in this step, you will get the result from AutoScribbleDom only."
