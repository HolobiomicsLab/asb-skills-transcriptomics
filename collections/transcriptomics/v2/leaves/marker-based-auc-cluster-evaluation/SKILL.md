---
name: marker-based-auc-cluster-evaluation
description: Use when a clustering (e.g., Louvain on a Sphetcher sketch) should be
  validated against a known rare cell type for which canonical marker genes are available,
  to quantify recovery of that population.
when_to_use_negative:
- No prior marker-gene knowledge exists for the population of interest, so no marker-based
  labels can be constructed for AUROC computation
- Clustering was performed on the full dataset rather than a sketch and the goal is
  de novo cell-type discovery rather than validation of recovery of a known population
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: Louvain
  role: Community-detection clustering of the sketched cells, producing the clusters
    that are then evaluated via marker-gene AUROC
- name: Sphetcher
  role: Produces the spherical sketch (sketch_indicator_output.csv with 20,000 ones)
    whose clustered cells are evaluated for marker-gene enrichment
  repo: https://github.com/canzarlab/Sphetcher
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2020.101126
    title: 'Sphetcher: Spherical Thresholding Improves Sketching of Single-Cell Transcriptomic
      Heterogeneity'
  source_content_hash: sha256:ca166aae9e3ea6b37400a8f8ca6739af9aa9ddf15f8072f5b88292ca6839ac2b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/canzarlab/Sphetcher
  tools:
  - Louvain
  - Sphetcher
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Marker-based AUROC cluster evaluation

## Summary

Evaluate whether a clustering of sketched single-cell data recovered a target rare cell population by quantifying enrichment of known marker genes within the cluster and computing an AUROC of the cluster against marker-gene labels (e.g., inflammatory macrophages scored on CD74, HLA-DRA, B2M, JUNB).

## When to use

Use when a clustering (e.g., Louvain on a Sphetcher sketch) should be validated against a known rare cell type for which canonical marker genes are available, to quantify recovery of that population.

## When NOT to use

- No prior marker-gene knowledge exists for the population of interest, so no marker-based labels can be constructed for AUROC computation
- Clustering was performed on the full dataset rather than a sketch and the goal is de novo cell-type discovery rather than validation of recovery of a known population

## Inputs

- Cluster assignments for the sketched cells (e.g., Louvain communities over the 20,000-cell Sphetcher-H spherical sketch)
- Expression matrix of the sketched cells (rows = cells, columns = genes) for scoring marker genes
- Marker-gene list for the target population (e.g., CD74, HLA-DRA, B2M, JUNB for inflammatory macrophages)

## Outputs

- Identified cluster corresponding to the rare inflammatory macrophage population
- AUROC of the macrophage cluster against marker-gene labels, written to macrophage_marker_auroc.csv

## How to apply

After subsetting the original expression matrix to the sketched cells (the 20,000 cells marked with 1 in sketch_indicator_output.csv), cluster them with a standard scRNA-seq method such as Louvain community detection, as noted in the Sphetcher paper. Identify the cluster that plausibly corresponds to the target rare population — for umbilical cord blood, the inflammatory macrophages. For each cell, derive marker-gene labels (positive if the cell expresses the markers CD74, HLA-DRA, B2M, and JUNB) and compute the AUROC of the macrophage cluster against these marker-gene labels to quantify marker enrichment. Write the resulting AUROC to macrophage_marker_auroc.csv and compare against the paper's reported value of AUROC > 0.88 as the success threshold.

## Related tools

- **Louvain** (Community-detection clustering of the sketched cells, producing the clusters that are then evaluated via marker-gene AUROC)
- **Sphetcher** (Produces the spherical sketch (sketch_indicator_output.csv with 20,000 ones) whose clustered cells are evaluated for marker-gene enrichment) — https://github.com/canzarlab/Sphetcher

## Examples

```
sphetcher cord_blood_pcs.csv 20000 sketch_indicator_output.csv  # then cluster the 20,000 indicated cells with Louvain and compute macrophage_marker_auroc.csv
```

## Evaluation signals

- The sketch indicator vector contains exactly 20,000 ones before clustering and evaluation
- A Louvain cluster corresponding to inflammatory macrophages is identifiable among the sketched cells
- Marker genes CD74, HLA-DRA, B2M, and JUNB are enriched within the macrophage cluster
- The computed AUROC of the macrophage cluster against marker-gene labels exceeds 0.88, matching the paper's reported value
- Result is written to macrophage_marker_auroc.csv in the expected single-value output format

## Limitations

- The paper did not systematically evaluate the interplay between distance-metric-based sketching and internal preprocessing routines of downstream scRNA-seq algorithms
- Sphetcher's underlying model might be less compatible with assumptions made by algorithms other than Louvain and Monocle 2, so AUROC recovery may not transfer to other clustering methods
- No software version numbers are stated for Sphetcher or the downstream tools, complicating exact reproduction of the reported AUROC

## Evidence

- [full_text] Clustering the 20,000-cell Sphetcher-H spherical sketch of the umbilical cord blood data with Louvain recovered a cluster of inflammatory macrophages with marker genes CD74, HLA-DRA, B2M, and JUNB (AUROC > 0.88).: [evidence span withheld — anchor status: not_found]
- [discussion] Louvain community detection and Monocle 2 are widely used for scRNA-seq clustering: "Louvain community detection and Monocle 2 are widely used for scRNA-seq clustering"
- [methods] sphetcher expression_matrix.csv sketch_size sketch_indicator_output.csv: "sphetcher expression_matrix.csv sketch_size sketch_indicator_output.csv"
- [readme] an indicator vector of `n` samples where 1 indicates the sample is in the sketch, 0 otherwise (there are `sketch_size` 1's in the vector).: "an indicator vector of `n` samples where 1 indicates the sample is in the sketch, 0 otherwise (there are `sketch_size` 1's in the vector)."
- [intro] which in turn benefits downstream analyses such as clustering and allows to detect a rare population of inflammatory macrophages: "which in turn benefits downstream analyses such as clustering and allows to detect a rare population of inflammatory macrophages"
