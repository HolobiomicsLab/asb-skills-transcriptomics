---
name: milo-neighbourhood-pseudobulking
description: Use when a SingleCellExperiment with TCR contigs and a scVI/PCA embedding
  (e.g. X_scvi) needs neighbourhood-level pseudobulk VDJ features before trajectory
  inference.
when_to_use_negative:
- The dataset lacks paired single-cell VDJ/TCR sequencing, so no VDJ usage can be
  aggregated into pseudobulks.
- Too few cells for neighbourhood construction — the article notes ~972 B cells was
  'likely insufficient even for miloR neighbourhood construction'.
- The trajectory of interest is cyclic (e.g. B cell receptor editing), which the downstream
  absorbing Markov chain pseudotime cannot represent.
edam_operation: http://edamontology.org/operation_3934
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2830
tools:
- name: dandelionR
  role: Provides vdjPseudobulk and setupVdjPseudobulk to aggregate productive TCR
    V/D/J usage into miloR neighbourhood pseudobulks forming the VDJ feature space
  repo: https://github.com/tuonglab/dandelionR
- name: miloR
  role: Builds the neighbourhood graph (buildGraph k = 30, d = 20) and samples neighbourhoods
    (makeNhoods prop = 0.3) used as the pseudobulking units
- name: scRepertoire
  role: Loads AIRR-format contigs (loadContigs), combines TCR chains (combineTCR removeNA
    = TRUE, filterMulti = TRUE) and merges onto the SCE (combineExpression) prior
    to pseudobulking
  repo: https://github.com/ncborcherding/scRepertoire
- name: Dandelion
  role: Original Python implementation whose VDJ feature space construction dandelionR's
    miloR-based pseudobulking reproduces in R
  repo: https://github.com/zktuong/dandelion
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2025.06.047
    title: '<i>dandelionR</i> : Single-cell immune repertoire trajectory analysis
      in R'
  source_content_hash: sha256:f081a3cb550dc4119caa36dbb0fdcc67cc23839caac037893d09ea1bc419b75c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tuonglab/dandelionR
  tools:
  - Dandelion
  - R
  - dandelionR
  - miloR
  - scRepertoire
  edam_operation: http://edamontology.org/operation_3934
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2830
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# milo-neighbourhood-pseudobulking

## Summary

Build miloR neighbourhoods on a single-cell immune dataset and aggregate VDJ usage into neighbourhood-level pseudobulks, producing the VDJ feature space (e.g. 160 V/D/J genes x 1516 pseudobulks in the dandelionR fetal immune atlas demo) used for downstream diffusion-map trajectory analysis.

## When to use

Use when a SingleCellExperiment with TCR contigs and a scVI/PCA embedding (e.g. X_scvi) needs neighbourhood-level pseudobulk VDJ features before trajectory inference.

## When NOT to use

- The dataset lacks paired single-cell VDJ/TCR sequencing, so no VDJ usage can be aggregated into pseudobulks.
- Too few cells for neighbourhood construction — the article notes ~972 B cells was 'likely insufficient even for miloR neighbourhood construction'.
- The trajectory of interest is cyclic (e.g. B cell receptor editing), which the downstream absorbing Markov chain pseudotime cannot represent.

## Inputs

- SingleCellExperiment merged with TCR contigs via scRepertoire combineExpression (demo_sce + demo_airr, AIRR-format contigs)
- Reduced dimension embedding such as X_scvi for graph construction
- miloR Milo object after setupVdjPseudobulk preprocessing (mode_option = "abT", already.productive = TRUE)

## Outputs

- Milo object with miloR neighbourhood graph and sampled neighbourhoods
- Feature_space assay containing the VDJ pseudobulk matrix (160 V/D/J genes x 1516 pseudobulks in the demo)

## How to apply

