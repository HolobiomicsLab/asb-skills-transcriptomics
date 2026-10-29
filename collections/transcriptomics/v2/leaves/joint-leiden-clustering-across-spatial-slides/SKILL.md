---
name: joint-leiden-clustering-across-spatial-slides
description: Use when per-slide clustering of spatial transcriptomics data hides a
  global migratory/regenerative subpopulation behind local spatial neighborhoods,
  and you need clusters comparable across multiple APAP-injury slides (GSE223561).
when_to_use_negative:
- The analysis goal is to study local spatial niches or zonation patterns themselves
  — filtering the spatial-coordinate signal with SiFT removes exactly the signal of
  interest in that case.
- 'The slides have not been SiFT-filtered with a spatial-coordinate kernel and you
  are clustering raw per-slide data: cluster identities will then reflect local neighborhoods
  and batch effects and will not be comparable across slides.'
- There is no prior spatial or coordinate information available to define the cell-cell
  similarity kernel — SiFT 'strongly relies on the prior knowledge used to define
  the cell–cell similarity kernel' and an inaccurate kernel can cause signal loss.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SiFT (sift-sc)
  role: Filters the local spatial neighborhood signal from each slide using a KnnKernel
    built from spatial coordinates before slides are concatenated for joint clustering
  repo: https://github.com/nitzanlab/sift-sc
- name: scanpy
  role: Performs the joint embedding (PCA, neighbor graph, UMAP, Leiden clustering)
    and rank_genes_groups marker detection on the concatenated SiFT-filtered slides
- name: scFates
  role: Runs pseudotime inference on the filtered data and partitions genes into early
    and late pseudotime groups as a downstream step after joint clustering
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-44757-7
    title: 'SiFT: uncovering hidden biological processes by probabilistic filtering
      of single-cell data'
  source_content_hash: sha256:fa49bb67f8b694abdaaba677d47f32dd943465975546de74b3d4a663f23bafc6
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
  - scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# joint-leiden-clustering-across-spatial-slides

## Summary

Joint Leiden clustering of SiFT-filtered APAP-injury spatial transcriptomics slides: the per-slide SiFT-filtered AnnData objects (filtered with a KnnKernel over spatial coordinates) are concatenated, embedded via PCA plus a neighbor graph, and Leiden-clustered across samples so that clusters correspond to global subpopulations (e.g., the ANXA2+ migratory hepatocytes) rather than slide-local spatial niches.

## When to use

Use when per-slide clustering of spatial transcriptomics data hides a global migratory/regenerative subpopulation behind local spatial neighborhoods, and you need clusters comparable across multiple APAP-injury slides (GSE223561).

## When NOT to use

- The analysis goal is to study local spatial niches or zonation patterns themselves — filtering the spatial-coordinate signal with SiFT removes exactly the signal of interest in that case.
- The slides have not been SiFT-filtered with a spatial-coordinate kernel and you are clustering raw per-slide data: cluster identities will then reflect local neighborhoods and batch effects and will not be comparable across slides.
- There is no prior spatial or coordinate information available to define the cell-cell similarity kernel — SiFT 'strongly relies on the prior knowledge used to define the cell–cell similarity kernel' and an inaccurate kernel can cause signal loss.

## Inputs

- SiFT-filtered expression matrices (AnnData objects), one per APAP-injury spatial transcriptomics slide from GEO accession GSE223561
- Spatial coordinates per cell used upstream to build the KnnKernel for SiFT filtering
- Raw count matrices and slide/sample labels for the concatenated object

## Outputs

- Joint Leiden cluster assignments across all filtered slides (cluster 1 = ANXA2+ migratory hepatocyte subpopulation)
- Joint UMAP embedding of the concatenated SiFT-filtered data highlighting the Anxa2+ cluster
- Marker gene table per cluster from scanpy rank_genes_groups (markers.csv)
- Per-cluster sample-composition proportions demonstrating homogeneity across slides
- Early/late pseudotime gene grouping from scFates (pseudotime_groups.csv) and figure outputs (fig5_umap_clusters.png)

## How to apply

