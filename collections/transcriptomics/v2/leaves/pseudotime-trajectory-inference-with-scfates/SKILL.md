---
name: pseudotime-trajectory-inference-with-scfates
description: Use when you have SiFT-filtered AnnData expression (e.g. local spatial
  signal removed via a coordinate-based kernel) and need to order cells along a trajectory
  and split genes into early/late pseudotime groups.
when_to_use_negative:
- When the input data still contains the local spatial-neighborhood signal (no SiFT
  filtering with a coordinate-based kernel has been applied), since the migratory
  subpopulation and its trajectory are otherwise obscured
- When the analysis goal is marker discovery or clustering itself rather than trajectory
  ordering — use scanpy rank_genes_groups and joint Leiden clustering instead
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SiFT (sift-sc)
  role: Generates the filtered expression matrix (local spatial signal removed using
    spatial coordinates) that serves as input to scFates pseudotime inference
  repo: https://github.com/nitzanlab/sift-sc
- name: scFates
  role: Performs pseudotime inference on the SiFT-filtered data and supports partitioning
    genes into early and late pseudotime groups
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-44757-7
    title: 'SiFT: uncovering hidden biological processes by probabilistic filtering
      of single-cell data'
  source_content_hash: sha256:bb2154380f77588819b4d19e9f10fe63d8c6d769cd66631c99e311a7b236bd72
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nitzanlab/sift-sc
  tools:
  - Python
  - SiFT
  - SiFT (sift-sc)
  - pip
  - scFates
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# pseudotime-trajectory-inference-with-scfates

## Summary

Runs pseudotime/trajectory inference with scFates on SiFT-filtered single-cell expression data (e.g. the APAP liver-regeneration ST slides) and partitions genes into early and late pseudotime groups for downstream figure assembly.

## When to use

Use when you have SiFT-filtered AnnData expression (e.g. local spatial signal removed via a coordinate-based kernel) and need to order cells along a trajectory and split genes into early/late pseudotime groups.

## When NOT to use

- When the input data still contains the local spatial-neighborhood signal (no SiFT filtering with a coordinate-based kernel has been applied), since the migratory subpopulation and its trajectory are otherwise obscured
- When the analysis goal is marker discovery or clustering itself rather than trajectory ordering — use scanpy rank_genes_groups and joint Leiden clustering instead

## Inputs

- SiFT-filtered expression matrix per AnnData object (local spatial signal removed via coordinate-based KnnKernel)
- Concatenated SiFT-filtered slides from GEO accession GSE223561 with PCA embedding and neighbor graph
- Joint Leiden cluster labels (with cluster 1 as the ANXA2+/Anxa2+ migratory hepatocyte subpopulation)

## Outputs

- Pseudotime trajectory over the filtered cells
- Early and late pseudotime gene groupings (pseudotime_groups.csv)
- Fig. 5 pseudotime gene-grouping panel (with fig5_umap_clusters.png and markers.csv)

## How to apply

First generate the trajectory input by running sift.SiFT with a cell-cell similarity kernel built from the spatial coordinates (e.g. sift.kernels.KnnKernel) on each APAP-injury slide, then concatenating the filtered slides and embedding them with PCA plus a neighbor graph for joint Leiden clustering. Run pseudotime inference with scFates on this filtered data — not on the raw counts — because the global migratory hepatocyte signal is only exposed after local spatial neighborhoods are filtered out. Partition the resulting trajectory genes into early and late pseudotime groups. Assemble the outputs (pseudotime gene grouping alongside the joint Leiden UMAP and marker table) into the Fig. 5 panels, saving the pseudotime grouping as pseudotime_groups.csv. Note that SiFT-modified expression is no longer properly log-transformed, so downstream trajectory tooling should tolerate this.

## Related tools

- **SiFT (sift-sc)** (Generates the filtered expression matrix (local spatial signal removed using spatial coordinates) that serves as input to scFates pseudotime inference) — https://github.com/nitzanlab/sift-sc
- **scFates** (Performs pseudotime inference on the SiFT-filtered data and supports partitioning genes into early and late pseudotime groups)

## Examples

```
pip install sift-sc
```

## Evaluation signals

- Pseudotime inference is run on the SiFT-filtered data, not raw counts — verify the input matrix is the filtered one
- Genes are successfully partitioned into two non-empty early and late pseudotime groups saved as pseudotime_groups.csv
- The trajectory covers the joint Leiden clusters including the Anxa2+ cluster 1, whose top marker genes include Anxa2
- Cluster compositions are homogeneous across APAP-injury slides (comparable proportions of cells from each slide), matching the Fig. 5 validation criteria

## Limitations

- SiFT modifies the gene expression so that data is no longer properly log-transformed, a property expected in certain downstream analysis tasks
- SiFT strongly relies on the prior knowledge used to define the cell-cell similarity kernel; an inaccurate or coarse-grained mapping may lead to a loss of signal
- There is no ground truth reference available to evaluate computational filtering results, so evaluation relies on application-specific integration metrics

## Evidence

- [full_text] Run pseudotime inference with scFates on the filtered data and partition genes into early and late pseudotime groups.: [evidence span withheld — anchor status: not_found]
- [intro] Diverse analysis tools on filtered data: "Now, diverse analysis tools can be applied to study the filtered data and explore the underlying biological signals it encodes"
- [discussion] Log-transform caveat: "it modifies the gene expression so that data is no longer properly log-transformed, a property that is expected in certain downstream analysis tasks"
