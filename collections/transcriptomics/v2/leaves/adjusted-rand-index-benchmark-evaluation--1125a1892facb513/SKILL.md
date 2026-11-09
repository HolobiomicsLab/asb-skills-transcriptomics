---
name: adjusted-rand-index-benchmark-evaluation
description: Use when a spatial transcriptomics clustering method has produced per-spot
  domain labels and a manual layer/tumor annotation (e.g., DLPFC layers, tumor/non-tumor)
  is available as ground truth for ARI-based benchmarking.
when_to_use_negative:
- No manual ground-truth annotation exists for the sample — ARI cannot be computed,
  so use qualitative domain assessment instead
- Predicted labels come from a task where the number and identity of clusters are
  unconstrained relative to the annotation (ARI assumptions on partition comparability
  do not hold)
- The goal is spatial domain discovery itself rather than benchmarking — use the ScribbleDom
  pipeline directly instead of this evaluation skill
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_0203
tools:
- name: ScribbleDom
  role: Method under benchmark whose predicted domain labels are scored by ARI against
    manual annotations
  repo: https://github.com/1alnoman/ScribbleDom
- name: Mclust
  role: Generates the automated scribble (mclust_backbone.csv) used for the AutoScribbleDom
    variant that is also benchmarked by ARI
- name: Space Ranger
  role: Provides the raw Visium output (filtered_feature_bc_matrix.h5, spatial/ folder,
    .cloupe file) from which spot labels and annotations derive
- name: Loupe browser
  role: Used to create manual_scribble.csv from the .cloupe file; manual annotations
    serve as ARI ground truth
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad594
    title: 'ScribbleDom: using scribble-annotated histology images to identify domains
      in spatial transcriptomics data'
  source_content_hash: sha256:dc70cf42527123c559c6592f333f8229b96e32c8b19d07dd1995aa2b4c2c3049
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1alnoman/ScribbleDom
  tools:
  - Loupe browser
  - Mclust
  - Python
  - R
  - ScribbleDom
  - Space Ranger
  - conda
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# adjusted-rand-index-benchmark-evaluation

## Summary

Evaluates spatial domain detection methods (e.g., ScribbleDom, AutoScribbleDom, BayesSpace, SC-MEB, SpaGCN) by computing the adjusted Rand index (ARI) of predicted spot/domain labels against manual annotations, as used to benchmark ScribbleDom on 12 human DLPFC Visium samples, the melanoma dataset, and the breast cancer (bcdc_ffpe) Visium sample.

## When to use

Use when a spatial transcriptomics clustering method has produced per-spot domain labels and a manual layer/tumor annotation (e.g., DLPFC layers, tumor/non-tumor) is available as ground truth for ARI-based benchmarking.

## When NOT to use

- No manual ground-truth annotation exists for the sample — ARI cannot be computed, so use qualitative domain assessment instead
- Predicted labels come from a task where the number and identity of clusters are unconstrained relative to the annotation (ARI assumptions on partition comparability do not hold)
- The goal is spatial domain discovery itself rather than benchmarking — use the ScribbleDom pipeline directly instead of this evaluation skill

## Inputs

- Predicted spatial domain labels per spot from each method (e.g., final_outputs/cancers/bcdc_ffpe/ for ScribbleDom and AutoScribbleDom runs)
- Manual ground-truth annotations (e.g., manual_annotation.csv tumor/non-tumor labels from Ni et al. 2022; DLPFC manual layer annotations; melanoma manual_annotations.csv mapping 0=Stroma, 1=Lymphoid tissue, 2=Melanoma, 3=Unannotated)
- Per-spot coordinates from the Space Ranger spatial/ folder to align labels

## Outputs

- Adjusted Rand index per method per sample
- Comparison table of ARI values across methods (ScribbleDom, AutoScribbleDom, BayesSpace, SC-MEB, SpaGCN, Giotto, GMM, Louvain) and samples
- Reproduction verdict (pass/fail against reported ARI values with tolerance)

## How to apply