First filter each spatial slide individually with SiFT using a distance/knn-type cell-cell similarity kernel built from the spatial coordinates (e.g., sift.kernels.KnnKernel), so the local neighborhood signal is removed from the expression matrix. Concatenate the SiFT-filtered slides into one AnnData object, compute a PCA embedding, build a neighbor graph, and run Leiden clustering jointly across all samples rather than per slide — this is what makes cluster identities comparable across slides. Visualize the joint clustering with a UMAP highlighting the clusters, then call scanpy's rank_genes_groups per cluster and verify that cluster 1 corresponds to the ANXA2+ migratory hepatocyte subpopulation with Anxa2 uniquely among its top marker genes, plus a migratory GO-term phenotype. Quantify per-cluster sample compositions and check that every cluster contains comparable proportions of cells from each APAP-injury slide (homogeneity across slides). Optionally continue downstream with scFates pseudotime on the filtered data and partition genes into early and late pseudotime groups. Assemble the Fig. 5 panels and save fig5_umap_clusters.png, markers.csv, and pseudotime_groups.csv; note that SiFT-modified expression is no longer properly log-transformed, which matters for some downstream statistical assumptions.

## Related tools

- **SiFT (sift-sc)** (Filters the local spatial neighborhood signal from each slide using a KnnKernel built from spatial coordinates before slides are concatenated for joint clustering) — https://github.com/nitzanlab/sift-sc
- **scanpy** (Performs the joint embedding (PCA, neighbor graph, UMAP, Leiden clustering) and rank_genes_groups marker detection on the concatenated SiFT-filtered slides)
- **scFates** (Runs pseudotime inference on the filtered data and partitions genes into early and late pseudotime groups as a downstream step after joint clustering)

## Examples

```
import scanpy as sc; sc.pp.pca(ad_sifted); sc.pp.neighbors(ad_sifted); sc.tl.umap(ad_sifted); sc.tl.leiden(ad_sifted, key_added='leiden_joint'); sc.tl.rank_genes_groups(ad_sifted, 'leiden_joint')
```

## Evaluation signals

- Cluster 1 of the joint Leiden clustering has Anxa2 among its top marker genes (via scanpy rank_genes_groups), identifying it as the ANXA2+ migratory hepatocyte subpopulation
- Every cluster contains comparable proportions of cells from each APAP-injury slide (per-cluster sample compositions are homogeneous across slides)
- Cluster 1 shows a migratory GO-term phenotype consistent with the paper's Fig. 5 claims
- The joint clustering is performed on the SiFT-filtered, concatenated slides (PCA + neighbor graph), not on per-slide raw or unfiltered data
- Outputs match the expected artifact set: fig5_umap_clusters.png, markers.csv, pseudotime_groups.csv

## Limitations

- SiFT strongly relies on the prior knowledge used to define the cell–cell similarity kernel; an inaccurate or coarse-grained kernel/mapping may lead to a loss of signal.
- SiFT modifies gene expression so the filtered data is no longer properly log-transformed, a property expected in certain downstream analysis tasks.
- There is no ground truth reference to evaluate the computational filtering results; evaluation relies on application-specific integration metrics and qualitative reproduction criteria.
- Statistical implications of applying commonly used analysis tools to corrected gene expression data remain unexplored.

## Evidence

- [readme] Signal FilTering is a tool for uncovering hidden biological processes in single-cell data.: "Signal FilTering is a tool for uncovering hidden biological processes in single-cell data."
- [discussion] where we used the spatial coordinates to remove local signals and expose global subpopulations of interest involved in the regeneration process: "where we used the spatial coordinates to remove local signals and expose global subpopulations of interest involved in the regeneration process"
- [intro] Now, diverse analysis tools can be applied to study the filtered data and explore the underlying biological signals it encodes: "Now, diverse analysis tools can be applied to study the filtered data and explore the underlying biological signals it encodes"
- [discussion] it modifies the gene expression so that data is no longer properly log-transformed, a property that is expected in certain downstream analysis tasks: "it modifies the gene expression so that data is no longer properly log-transformed, a property that is expected in certain downstream analysis tasks"
