---
name: spatial-transcriptomics-automated-scribble-generation
description: Use when expert-provided scribbles are unavailable for a spatial transcriptomics
  sample (Visium or ST) and you need to bootstrap ScribbleDom from mclust's non-spatial
  clustering instead of manual histology annotation.
when_to_use_negative:
- A manual_scribble.csv exists (created via Loupe browser from the .cloupe file) —
  the pipeline will use the expert scribble instead of AutoScribbleDom's automated
  one.
- Your goal is a fully supervised manual domain annotation of tissue types (e.g. Stroma/Lymphoid
  tissue/Melanoma labels from the Thrane et al. melanoma sample) rather than automated
  mclust-derived scribbles.
- The data is not Visium or ST technology, since the pipeline builds six-neighbor
  (Visium) or four-neighbor (ST) lattices specifically.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: Mclust
  role: Gaussian mixture model run on spot principal components to produce the initial
    non-spatial cluster labels that are then consensus-filtered into the automated
    scribble
- name: ScribbleDom
  role: Pipeline containing the AutoScribbleDom variant that applies the lattice-neighbor
    consensus filter and writes mclust_backbone.csv
  repo: https://github.com/1alnoman/ScribbleDom
- name: R
  role: R environment (R 4.3.1) in which the preprocessing scripts and Mclust clustering
    are executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad594
    title: 'ScribbleDom: using scribble-annotated histology images to identify domains
      in spatial transcriptomics data'
  source_content_hash: sha256:5a8c9262b50e5c32cd94d1a849583f4b059695cb1ec9c47d801d85684307a17d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1alnoman/ScribbleDom
  tools:
  - Mclust
  - R
  - ScribbleDom
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# spatial-transcriptomics-automated-scribble-generation

## Summary

AutoScribbleDom generates scribble annotations without expert input by running the Mclust Gaussian mixture model on spot principal components and applying a lattice-neighbor consensus filter, producing the automated scribble file mclust_backbone.csv.

## When to use

Use when expert-provided scribbles are unavailable for a spatial transcriptomics sample (Visium or ST) and you need to bootstrap ScribbleDom from mclust's non-spatial clustering instead of manual histology annotation.

## When NOT to use

- A manual_scribble.csv exists (created via Loupe browser from the .cloupe file) — the pipeline will use the expert scribble instead of AutoScribbleDom's automated one.
- Your goal is a fully supervised manual domain annotation of tissue types (e.g. Stroma/Lymphoid tissue/Melanoma labels from the Thrane et al. melanoma sample) rather than automated mclust-derived scribbles.
- The data is not Visium or ST technology, since the pipeline builds six-neighbor (Visium) or four-neighbor (ST) lattices specifically.

## Inputs

- Raw Space Ranger gene-by-spot count matrix (.h5 for Visium, .rds for ST) under space_ranger_output_directory
- config_mclust.json with fields n_pcs (e.g. 15), n_cluster_for_auto_scribble (e.g. 2), technology (visium/st), dataset, and samples
- Preprocessed spot principal components and spot coordinates under preprocessed_data/{dataset}/{sample}/

## Outputs

- mclust_backbone.csv — automated scribble with spot-level labels for retained spots, placed in preprocessed_data/{dataset}/{sample}/

## How to apply

Load the preprocessed spot principal components, spot coordinates, and config_mclust.json, using the configured n_pcs (e.g. 15) as the clustering feature dimension. Run the Mclust Gaussian mixture model on the principal components with the number of clusters set to n_cluster_for_auto_scribble (e.g. 2) to obtain an initial non-spatial cluster label for every spot. Build the spot lattice adjacency from the coordinates, using the six-neighbor hexagonal grid for Visium data and the four-neighbor square grid for ST data. Apply the AutoScribbleDom consensus filter: keep a spot's scribble label only if it matches the Mclust labels of all of its lattice neighbors, and clear labels on spots whose neighbors disagree. Write the retained spot-level labels to mclust_backbone.csv under preprocessed_data/{dataset}/{sample}/ so the pipeline can run with schema set to 'mclust'. This filtering step enforces spatial coherence on mclust's non-spatial output, which is what allows the scribbles to serve as trustworthy supervision for the semi-supervised network.

## Related tools

- **Mclust** (Gaussian mixture model run on spot principal components to produce the initial non-spatial cluster labels that are then consensus-filtered into the automated scribble)
- **ScribbleDom** (Pipeline containing the AutoScribbleDom variant that applies the lattice-neighbor consensus filter and writes mclust_backbone.csv) — https://github.com/1alnoman/ScribbleDom
- **R** (R environment (R 4.3.1) in which the preprocessing scripts and Mclust clustering are executed)

## Examples

```
Rscript get_genex_data_from_10x_h5.R config_mclust.json
```

## Evaluation signals

- mclust_backbone.csv exists in preprocessed_data/{dataset}/{sample}/ after the preprocessing script runs
- The label column covers only the retained spots — spots whose lattice neighbors had disagreeing Mclust labels have cleared labels
- Sample identifiers in mclust_backbone.csv match the input spot coordinates file
- The technology setting (visium vs st) matches the lattice used: six-neighbor hexagonal for Visium, four-neighbor square for ST
- The number of Mclust clusters equals n_cluster_for_auto_scribble from config_mclust.json (e.g. 2)

## Limitations

- Mclust's initial clustering is non-spatial, so spots in border regions or with mixed expression profiles may receive labels cleared by the neighbor-consensus filter, reducing scribble coverage.
- Only supports technologies with a regular lattice (six-neighbor for Visium, four-neighbor for ST); other spatial array layouts are not handled.
- Running without a manual_scribble.csv yields only the AutoScribbleDom result, which may be less accurate than the expert-scribble variant.

## Evidence

- [other] In the AutoScribbleDom pipeline, mclust's initial non-spatial clustering is filtered so that only spots whose labels match all lattice neighbors (six for Visium, four for ST) are kept as scribbles, and labels on the remaining spots are cleared.: [evidence span withheld — anchor status: not_found]
- [readme] Preprocessing runs Mclust to create the automated scribble file alongside principal components and coordinates.: "Finds the top 2000 highly variable genes and calculates principal components from raw Gene expression data. Also runs Mclust algorithm to create a automated scribble."
- [readme] The preprocessing step outputs mclust_backbone.csv as the automated scribble in the preprocessed data folder.: "This step will produce preprocessed data with principal components of spots, coordinates and mclust_backbone.csv as Automated Scribble in the folder below:"
- [readme] Skipping manual scribble generation restricts results to AutoScribbleDom.: "If you don't generate manual_scribble.csv in this step, you will get the result from AutoScribbleDom only."
- [abstract] ScribbleDom can operate fully unsupervised without expert input and remains competitive with state-of-the-art methods.: "when the expert input is absent, ScribbleDom can still operate, in a fully unsupervised manner like the state-of-the-art methods, and produces results that remain competitive."
