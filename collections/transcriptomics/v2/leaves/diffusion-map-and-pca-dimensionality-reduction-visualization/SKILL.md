---
name: diffusion-map-and-pca-dimensionality-reduction-visualization
description: 'Decision support for benchmarking a trajectory-inference method: when
  you need to test whether standard dimensionality-reduction embeddings (diffusion
  map, PCA) recapitulate a known lineage-branching event in progenitor scRNA-seq data.'
when_to_use_negative:
- When the goal is quantitative trajectory inference with fate prediction rather than
  visual benchmarking of embeddings — use Mpath itself instead
- When the dataset starts from multiple independent progenitor cell types, which Mpath
  does not yet support and for which this benchmark design is out of scope
- When the input is already a reduced-dimension embedding and the question is only
  about clustering, not about resolving a lineage branch
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0092
tools:
- name: Mpath
  role: Reference trajectory-inference method whose neighborhood-based branching result
    the diffusion map and PCA embeddings are benchmarked against
  repo: JinmiaoChenLab/Mpath
- name: SCUBA
  role: Alternative trajectory-inference baseline run on the same DC progenitor matrix;
    its cluster structure is checked for the cDC1/cDC2 branching event
- name: R
  role: Environment in which the diffusion map, PCA, MST, and Mpath analyses are implemented
    for the comparison
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/ncomms11988
    title: Mpath maps multi-branching single-cell trajectories revealing progenitor
      cell progression during development
  source_content_hash: sha256:adcbf00511feccb5795528cad3891906c616031ff35c79a270383d41b9ce8b02
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JinmiaoChenLab/Mpath
  tools:
  - Mpath
  - R
  - SCUBA
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# diffusion-map-and-pca-dimensionality-reduction-visualization

## Summary

This skill benchmarks whether diffusion map and PCA embeddings of a single-cell dataset (here the mouse DC progenitor matrix) can visually resolve a known branching event, using them as comparison baselines against Mpath's neighborhood-based trajectory inference.

## When to use

Decision support for benchmarking a trajectory-inference method: when you need to test whether standard dimensionality-reduction embeddings (diffusion map, PCA) recapitulate a known lineage-branching event in progenitor scRNA-seq data.

## When NOT to use

- When the goal is quantitative trajectory inference with fate prediction rather than visual benchmarking of embeddings — use Mpath itself instead
- When the dataset starts from multiple independent progenitor cell types, which Mpath does not yet support and for which this benchmark design is out of scope
- When the input is already a reduced-dimension embedding and the question is only about clustering, not about resolving a lineage branch

## Inputs

- QC-passed mouse DC progenitor single-cell expression matrix (from Mpath/SCUBA benchmarking dataset)
- Mature cDC1/cDC2 differentially expressed gene (DEG) signature lists (Supplementary Data 1)
- Cell subset labels for landmark populations (e.g. preDC_c, preDC_d, CDP)

## Outputs

- Diffusion map embedding of cells showing presence/absence of cDC1/cDC2 preDC branching
- PCA embedding of cells scored for visual separation of preDC_c and preDC_d
- Minimum spanning tree with recorded branch placement of preDC_c and preDC_d
- trajectory_method_comparison.csv tabulating per-method branch detection and preDC placement

## How to apply

Load the QC-passed DC progenitor expression matrix and the mature cDC1/cDC2 DEG signature lists (Supplementary Data 1) into R. Embed the cells with a diffusion map and with PCA, exactly as implemented for the Mpath comparisons, and record whether either embedding visually separates the cDC1-committed preDC_c subset from the cDC2-committed preDC_d subset. As a further baseline, construct a conventional minimum spanning tree on Euclidean distances between cells and note the branch placement of preDC_c and preDC_d. Run Mpath on the identical input as the reference method, confirming its neighborhood-based cell state transitions resolve the preDC_c (cDC1) / preDC_d (cDC2) branch. Tabulate per-method outcomes (branch detected yes/no, placement of preDC_c and preDC_d) and write trajectory_method_comparison.csv. The rationale is that Mpath was the only method to faithfully recapitulate the branching of cDC1- and cDC2-committed preDCs, and conventional MST even placed cDC1- and cDC2-committed preDCs on the wrong branch.

## Related tools

- **Mpath** (Reference trajectory-inference method whose neighborhood-based branching result the diffusion map and PCA embeddings are benchmarked against) — JinmiaoChenLab/Mpath
- **SCUBA** (Alternative trajectory-inference baseline run on the same DC progenitor matrix; its cluster structure is checked for the cDC1/cDC2 branching event)
- **R** (Environment in which the diffusion map, PCA, MST, and Mpath analyses are implemented for the comparison)

## Examples

```
Rscript mpath_benchmark.R --expr dc_progenitors.qc.txt --signatures SupplementaryData1.txt --methods diffusionmap,PCA,MST,Mpath --out trajectory_method_comparison.csv
```

## Evaluation signals

- The diffusion map or PCA embedding visually shows the cDC1/cDC2 preDC branching event separating preDC_c from preDC_d — record yes/no per method
- A conventional MST built on Euclidean distances is checked for whether it places preDC_c (cDC1-committed) and preDC_d (cDC2-committed) on the correct branches; the paper reports it placed them on the wrong branch, so this is a known failure marker
- Mpath run on the identical input resolves the preDC_c/preDC_d branch, providing the positive control against which embeddings and MST are compared
- trajectory_method_comparison.csv contains one row per method with branch detected (yes/no) and preDC_c/preDC_d branch placement
- Only Mpath is expected to faithfully recapitulate the branching of cDC1- and cDC2-committed preDCs; any method matching this outcome passes the visual benchmark

## Limitations

- Conventional minimum spanning tree on Euclidean distances misplaces cDC1- and cDC2-committed preDCs on the wrong branch, so embeddings/MST alone cannot be trusted for lineage commitment calls
- Mpath (and hence this benchmark design) requires signature genes of differentiated cell types from end-stage contrasts and does not support processes starting from multiple independent progenitor cell types
- Demonstrated utility is limited to mouse bone marrow-derived DC progenitors and primary human myoblasts datasets

## Evidence

- [other] Benchmarking on the DC progenitor dataset showed that Mpath outperformed SCUBA, monocle, diffusion map, PCA and MST: [evidence span withheld — anchor status: not_found]
- [results] Mpath outperformed baselines: [evidence span withheld — anchor status: not_found]
- [results] MST failure: [evidence span withheld — anchor status: not_found]
- [abstract] Mpath algorithm description: "Mpath, an algorithm that derives multi-branching developmental trajectories using neighborhood-based cell state transitions"
- [abstract] Mpath detect branching event: "The Mpath-generated trajectories detect a branching event at the preDC stage revealing preDC subsets that are exclusively committed to cDC1 or cDC2 lineages."
