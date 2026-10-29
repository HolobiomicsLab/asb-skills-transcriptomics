---
name: feature-space-normalisation-vdj-counts
description: Use when a miloR pseudobulk object has been processed with vdjPseudobulk
  and the raw V/D/J gene usage counts must be normalised into the Feature_space assay
  prior to runPCA and trajectory inference.
when_to_use_negative:
- Input cells lack complete TCR information or were not filtered through setupVdjPseudobulk
  — the pseudobulk counts and feature space will not be well-defined
- The goal is B cell (BCR) trajectory analysis — dandelionR has not yet been applied
  to B cell development data, and mode_option = 'abT' is T cell specific
- A cyclic trajectory is expected (e.g. B cell receptor editing), which the downstream
  absorbing Markov chain trajectory step cannot represent
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_2229
tools:
- name: dandelionR
  role: Provides vdjPseudobulk and the internal .normalizeFeatureSpace routine that
    tabulate and normalise V/D/J gene usage into the Feature_space assay
  repo: https://github.com/tuonglab/dandelionR
- name: miloR
  role: Supplies the milo object and pseudobulk neighbourhood structure over which
    VDJ gene usage is tabulated
- name: SingleCellExperiment
  role: Container format for the input cells and the Feature_space assay
- name: Dandelion
  role: Original Python implementation whose VDJ feature space strategy dandelionR's
    normalisation replicates
  repo: https://github.com/zktuong/dandelion
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2025.06.047
    title: '<i>dandelionR</i> : Single-cell immune repertoire trajectory analysis
      in R'
  source_content_hash: sha256:349f61155479750e7d3729474ae864074999032709a95227c30d5aae2801ef3c
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
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# feature-space-normalisation-vdj-counts

## Summary

Normalises the tabulated V/D/J gene usage counts produced by dandelionR::vdjPseudobulk into a continuous VDJ feature space suitable for PCA and trajectory analysis, applied to the Suo et al. single-cell VDJ data (160 V/D/J genes x 1516 pseudobulks).

## When to use

Use when a miloR pseudobulk object has been processed with vdjPseudobulk and the raw V/D/J gene usage counts must be normalised into the Feature_space assay prior to runPCA and trajectory inference.

## When NOT to use

- Input cells lack complete TCR information or were not filtered through setupVdjPseudobulk — the pseudobulk counts and feature space will not be well-defined
- The goal is B cell (BCR) trajectory analysis — dandelionR has not yet been applied to B cell development data, and mode_option = 'abT' is T cell specific
- A cyclic trajectory is expected (e.g. B cell receptor editing), which the downstream absorbing Markov chain trajectory step cannot represent

## Inputs

- miloR milo object with pseudobulk neighbourhoods built from the Suo et al. SingleCellExperiment (17308 cells with complete TCR information)
- VDJ gene usage count table tabulated per pseudobulk by dandelionR::vdjPseudobulk with mode_option = 'abT'

## Outputs

- milo object containing a normalised Feature_space assay of 160 V/D/J genes x 1516 pseudobulks
- PCA embedding (runPCA, ncomponents = 20) stored for trajectory analysis
- pb.milo RDS file

## How to apply

Load the milo object produced by setupVdjPseudobulk (17308 cells with complete TCR information retained from the 65102-cell Suo et al. dataset) and call dandelionR::vdjPseudobulk with mode_option = 'abT' so abT-prefixed VDJ gene columns are tabulated per pseudobulk neighbourhood. Pass the result through the package's internal feature-space normalisation (.normalizeFeatureSpace) with default min_count and renormalize settings to convert counts into a continuous feature representation. Store the normalised matrix as the Feature_space assay within the milo object and run PCA with runPCA(ncomponents = 20) for downstream diffusion-map trajectory analysis. Save the resulting pb.milo as an RDS file and confirm the feature space is 160 V/D/J genes x 1516 pseudobulks.

## Related tools

- **dandelionR** (Provides vdjPseudobulk and the internal .normalizeFeatureSpace routine that tabulate and normalise V/D/J gene usage into the Feature_space assay) — https://github.com/tuonglab/dandelionR
- **miloR** (Supplies the milo object and pseudobulk neighbourhood structure over which VDJ gene usage is tabulated)
- **SingleCellExperiment** (Container format for the input cells and the Feature_space assay)
- **Dandelion** (Original Python implementation whose VDJ feature space strategy dandelionR's normalisation replicates) — https://github.com/zktuong/dandelion

## Examples

```
pb.milo <- vdjPseudobulk(milo_object, mode_option = "abT"); pb.milo <- runPCA(pb.milo, ncomponents = 20); saveRDS(pb.milo, "pb.milo.rds")
```

## Evaluation signals

- Feature space dimensionality equals 160 V/D/J genes x 1516 pseudobulks
- Feature_space assay is present in the milo object and PCA (ncomponents = 20) runs without error
- Input milo object retains 17308 cells with complete TCR information (from 65102 total cells in Suo et al.)
- mode_option = 'abT' was used so only abT-prefixed VDJ gene columns are tabulated
- pb.milo object saves/loads as RDS without loss of the Feature_space assay

## Limitations

- The destiny-based diffusion map used downstream limits trajectory inference to bifurcation or linear topologies
- Fundamental discrepancies in diffusion map and KNN graph construction between the R and Python packages hinder exact equivalence with Dandelion (dandelionR output: 17281 cells vs Dandelion: 17234, 17208 shared)
- mode_option = 'abT' restricts the feature space to alpha-beta T cell receptor genes; BCR feature spaces are not yet validated

## Evidence

- [results] The vdjPseudobulk function counts usage of each V/D/J gene per miloR-derived pseudobulk: "With 160 V/D/J genes and 1516 pseudobulks, the VDJ feature space captured features"
- [methods] Feature space construction steps: "pb.milo <- vdjPseudobulk(milo_object,
    mode_option = "abT","
- [results] Filtering rationale: "Out of 65102 cells in the data, 17308 cells were retained after filtering due to having complete TCR information"
- [readme] Package purpose from README: "dandelionR` is an R package for performing single-cell immune repertoire trajectory analysis, based on the original python implementation"
- [methods] Diffusion map step on the feature space: "dm <- DiffusionMap(as.matrix(feature_space), n_pcs = 50, n_eigs = 10)"