Create a miloR object from the merged SingleCellExperiment and build a neighbourhood graph with buildGraph(k = 30, d = 20, reduced.dim = "X_scvi"), then sample neighbourhoods with makeNhoods(reduced_dims = "X_scvi", d = 20, prop = 0.3); optionally refine visualisation with miloUmap(n_neighbors = 30). Run dandelionR::vdjPseudobulk(milo_object, mode_option = "abT") to aggregate productive TCR V/D/J gene usage into pseudobulk profiles per neighbourhood, yielding a Feature_space assay that is the VDJ feature space. In the dandelionR fetal immune atlas demo this feature space contained 160 V/D/J genes and 1516 pseudobulks, and it is subsequently reduced with 20-component PCA and fed to destiny::DiffusionMap (n_pcs = 50, n_eigs = 10). Preprocessing upstream must have filtered non-productive and ambiguous chains (e.g. combineTCR with removeNA = TRUE, filterMulti = TRUE) so only complete TCR information is pseudobulked.

## Related tools

- **dandelionR** (Provides vdjPseudobulk and setupVdjPseudobulk to aggregate productive TCR V/D/J usage into miloR neighbourhood pseudobulks forming the VDJ feature space) — https://github.com/tuonglab/dandelionR
- **miloR** (Builds the neighbourhood graph (buildGraph k = 30, d = 20) and samples neighbourhoods (makeNhoods prop = 0.3) used as the pseudobulking units)
- **scRepertoire** (Loads AIRR-format contigs (loadContigs), combines TCR chains (combineTCR removeNA = TRUE, filterMulti = TRUE) and merges onto the SCE (combineExpression) prior to pseudobulking) — https://github.com/ncborcherding/scRepertoire
- **Dandelion** (Original Python implementation whose VDJ feature space construction dandelionR's miloR-based pseudobulking reproduces in R) — https://github.com/zktuong/dandelion

## Examples

```
milo_object <- buildGraph(milo_object, k = 30, d = 20, reduced.dim = "X_scvi"); milo_object <- makeNhoods(milo_object, reduced_dims = "X_scvi", d = 20, prop = 0.3); pb.milo <- vdjPseudobulk(milo_object, mode_option = "abT")
```

## Evaluation signals

- Feature_space assay dimensions match expectation: 160 V/D/J gene features x 1516 pseudobulks in the fetal immune atlas abT demo.
- miloR graph was built on the intended reduced.dim (X_scvi) with k = 30, d = 20, and neighbourhoods sampled with prop = 0.3 before pseudobulking.
- Only cells with complete, productive TCR information enter the pseudobulk (Suo et al.: 17308 of 65102 cells retained after filtering).
- Downstream 20-component PCA on the Feature_space yields a PC1/PC2 layout where the right-most pseudobulk along PC1 is a valid DP root and CD8/CD4 branch tips sit at PC2 extremes.
- dandelionR cell output should overlap the original Dandelion Python output (17,281 vs 17,234 cells, 17,208 shared).

## Limitations

- Neighbourhood construction requires sufficient cell numbers; the article judged 972 healthy B cells 'likely insufficient even for miloR neighbourhood construction'.
- Fundamental discrepancies in KNN graph construction between R and Python packages hinder exact reimplementation of the original Dandelion pipeline.
- It remains unclear whether brute-force or KD-tree KNN methods more accurately capture the underlying biological structure of neighbourhoods.

## Evidence

- [methods] Pseudobulking via miloR neighbourhoods: "Pseudobulking can be achieved through the miloR (v2.0.0) package [7]."
- [methods] buildGraph parameters: "milo_object <- buildGraph(milo_object, k = 30, d = 20, reduced.dim = "X_scvi")"
- [methods] makeNhoods parameters: "milo_object <- makeNhoods(milo_object,
    reduced_dims = "X_scvi", d = 20,
    prop = 0.3
)"
- [results] Feature space dimensions: "With 160 V/D/J genes and 1516 pseudobulks, the VDJ feature space captured features"
- [results] Upstream filtering rationale: "Out of 65102 cells in the data, 17308 cells were retained after filtering due to having complete TCR information"
- [results] Cross-implementation overlap: "The dandelionR output includes 17,281 cells, while the original Dandelion output contains 17,234 cells, with 17,208 shared between the two."
