---
name: feature-space-dimensionality-verification
description: Use when a milo pseudobulk object has been processed with vdjPseudobulk
  and you must confirm the resulting Feature_space assay has the expected genes-by-pseudobulks
  dimensions before PCA.
when_to_use_negative:
- Input is already a validated feature table or expression matrix, not a milo object
  requiring vdjPseudobulk tabulation
- The dataset is not the abT (alpha-beta T cell) mode — e.g. B cell data, which the
  article notes dandelionR has not yet been applied to
- Only a small number of cells is available (e.g. the Tonglin et al. 972 healthy B
  cells), which is likely insufficient for miloR neighbourhood construction and would
  make the 1516-pseudobulk expectation invalid
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_2830
tools:
- name: dandelionR
  role: Provides vdjPseudobulk and .normalizeFeatureSpace to build and normalise the
    VDJ feature space whose dimensions are verified
  repo: https://github.com/tuonglab/dandelionR
- name: miloR
  role: Constructs the pseudobulk neighbourhoods (buildGraph k=30, makeNhoods prop=0.3)
    that define the pseudobulk axis of the feature space
- name: SingleCellExperiment
  role: Container format (SingleCellExperiment/milo object) holding the Feature_space
    assay and PCA slot
- name: R
  role: Language in which the validation and RDS export of pb.milo are performed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2025.06.047
    title: '<i>dandelionR</i> : Single-cell immune repertoire trajectory analysis
      in R'
  source_content_hash: sha256:7e02aec49ec479db14aefd55b52a36d617946ee3958f48f2e62e5d7808bffde1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tuonglab/dandelionR
  tools:
  - Dandelion
  - R
  - SingleCellExperiment
  - dandelionR
  - miloR
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_2830
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# feature-space-dimensionality-verification

## Summary

Verifies that the pseudobulked VDJ feature space built by dandelionR::vdjPseudobulk has the expected dimensionality — 160 V/D/J genes × 1516 pseudobulks for the Suo et al. single-cell VDJ data — before downstream PCA/diffusion-map trajectory analysis.

## When to use

Use when a milo pseudobulk object has been processed with vdjPseudobulk and you must confirm the resulting Feature_space assay has the expected genes-by-pseudobulks dimensions before PCA.

## When NOT to use

- Input is already a validated feature table or expression matrix, not a milo object requiring vdjPseudobulk tabulation
- The dataset is not the abT (alpha-beta T cell) mode — e.g. B cell data, which the article notes dandelionR has not yet been applied to
- Only a small number of cells is available (e.g. the Tonglin et al. 972 healthy B cells), which is likely insufficient for miloR neighbourhood construction and would make the 1516-pseudobulk expectation invalid

## Inputs

- milo object (miloR) with pseudobulk neighbourhoods from Suo et al. single-cell VDJ data, retaining 17308 cells with complete TCR information
- Feature_space assay output of dandelionR::vdjPseudobulk with mode_option = "abT"

## Outputs

- Dimensionality check: feature space of 160 V/D/J genes x 1516 pseudobulks
- Validated pb.milo object (RDS) containing the Feature_space assay and a 20-component PCA

## How to apply

After running dandelionR::setupVdjPseudobulk (which retained 17308 of 65102 Suo et al. cells with complete TCR information) and miloR neighbourhood construction, call vdjPseudobulk with mode_option = "abT" so VDJ gene columns carrying the abT prefix are tabulated per pseudobulk neighbourhood. Apply the package's .normalizeFeatureSpace with default min_count and renormalize settings, then check the resulting matrix dimensions against the published reference: 160 V/D/J genes and 1516 pseudobulks. Store the result in the milo object as the Feature_space assay and run runPCA with ncomponents = 20 for trajectory use; a dimension mismatch indicates that filtering or neighbourhood construction changed the cell/neighbourhood composition. Save the validated pb.milo object as an RDS file so downstream diffusion-map steps use a verified feature space.

## Related tools

- **dandelionR** (Provides vdjPseudobulk and .normalizeFeatureSpace to build and normalise the VDJ feature space whose dimensions are verified) — https://github.com/tuonglab/dandelionR
- **miloR** (Constructs the pseudobulk neighbourhoods (buildGraph k=30, makeNhoods prop=0.3) that define the pseudobulk axis of the feature space)
- **SingleCellExperiment** (Container format (SingleCellExperiment/milo object) holding the Feature_space assay and PCA slot)
- **R** (Language in which the validation and RDS export of pb.milo are performed)

## Examples

```
pb.milo <- vdjPseudobulk(milo_object, mode_option = "abT"); stopifnot(nrow(assay(pb.milo, "Feature_space")) == 160, ncol(assay(pb.milo, "Feature_space")) == 1516); saveRDS(pb.milo, "pb.milo.rds")
```

## Evaluation signals

- dim(Feature_space) equals 160 V/D/J genes by 1516 pseudobulks for the Suo et al. abT data
- The milo object retains 17308 cells with complete TCR information after setupVdjPseudobulk filtering (from 65102 input cells)
- runPCA succeeds with ncomponents = 20 on the Feature_space assay without dimension errors
- Feature space values are normalised usage counts consistent with .normalizeFeatureSpace default min_count and renormalize settings
- pb.milo is serialisable to RDS and reproduces the published cell counts (dandelionR output 17,281 cells vs original Dandelion 17,234, 17,208 shared)

## Limitations

- The 160 x 1516 dimensionality reference is specific to the Suo et al. data with mode_option = "abT"; other datasets or modes will differ
- dandelionR output differs slightly from original Dandelion (17,281 vs 17,234 cells), so exact Python parity of the feature space cannot be assumed
- Fundamental discrepancies in diffusion map and KNN graph construction between R and Python packages hinder full reimplementation of the original Dandelion feature space

## Evidence

- [results] The VDJ feature space contained 160 V/D/J genes and 1516 pseudobulks: "With 160 V/D/J genes and 1516 pseudobulks, the VDJ feature space captured features"
- [results] Filtering of Suo et al. cells: "Out of 65102 cells in the data, 17308 cells were retained after filtering due to having complete TCR information"
- [methods] vdjPseudobulk invocation: "pb.milo <- vdjPseudobulk(milo_object,
    mode_option = "abT","
- [results] dandelionR vs Dandelion cell counts: "The dandelionR output includes 17,281 cells, while the original Dandelion output contains 17,234 cells, with 17,208 shared between the two."
- [readme] Purpose of dandelionR: "dandelionR is an R package for performing single-cell immune repertoire trajectory analysis, based on the original python implementation"
