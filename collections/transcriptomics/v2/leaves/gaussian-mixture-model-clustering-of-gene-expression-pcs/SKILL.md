---
name: gaussian-mixture-model-clustering-of-gene-expression-pcs
description: Use when expert-provided scribble annotations are unavailable and you
  need an initial non-spatial domain labeling of Visium or ST spots from principal
  components of raw gene expression as input to automated scribble generation.
when_to_use_negative:
- When expert-generated scribbles are available — use the expert schema (manual_scribble.csv
  created in Loupe browser) instead of the mclust backbone.
- When spatially resolved clustering is desired as the final domain calls — mclust
  here is deliberately non-spatial and only an initialization; AutoScribbleDom's lattice-neighbor
  consensus filter and the ScribbleDom network supply spatial continuity.
- When the number of expected domains is unknown — n_cluster_for_auto_scribble must
  be specified (e.g. 2) for the GMM initialization.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: mclust
  role: Gaussian mixture model algorithm run on spot principal components to create
    the automated scribble (mclust_backbone.csv)
  repo: github.com/1alnoman/ScribbleDom
- name: R
  role: Execution environment (R 4.3.1) for the preprocessing scripts get_genex_data_from_10x_h5.R
    / get_genex_data_from_rds_ST_data.R that compute PCs and run Mclust
  repo: github.com/1alnoman/ScribbleDom
- name: ScribbleDom
  role: Pipeline consuming the mclust backbone as automated scribble input when expert
    scribbles are absent (AutoScribbleDom mode)
  repo: https://github.com/1alnoman/ScribbleDom
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad594
    title: 'ScribbleDom: using scribble-annotated histology images to identify domains
      in spatial transcriptomics data'
  source_content_hash: sha256:f40695d318c710ccbe18b6348c711e6a60f7eec902eb65f2760135115f47b04c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - ScribbleDom
  - mclust
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# Gaussian mixture model clustering of gene expression PCs

## Summary

Runs mclust's Gaussian mixture model on spot-level principal components (e.g. n_pcs=15) of spatial transcriptomics data with n_cluster_for_auto_scribble clusters (e.g. 2) to produce an initial non-spatial cluster label per spot, which serves as the backbone for automated scribble generation in AutoScribbleDom.

## When to use

Use when expert-provided scribble annotations are unavailable and you need an initial non-spatial domain labeling of Visium or ST spots from principal components of raw gene expression as input to automated scribble generation.

## When NOT to use

- When expert-generated scribbles are available — use the expert schema (manual_scribble.csv created in Loupe browser) instead of the mclust backbone.
- When spatially resolved clustering is desired as the final domain calls — mclust here is deliberately non-spatial and only an initialization; AutoScribbleDom's lattice-neighbor consensus filter and the ScribbleDom network supply spatial continuity.
- When the number of expected domains is unknown — n_cluster_for_auto_scribble must be specified (e.g. 2) for the GMM initialization.

## Inputs

- config_mclust.json (schema='mclust', n_pcs e.g. 15, n_cluster_for_auto_scribble e.g. 2)
- Raw gene expression count matrix from Space Ranger output (.h5 for Visium with tissue_positions_list.csv, scalefactors_json.json, tissue images; .rds for ST data)
- Preprocessed spot principal components (top 2000 highly variable genes, n_pcs PCs) and spot coordinates

## Outputs

- Initial non-spatial cluster label for every spot from Mclust
- mclust_backbone.csv (automated scribble) in preprocessed_data/{dataset}/{sample}/

## How to apply

Prepare a config_mclust.json with 'schema': 'mclust', a chosen n_pcs (e.g. 15), and n_cluster_for_auto_scribble (e.g. 2), then preprocess the raw count data — top 2000 highly variable genes are selected and principal components are calculated from the raw gene expression data (via Rscript get_genex_data_from_10x_h5.R for Visium or get_genex_data_from_rds_ST_data.R for ST data). Run the Mclust GMM on the spot principal components with the number of clusters fixed to n_cluster_for_auto_scribble to obtain an initial non-spatial cluster label for every spot. The resulting labels are written to mclust_backbone.csv under preprocessed_data/{dataset}/{sample}/ as the automated scribble. Rationale: mclust output is explicitly accepted by ScribbleDom as the 'output of other possibly non-spatial spatial domain detection method (e.g. mclust)' when human scribbles are absent; for AutoScribbleDom, the labels are then filtered against the lattice neighborhood (six neighbors for Visium, four for ST) so only spots whose labels match all neighbors are kept as scribbles.

## Related tools

- **mclust** (Gaussian mixture model algorithm run on spot principal components to create the automated scribble (mclust_backbone.csv)) — github.com/1alnoman/ScribbleDom
- **R** (Execution environment (R 4.3.1) for the preprocessing scripts get_genex_data_from_10x_h5.R / get_genex_data_from_rds_ST_data.R that compute PCs and run Mclust) — github.com/1alnoman/ScribbleDom
- **ScribbleDom** (Pipeline consuming the mclust backbone as automated scribble input when expert scribbles are absent (AutoScribbleDom mode)) — https://github.com/1alnoman/ScribbleDom

## Examples

```
Rscript get_genex_data_from_10x_h5.R config_mclust.json
```

## Evaluation signals

- mclust_backbone.csv exists in preprocessed_data/{dataset}/{sample}/ after running the preprocessing Rscript
- mclust_backbone.csv contains a label column covering the retained spots, and sample identifiers match the input spot coordinates
- Preprocessing outputs include spot principal components and spot coordinates alongside mclust_backbone.csv in the expected folder
- Number of clusters in the GMM equals n_cluster_for_auto_scribble from config_mclust.json (e.g. 2)
- For AutoScribbleDom, retained scribble spots are exactly those whose Mclust labels match all lattice neighbors (six for Visium, four for ST); disagreeing spots' labels are cleared

## Limitations

- Mclust produces a non-spatial clustering; spatial coherence is only imposed afterward by the AutoScribbleDom consensus filter, so border spots are discarded
- The number of clusters must be hand-specified via n_cluster_for_auto_scribble in config_mclust.json
- In the paper's evaluation, ScribbleDom with expert scribbles gave larger gains (1.82%–169.38% ARI on 9 of 12 DLPFC samples); the unsupervised AutoScribbleDom route is only 'competitive' with state-of-the-art unsupervised methods, not necessarily superior
- Recommended environment is Linux with Python 3.10.6, R 4.3.1, conda 4.12.0, plus libcairo2-dev and libxt-dev for R libraries

## Evidence

- [methods] mclust GMM on PCs as automated scribble: "Also runs ```Mclust``` algorithm to create a automated scribble."
- [methods] preprocessing selects HVGs and PCs: "Finds the top 2000 highly variable genes and calculates principal components from raw Gene expression data."
- [intro] mclust as fallback input when non-spatial: "output of other possibly non-spatial spatial domain detection method (e.g. mclust)"
- [abstract] unsupervised operation without expert input: "Notably, when the expert input is absent, ScribbleDom can still operate, in a fully unsupervised manner like the state-of-the-art methods, and produces results that remain competitive."
- [readme] preprocessing step produces mclust_backbone.csv: "This step will produce preprocessed data with principal components of spots, coordinates and mclust_backbone.csv as Automated Scribble"
- [readme] config fields for auto scribble: ""n_cluster_for_auto_scribble" : This field is used for automatic scribble generation. Give number of cluster for mclust initialization of your data."
- [readme] only AutoScribbleDom without manual scribble: "If you don't generate ```manual_scribble.csv``` in this step, you will get the result from AutoScribbleDom only."
