---
name: milo-neighbourhood-pseudobulk-construction
description: Use when converting single-cell VDJ-seq data (e.g. SingleCellExperiment
  with complete TCR information) into a pseudobulked VDJ feature space so continuous
  VDJ gene-usage features can feed diffusion-map trajectory analysis.
when_to_use_negative:
- The dataset has too few relevant cells for miloR neighbourhood construction (e.g.
  Tonglin et al. with only 972 healthy B cells was judged likely insufficient)
- The goal is B cell development trajectory analysis — dandelionR has not yet been
  applied to B cell development data and its absorbing Markov chain cannot represent
  cyclic trajectories such as B cell receptor editing
- Input is already a pseudobulked or feature-space matrix rather than per-cell VDJ-annotated
  single-cell data
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2830
tools:
- name: miloR
  role: Provides neighbourhood graph construction (buildGraph k=30, d=20 on X_scvi)
    and makeNhoods (d=20, prop=0.3) used to define pseudobulks; pseudobulking is achieved
    through the miloR (v2.0.0) package
- name: dandelionR
  role: Implements setupVdjPseudobulk preprocessing and vdjPseudobulk, which tabulates
    abT-prefixed VDJ gene usage per pseudobulk neighbourhood and normalizes the feature
    space
  repo: https://github.com/tuonglab/dandelionR
- name: scRepertoire
  role: Interfaces with the workflow via combineTCR and combineExpression to attach
    TCR contig information to the SingleCellExperiment prior to pseudobulking
  repo: https://github.com/ncborcherding/scRepertoire
- name: SingleCellExperiment
  role: Container format for the preprocessed single-cell VDJ data and the resulting
    milo object with the Feature_space assay
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2025.06.047
    title: '<i>dandelionR</i> : Single-cell immune repertoire trajectory analysis
      in R'
  source_content_hash: sha256:0df309a9062ca959c626a80d77fe46a91f2baebee53e1e1fe5ceeb64a5a229f9
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
  - scRepertoire
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2830
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# milo-neighbourhood-pseudobulk-construction

## Summary

Constructs miloR-derived neighbourhood pseudobulks from a single-cell VDJ object and tabulates V/D/J gene usage per pseudobulk, yielding the VDJ feature space (160 V/D/J genes x 1516 pseudobulks on the Suo et al. data) used for downstream trajectory analysis in dandelionR.

## When to use

Use when converting single-cell VDJ-seq data (e.g. SingleCellExperiment with complete TCR information) into a pseudobulked VDJ feature space so continuous VDJ gene-usage features can feed diffusion-map trajectory analysis.

## When NOT to use

- The dataset has too few relevant cells for miloR neighbourhood construction (e.g. Tonglin et al. with only 972 healthy B cells was judged likely insufficient)
- The goal is B cell development trajectory analysis — dandelionR has not yet been applied to B cell development data and its absorbing Markov chain cannot represent cyclic trajectories such as B cell receptor editing
- Input is already a pseudobulked or feature-space matrix rather than per-cell VDJ-annotated single-cell data

## Inputs

- SingleCellExperiment / milo object with single-cell VDJ annotations (e.g. preprocessed Suo et al. data with complete TCR information)
- scVI latent embedding (e.g. 'X_scvi' reduced dimension) for KNN graph construction

## Outputs

- milo object containing the Feature_space assay of V/D/J gene usage counts per pseudobulk (160 genes x 1516 pseudobulks)
- PCA embedding of the feature space (ncomponents = 20) for trajectory analysis

## How to apply

First preprocess the Suo et al.-style single-cell VDJ data with setupVdjPseudobulk (mode_option = 'abT'), which filters to cells with complete TCR information (17308 of 65102 cells retained) and builds a milo object. Build the neighbourhood graph with miloR's buildGraph (k = 30, d = 20, reduced.dim = 'X_scvi') and makeNhoods (d = 20, prop = 0.3); optionally embed with miloUmap (n_neighbors = 30). Then call dandelionR::vdjPseudobulk with mode_option = 'abT' so VDJ gene columns with the abT prefix are counted per pseudobulk neighbourhood, and normalize the tabulated usage counts via the package's feature-space normalisation (.normalizeFeatureSpace with default min_count and renormalize settings). Store the result as a milo object holding the Feature_space assay, run PCA with ncomponents = 20 for downstream trajectory use, and confirm the feature space is 160 V/D/J genes x 1516 pseudobulks.

## Related tools

- **miloR** (Provides neighbourhood graph construction (buildGraph k=30, d=20 on X_scvi) and makeNhoods (d=20, prop=0.3) used to define pseudobulks; pseudobulking is achieved through the miloR (v2.0.0) package)
- **dandelionR** (Implements setupVdjPseudobulk preprocessing and vdjPseudobulk, which tabulates abT-prefixed VDJ gene usage per pseudobulk neighbourhood and normalizes the feature space) — https://github.com/tuonglab/dandelionR
- **scRepertoire** (Interfaces with the workflow via combineTCR and combineExpression to attach TCR contig information to the SingleCellExperiment prior to pseudobulking) — https://github.com/ncborcherding/scRepertoire
- **SingleCellExperiment** (Container format for the preprocessed single-cell VDJ data and the resulting milo object with the Feature_space assay)

## Examples

```
pb.milo <- vdjPseudobulk(milo_object, mode_option = "abT")
```

## Evaluation signals

- Feature space dimensionality matches 160 V/D/J genes x 1516 pseudobulks on the Suo et al. abT data
- Preprocessed object retains 17308 cells with complete TCR information out of 65102
- Feature_space assay is present in the saved pb.milo RDS object and PCA was run with ncomponents = 20
- VDJ gene columns tabulated per neighbourhood all carry the abT prefix (mode_option = 'abT')
- Reproduced cell counts should be comparable to original Dandelion output (17281 vs 17234 cells, 17208 shared) as a cross-implementation sanity check

## Limitations

- The destiny-based diffusion map built on the feature space only suits bifurcation or linear topologies, limiting trajectory inference versatility
- Fundamental discrepancies in diffusion map and KNN graph construction between R and Python packages hinder full reimplementation of Dandelion
- Only ~90% agreement with the original Dandelion output: 17281 vs 17234 cells with 17208 shared; Lin's CCC for CD4+ probability was 0.850, below the 0.9 threshold
- dandelionR has not been applied to B cell development data, and cyclic trajectories (e.g. B cell receptor editing) cannot be represented by absorbing Markov chains

## Evidence

- [results] The dandelionR::setupVdjPseudobulk function preprocessed the single-cell VDJ data by Suo et al. [24].: "The dandelionR::setupVdjPseudobulk function preprocessed the single-cell VDJ data by Suo et al."
- [methods] Pseudobulking via miloR: "Pseudobulking can be achieved through the miloR (v2.0.0) package [7]."
- [methods] buildGraph parameters: "milo_object <- buildGraph(milo_object, k = 30, d = 20, reduced.dim = "X_scvi")"
- [results] Feature space dimensions: "With 160 V/D/J genes and 1516 pseudobulks, the VDJ feature space captured features"
- [readme] dandelionR purpose: "dandelionR is an R package for performing single-cell immune repertoire trajectory analysis, based on the original python implementation in"
