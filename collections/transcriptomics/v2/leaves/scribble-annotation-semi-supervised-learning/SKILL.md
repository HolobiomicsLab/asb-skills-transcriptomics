---
name: scribble-annotation-semi-supervised-learning
description: Use when you have Visium or ST spatial transcriptomics data with a Space
  Ranger output (filtered_feature_bc_matrix.h5 + spatial/ files) and either expert
  scribbles on histology images or a non-spatial clustering backbone (e.
when_to_use_negative:
- When no histology image or .cloupe file is available and no non-spatial clustering
  backbone (e.g. Mclust) can be generated — the scribble input is central to the method
- When the data are neither Visium nor ST platform ('technology' supports only visium/st
  in this pipeline)
- When a fully supervised or purely expression-based spatial clustering baseline (e.g.
  BayesSpace, SC-MEB, SpaGCN, Giotto) is the intended comparison rather than a scribble-informed
  semi-supervised method
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_3794
tools:
- name: ScribbleDom
  role: Semi-supervised CNN that consumes scribble annotations and gene expression
    to identify spatial domains
  repo: https://github.com/1alnoman/ScribbleDom
- name: Loupe browser
  role: Used to create manual_scribble.csv from the Space Ranger .cloupe file
  repo: https://support.10xgenomics.com/single-cell-gene-expression/software/visualization/latest/what-is-loupe-cell-browser
- name: Mclust
  role: Generates the automated scribble (mclust_backbone.csv) during preprocessing
    for the unsupervised fallback mode
- name: Space Ranger
  role: Produces the .cloupe file and h5/spatial outputs that form the pipeline input
  repo: https://support.10xgenomics.com/spatial-gene-expression/software/pipelines/latest/output/aggr-overview
- name: ST Spot Detector
  role: Maps ST spots to high-resolution image pixels for manual annotation (melanoma
    sample)
  repo: https://github.com/SpatialTranscriptomicsResearch/st_spot_detector
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad594
    title: 'ScribbleDom: using scribble-annotated histology images to identify domains
      in spatial transcriptomics data'
  source_content_hash: sha256:352ebaeaa6646ece6043d194c2b6adbe87f1ca8217b313b762f6e4782e6852f1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1alnoman/ScribbleDom
  tools:
  - Loupe browser
  - Mclust
  - Python
  - R
  - ST Spot Detector
  - ScribbleDom
  - Space Ranger
  - conda
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_3794
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# scribble-annotation-semi-supervised-learning

## Summary

Semi-supervised spatial domain identification (ScribbleDom) that trains a convolutional network on spatial transcriptomics data using expert scribble annotations drawn on histology images, with a loss combining gene expression profile similarity and adherence to scribble input, plus an Inception-style spatial-continuity component. Falls back to a fully unsupervised mode (AutoScribbleDom) using an Mclust-generated automated scribble when expert input is absent.

## When to use

Use when you have Visium or ST spatial transcriptomics data with a Space Ranger output (filtered_feature_bc_matrix.h5 + spatial/ files) and either expert scribbles on histology images or a non-spatial clustering backbone (e.g. Mclust), and you need tissue domain labels evaluated by adjusted Rand index.

## When NOT to use

- When no histology image or .cloupe file is available and no non-spatial clustering backbone (e.g. Mclust) can be generated — the scribble input is central to the method
- When the data are neither Visium nor ST platform ('technology' supports only visium/st in this pipeline)
- When a fully supervised or purely expression-based spatial clustering baseline (e.g. BayesSpace, SC-MEB, SpaGCN, Giotto) is the intended comparison rather than a scribble-informed semi-supervised method

## Inputs

- Space Ranger output directory containing {sample}_filtered_feature_bc_matrix.h5 and spatial/ files (tissue_positions_list.csv, scalefactors_json.json, tissue_hires_image.png, tissue_lowres_image.png) for Visium data
- Sample .rds file for ST-platform data
- config_expert.json and config_mclust.json (schema 'expert' vs 'mclust', technology visium/st, n_pcs, n_cluster_for_auto_scribble)
- manual_scribble.csv created via Loupe browser from the Space Ranger .cloupe file (for semi-supervised expert mode)

## Outputs

- mclust_backbone.csv (automated scribble) and preprocessed data (spot principal components, coordinates) in preprocessed_data/{dataset}/{sample}/
- Spatial domain predictions per spot in final_outputs/{dataset}/{sample}/
- Manual annotation maps (e.g. melanoma manual_annotations.csv mapping spots to Stroma/Lymphoid tissue/Melanoma labels)

## How to apply

Prepare two JSON configs per sample — config_expert.json (schema 'expert') and config_mclust.json (schema 'mclust') — setting technology to 'visium' or 'st', the number of principal components (n_pcs), n_cluster_for_auto_scribble, and space_ranger_output_directory following the required layout ({dir}/{dataset}/{sample}/spatial/ plus {sample}_filtered_feature_bc_matrix.h5). Preprocess with 'Rscript get_genex_data_from_10x_h5.R config_mclust.json', which selects the top 2000 highly variable genes, computes principal components, and runs Mclust to emit mclust_backbone.csv (the automated scribble) into preprocessed_data/{dataset}/{sample}/. For expert mode, create manual_scribble.csv in Loupe browser from the Space Ranger .cloupe file and place it in the same preprocessed_data folder; omitting it yields only AutoScribbleDom results. Run the pipeline via './run_other_visium.sh config_expert.json config_mclust.json' (or run_other_st.sh for ST data) to write domain predictions to final_outputs/{dataset}/{sample}/. Rationale: the loss integrates similarity in gene expression profiles with the annotator's scribbles, and spatial continuity is captured by convolution filters of varying sizes in Inception blocks; hyperparameters such as max_iter=300, nConv, alpha_options, beta_options, and seed_options are tuned through the config.

## Related tools

- **ScribbleDom** (Semi-supervised CNN that consumes scribble annotations and gene expression to identify spatial domains) — https://github.com/1alnoman/ScribbleDom
- **Loupe browser** (Used to create manual_scribble.csv from the Space Ranger .cloupe file) — https://support.10xgenomics.com/single-cell-gene-expression/software/visualization/latest/what-is-loupe-cell-browser
- **Mclust** (Generates the automated scribble (mclust_backbone.csv) during preprocessing for the unsupervised fallback mode)
- **Space Ranger** (Produces the .cloupe file and h5/spatial outputs that form the pipeline input) — https://support.10xgenomics.com/spatial-gene-expression/software/pipelines/latest/output/aggr-overview
- **ST Spot Detector** (Maps ST spots to high-resolution image pixels for manual annotation (melanoma sample)) — https://github.com/SpatialTranscriptomicsResearch/st_spot_detector

## Examples

```
Rscript get_genex_data_from_10x_h5.R config_mclust.json; ./run_other_visium.sh config_expert.json config_mclust.json
```

## Evaluation signals

- Adjusted Rand index against manual layer annotations: ScribbleDom should improve ARI by 1.82% to 169.38% over state-of-the-art methods on 9 of 12 human DLPFC Visium samples, with average gains of 51.90% (BayesSpace), 47.40% (SC-MEB), and 49.84% (SpaGCN)
- On the melanoma cancer dataset, expect a 15.54% ARI improvement over comparable methods
- Preprocessing outputs must exist: preprocessed_data/{dataset}/{sample}/ should contain principal components of spots, coordinates, and mclust_backbone.csv
- Final predictions must appear at final_outputs/{dataset}/{sample}/ after running the shell script
- Reproducibility check: torch.backends.cudnn.deterministic = True and benchmark = False are set (needed only on GPU runs); conda env 'scribble_dom' built from environment.yml with Python 3.10.6 and R 4.3.1

## Limitations

- libcairo2-dev and libxt-dev must be installed as prerequisites for the R libraries; recommended environment is Linux with Python 3.10.6, R 4.3.1, conda 4.12.0
- If manual_scribble.csv is not generated, only the fully unsupervised AutoScribbleDom result is produced, not the expert semi-supervised mode
- Manual scribble generation requires a video-tutorial-driven Loupe browser workflow on the .cloupe file; expert annotation effort is a prerequisite for the best-performing mode
- The method remains competitive but not guaranteed superior without expert input — in the fully unsupervised mode results are only 'competitive' with state-of-the-art methods

## Evidence

- [abstract] Loss function integrates expression similarity and scribbles: "ScribbleDom incorporates a loss function that integrates two crucial components: similarity in gene expression profiles and adherence to the valuable input of a human annotator through scribbles on"
- [intro] Scribble annotation drives semi-supervised domain detection: "A method to find spatial domain from Spatial Transcriptomics data using scribble annotated histology image, or using output of other possibly non-spatial spatial domain detection method (e.g. mclust)."
- [readme] Preprocessing produces automated scribble via Mclust: "Finds the top 2000 highly variable genes and calculates principal components from raw Gene expression data. Also runs Mclust algorithm to create a automated scribble."
- [readme] Manual scribble created with Loupe browser: "Create a manual scribble (``manual_scribble.csv``) using [Loupe browser](https://support.10xgenomics.com/single-cell-gene-expression/software/visualization/latest/what-is-loupe-cell-browser) and a"
- [readme] Pipeline invocation for Visium data: "chmod +x run_other_visium.sh
./run_other_visium.sh config_expert.json config_mclust.json"