Load each method's predicted domain labels for all spots (e.g., 2518 spots for the bcdc_ffpe Visium sample from final_outputs/{dataset}/{sample}/) alongside the published manual annotation (e.g., Ni et al. 2022 tumor/non-tumor labels or the DLPFC manual layer annotations). Compute the adjusted Rand index between predicted and manual labels per sample; for ScribbleDom runs, generate predictions first via './run_other_visium.sh config_expert.json config_mclust.json' after preprocessing with 'Rscript get_genex_data_from_10x_h5.R config_mclust.json' (top 2000 highly variable genes, n_pcs principal components, Mclust automated scribble). Accept a reproduction when computed ARIs match reported values, e.g., approximately 0.82 (ScribbleDom), 0.83 (AutoScribbleDom), and 0.84 (BayesSpace) on bcdc_ffpe. Aggregate per-sample ARIs into a comparison table and verify aggregate claims such as average ARI improvements of 51.90% over BayesSpace, 47.40% over SC-MEB, and 49.84% over SpaGCN across the 12 DLPFC samples.

## Related tools

- **ScribbleDom** (Method under benchmark whose predicted domain labels are scored by ARI against manual annotations) — https://github.com/1alnoman/ScribbleDom
- **Mclust** (Generates the automated scribble (mclust_backbone.csv) used for the AutoScribbleDom variant that is also benchmarked by ARI)
- **Space Ranger** (Provides the raw Visium output (filtered_feature_bc_matrix.h5, spatial/ folder, .cloupe file) from which spot labels and annotations derive)
- **Loupe browser** (Used to create manual_scribble.csv from the .cloupe file; manual annotations serve as ARI ground truth)

## Examples

```
chmod +x run_other_visium.sh && ./run_other_visium.sh config_expert.json config_mclust.json  # then compute ARI of final_outputs labels vs manual_annotation.csv
```

## Evaluation signals

- Computed ARIs on bcdc_ffpe reproduce the reported values of approximately 0.82 (ScribbleDom), 0.83 (AutoScribbleDom), and 0.84 (BayesSpace) within tolerance
- Domain prediction files exist in final_outputs/{dataset}/{sample}/ for every method and sample benchmarked
- Aggregate DLPFC table reproduces the reported improvements: 1.82% to 169.38% on 9 of 12 samples, with averages of 51.90% (BayesSpace), 47.40% (SC-MEB), and 49.84% (SpaGCN)
- Melanoma benchmark reproduces the reported 15.54% ARI improvement
- Predicted label vector length matches the spot count (e.g., 2518 spots for bcdc_ffpe) and indices align with the manual annotation

## Limitations

- ARI comparison requires the same spot set across methods and ground truth; unannotated border spots (None in the melanoma annotation) must be handled consistently
- Reproduction tolerance is anchored to the BayesSpace benchmark; small numeric drift in preprocessing (n_pcs, HVG selection) can shift ARIs
- ScribbleDom results depend on scribble quality — without manual_scribble.csv only AutoScribbleDom results are obtained, changing the expected ARI

## Evidence

- [results] On the human breast cancer Visium sample, ScribbleDom achieved an ARI of 0.82, AutoScribbleDom achieved 0.83, and BayesSpace achieved 0.84.: [evidence span withheld — anchor status: not_found]
- [abstract] between 1.82% to 169.38% improvements in adjusted Rand index for 9 of the 12 human dorsolateral prefrontal cortex samples: "between 1.82% to 169.38% improvements in adjusted Rand index for 9 of the 12 human dorsolateral prefrontal cortex samples"
- [methods] Compute the adjusted Rand index of every method's predicted domains against the manual annotations for each sample: [evidence span withheld — anchor status: not_found]
- [results] Load the predicted domain labels for all 2518 spots, read the manual tumor/non-tumor annotation (Ni et al. 2022): [evidence span withheld — anchor status: not_found]
- [readme] This step will produce preprocessed data with principal components of spots, coordinates and mclust_backbone.csv as Automated Scribble: "This step will produce preprocessed data with principal components of spots, coordinates and mclust_backbone.csv as Automated Scribble"
