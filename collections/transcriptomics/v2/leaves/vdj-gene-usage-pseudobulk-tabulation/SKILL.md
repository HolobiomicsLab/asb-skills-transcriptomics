---
name: vdj-gene-usage-pseudobulk-tabulation
description: Use when single-cell VDJ data has been pseudobulked into miloR neighbourhoods
  and you need a gene-usage feature matrix (VDJ genes x pseudobulks) for downstream
  PCA/diffusion-map trajectory analysis.
when_to_use_negative:
- Input is a raw AIRR/contig list that has not yet been combined into a SingleCellExperiment
  and pseudobulked into miloR neighbourhoods (use combineTCR/combineExpression and
  setupVdjPseudobulk first)
- Data contains no productive TCR information or too few cells for neighbourhood construction
  (e.g. only 972 healthy B cells as in Tonglin et al.)
- The goal is per-cell VDJ usage rather than pseudobulk-level feature space construction
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_2229
tools:
- name: dandelionR
  role: Provides vdjPseudobulk and .normalizeFeatureSpace to tabulate and normalise
    V/D/J gene usage per pseudobulk into the Feature_space assay
  repo: https://github.com/tuonglab/dandelionR
- name: miloR
  role: Supplies the milo object and pseudobulk neighbourhood structure (buildGraph,
    makeNhoods) on which vdjPseudobulk tabulates gene usage
- name: SingleCellExperiment
  role: Container format for the preprocessed Suo et al. data and the resulting Feature_space
    assay with PCA
- name: R
  role: Execution environment for the dandelionR workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2025.06.047
    title: '<i>dandelionR</i> : Single-cell immune repertoire trajectory analysis
      in R'
  source_content_hash: sha256:10478ba3a67163a8ada0951afc8a7a6e3f9158be3caf30da861f567cd0cb0aca
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
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# vdj-gene-usage-pseudobulk-tabulation

## Summary

Tabulates V/D/J gene usage per miloR-derived pseudobulk neighbourhood using dandelionR::vdjPseudobulk, converting single-cell VDJ calls into a continuous VDJ feature space (160 V/D/J genes x 1516 pseudobulks on the Suo et al. abT data) that can be combined with gene expression for trajectory analysis.

## When to use

Use when single-cell VDJ data has been pseudobulked into miloR neighbourhoods and you need a gene-usage feature matrix (VDJ genes x pseudobulks) for downstream PCA/diffusion-map trajectory analysis.

## When NOT to use

- Input is a raw AIRR/contig list that has not yet been combined into a SingleCellExperiment and pseudobulked into miloR neighbourhoods (use combineTCR/combineExpression and setupVdjPseudobulk first)
- Data contains no productive TCR information or too few cells for neighbourhood construction (e.g. only 972 healthy B cells as in Tonglin et al.)
- The goal is per-cell VDJ usage rather than pseudobulk-level feature space construction

## Inputs

- miloR milo object (SingleCellExperiment) with milo neighbourhoods and X_scvi reduced dims, preprocessed by setupVdjPseudobulk
- 17308 cells with complete TCR information from the Suo et al. single-cell VDJ dataset

## Outputs

- milo object containing a Feature_space assay of V/D/J gene usage counts per pseudobulk (160 V/D/J genes x 1516 pseudobulks)
- PCA embedding of the feature space (runPCA, ncomponents = 20)
- Saved pb.milo object as an RDS file

## How to apply

Load the preprocessed Suo et al. milo/SingleCellExperiment object produced by setupVdjPseudobulk, which retains 17308 cells with complete TCR information. Call dandelionR::vdjPseudobulk with mode_option = "abT" so VDJ gene columns carrying the abT prefix are counted per pseudobulk neighbourhood. Normalise the tabulated counts via the package's feature-space normalisation (.normalizeFeatureSpace with default min_count and renormalize settings). Store the result as a milo object with a Feature_space assay and run PCA with ncomponents = 20 for trajectory use. Validate that the resulting feature space matches the expected 160 V/D/J genes x 1516 pseudobulks dimensionality.

## Related tools

- **dandelionR** (Provides vdjPseudobulk and .normalizeFeatureSpace to tabulate and normalise V/D/J gene usage per pseudobulk into the Feature_space assay) — https://github.com/tuonglab/dandelionR
- **miloR** (Supplies the milo object and pseudobulk neighbourhood structure (buildGraph, makeNhoods) on which vdjPseudobulk tabulates gene usage)
- **SingleCellExperiment** (Container format for the preprocessed Suo et al. data and the resulting Feature_space assay with PCA)
- **R** (Execution environment for the dandelionR workflow)

## Examples

```
pb.milo <- vdjPseudobulk(milo_object, mode_option = "abT", ...
feature_space <- t(assay(pb.milo, "Feature_space"))
pb.milo <- runPCA(pb.milo, ncomponents = 20)
```

## Evaluation signals

- Feature space dimensionality matches 160 V/D/J genes x 1516 pseudobulks on the Suo et al. abT data
- VDJ gene columns retained in the tabulation carry the abT prefix consistent with mode_option = "abT"
- Feature_space assay PCA (ncomponents = 20) completes without error on the pb.milo object
- Normalisation uses the package defaults for .normalizeFeatureSpace (min_count and renormalize settings)
- pb.milo object serialises to RDS and reloads with the Feature_space assay intact

## Limitations

- Counts of V/D/J genes are mode-specific: with mode_option = "abT" only abT-prefixed TCR genes are tabulated; BCR (B) modes require re-running
- The downstream destiny-based diffusion map trajectory inference is only suitable for bifurcation or linear topologies
- Neighbourhood construction requires sufficient cells; small datasets (e.g. 972 healthy B cells) are likely insufficient
- Only the 17308 of 65102 Suo et al. cells with complete TCR information are retained; results may differ on noisier TCR calls

## Evidence

- [results] The vdjPseudobulk function counts usage of each V/D/J gene per miloR-derived pseudobulk, yielding a VDJ feature space of 160 V/D/J genes and 1516 pseudobulks: [evidence span withheld — anchor status: not_found]
- [methods] Call dandelionR::vdjPseudobulk on the milo object with mode_option = "abT" so VDJ gene columns with the abT prefix are tabulated per pseudobulk neighbourhood: "pb.milo <- vdjPseudobulk(milo_object,
    mode_option = "abT","
- [results] Out of 65102 cells in the data, 17308 cells were retained after filtering due to having complete TCR information: "Out of 65102 cells in the data, 17308 cells were retained after filtering due to having complete TCR information"
- [methods] Pseudobulking can be achieved through the miloR (v2.0.0) package: "Pseudobulking can be achieved through the miloR (v2.0.0) package [7]."
- [readme] dandelionR enables construction of the VDJ feature space to perform trajectory analysis using diffusion maps and absorbing Markov chains: "It provides all the necessary tools to interface with [`scRepertoire`](https://github.com/ncborcherding/scRepertoire) and a custom implementation of absorbing markov chain for pseudotime inference"
