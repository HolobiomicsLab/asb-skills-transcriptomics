---
name: visium-h5-preprocessing-highly-variable-genes
description: Use when you have a Space Ranger output directory with a filtered_feature_bc_matrix.h5
  and spatial/ folder for a Visium sample (e.g. bcdc_ffpe) and need PCA-transformed
  spot data plus an automated scribble before running ScribbleDom.
when_to_use_negative:
- Your input is an ST dataset stored as .rds rather than a Visium 10x h5 — use get_genex_data_from_rds_ST_data.R
  and run_other_st.sh instead
- You already have a manual_scribble.csv and only want the expert (ScribbleDom) result
  — though preprocessing is still required, the Mclust backbone itself is not the
  final automated scribble input
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ScribbleDom
  role: Downstream spatial domain method that consumes the preprocessed PCA data and
    automated Mclust scribble
  repo: https://github.com/1alnoman/ScribbleDom
- name: Mclust
  role: Gaussian mixture clustering that creates the automated scribble (mclust_backbone.csv)
    from the top-2000-HVG PCs
- name: R
  role: Runtime for the get_genex_data_from_10x_h5.R preprocessing script (R 4.3.1
    recommended)
- name: Space Ranger
  role: Produces the filtered_feature_bc_matrix.h5 and .cloupe file that form the
    raw input structure
- name: Loupe browser
  role: Used to create the optional manual_scribble.csv placed in the preprocessed_data
    folder for the expert schema
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad594
    title: 'ScribbleDom: using scribble-annotated histology images to identify domains
      in spatial transcriptomics data'
  source_content_hash: sha256:54f046dea551fb06ad2745e1db45b4b0d1ac6b5e2490b351d3b445f2232906c3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1alnoman/ScribbleDom
  tools:
  - Loupe browser
  - Mclust
  - R
  - ScribbleDom
  - Space Ranger
  - conda
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# visium-h5-preprocessing-highly-variable-genes

## Summary

Preprocess raw Visium Space Ranger h5 count matrices for spatial domain detection by selecting the top 2000 highly variable genes, computing principal components, and generating an automated Mclust scribble (mclust_backbone.csv) as input to ScribbleDom.

## When to use

Use when you have a Space Ranger output directory with a filtered_feature_bc_matrix.h5 and spatial/ folder for a Visium sample (e.g. bcdc_ffpe) and need PCA-transformed spot data plus an automated scribble before running ScribbleDom.

## When NOT to use

- Your input is an ST dataset stored as .rds rather than a Visium 10x h5 — use get_genex_data_from_rds_ST_data.R and run_other_st.sh instead
- You already have a manual_scribble.csv and only want the expert (ScribbleDom) result — though preprocessing is still required, the Mclust backbone itself is not the final automated scribble input

## Inputs

- Space Ranger output directory with {sample}_filtered_feature_bc_matrix.h5 and spatial/ folder (tissue_positions_list.csv, scalefactors_json.json, tissue_hires_image.png, tissue_lowres_image.png)
- config_mclust.json (schema='mclust', n_pcs=15, n_cluster_for_auto_scribble=2, technology='visium')
- Optional: .rds file for ST-technology data instead of 10x h5

## Outputs

- preprocessed_data/{dataset}/{sample}/ folder containing spot principal components (n_pcs=15) and coordinates
- mclust_backbone.csv — the automated scribble from Mclust (n_cluster_for_auto_scribble=2 clusters)

## How to apply

Prepare a config_mclust.json pointing space_ranger_output_directory at the Space Ranger output, with dataset 'cancers', samples ['bcdc_ffpe'], technology 'visium', n_pcs 15, n_cluster_for_auto_scribble 2, and schema 'mclust'. Run Rscript get_genex_data_from_10x_h5.R config_mclust.json (for ST data use get_genex_data_from_rds_ST_data.R instead). The script finds the top 2000 highly variable genes, calculates principal components from the raw gene expression data, and runs Mclust to produce mclust_backbone.csv as the automated scribble. Outputs are written to preprocessed_data/{dataset}/{sample}/ with spot principal components, coordinates, and the Mclust backbone; for the expert schema, a manual_scribble.csv from Loupe browser must also be placed in that folder. The number of Mclust clusters (n_cluster_for_auto_scribble) should match the expected number of tissue domains, e.g. 2 for the tumor/non-tumor breast cancer sample.

## Related tools

- **ScribbleDom** (Downstream spatial domain method that consumes the preprocessed PCA data and automated Mclust scribble) — https://github.com/1alnoman/ScribbleDom
- **Mclust** (Gaussian mixture clustering that creates the automated scribble (mclust_backbone.csv) from the top-2000-HVG PCs)
- **R** (Runtime for the get_genex_data_from_10x_h5.R preprocessing script (R 4.3.1 recommended))
- **Space Ranger** (Produces the filtered_feature_bc_matrix.h5 and .cloupe file that form the raw input structure)
- **Loupe browser** (Used to create the optional manual_scribble.csv placed in the preprocessed_data folder for the expert schema)

## Examples

```
Rscript get_genex_data_from_10x_h5.R config_mclust.json
```

## Evaluation signals

- preprocessed_data/{dataset}/{sample}/ directory exists containing principal components, spot coordinates, and mclust_backbone.csv
- Principal component dimensionality equals n_pcs (15) from the config
- mclust_backbone.csv contains one cluster label per spot (2 clusters for the bcdc_ffpe tumor/non-tumor sample, 2518 spots)
- final_outputs/cancers/bcdc_ffpe prediction files exist after the downstream run reproduces the reported ARIs (~0.82 ScribbleDom, ~0.83 AutoScribbleDom, ~0.84 BayesSpace)
- Run environment matches recommended versions (Python 3.10.6, R 4.3.1, conda 4.12.0) and the scribble_dom conda environment is active

## Limitations

- If manual_scribble.csv is not generated, only AutoScribbleDom results are produced, not expert-schema ScribbleDom results
- Library installation requires apt-get install libcairo2-dev and libxt-dev as prerequisites for the R libraries
- Recommended OS is Linux; determinism settings (torch.backends.cudnn.deterministic = True) apply only on GPU runs

## Evidence

- [methods] Preprocess data with HVG selection and Mclust backbone: "Finds the top 2000 highly variable genes and calculates principal components from raw Gene expression data. Also runs ```Mclust``` algorithm to create a automated scribble."
- [readme] Visium h5 preprocessing command: "For visium data:
```
Rscript get_genex_data_from_10x_h5.R config_mclust.json
```"
- [readme] Input directory structure for Visium Space Ranger output: "└── {samples[i]}_filtered_feature_bc_matrix.h5"
- [readme] Output folder of preprocessing step: "This step will produce preprocessed data with principal components of spots, coordinates and mclust_backbone.csv as Automated Scribble"
