---
name: clone-barcode-weighted-louvain-clustering
description: Use when single-cell transcriptome data has paired clone barcode assignments
  and you want hybrid clusters weighting clonal origin against transcriptomic similarity,
  e.g. barcoded melanoma/iPSC/HSC datasets.
when_to_use_negative:
- No clone barcode data are available for the cells — pure transcriptome tools like
  Seurat or scanpy remain the appropriate choice.
- Data are full-lineage tracing (CRISPR-Cas9 genome editing or imaging-based family
  trees); the article notes ClonoCluster was not designed for these, though it could
  be adapted to weight by phylogenetic distance.
- You only need 2D visualization rather than clustering — the complementary Warp Factor
  tool warps UMAPs instead of producing cluster assignments.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: ClonoCluster
  role: 'R package performing α-weighted hybrid clustering: combines transcriptome
    NN edge weights with clone-barcode weights and runs Louvain community detection'
  repo: https://github.com/leeprichman/ClonoCluster
- name: Warp Factor
  role: Complementary tool to incorporate clone information into UMAP visualization
    of the hybrid clustering results
  repo: https://github.com/leeprichman/ClonoCluster
- name: data.table
  role: Loads the count matrix and barcode table with fread in the paper's analysis
    scripts
- name: irlba
  role: Provides irlba_wrap for the 25-component PCA used as transcriptome input to
    clustering
- name: magrittr
  role: Pipe operator (%<>%) used in the paper's processing workflow (cm %<>% dt2m)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.xgen.2022.100247
    title: 'ClonoCluster: A method for using clonal origin to inform transcriptome
      clustering'
  source_content_hash: sha256:0674aa5bc992724d09c52f43a2bfb8f36ea76b8952dec5eee726c2564677954e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leeprichman/ClonoCluster
  tools:
  - ClonoCluster
  - R (version >=)
  - Warp Factor
  - data.table
  - irlba
  - magrittr
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# clone-barcode-weighted-louvain-clustering

## Summary

ClonoCluster's method for clustering single-cell profiles by linearly combining transcriptome nearest-neighbor edge weights with clone-barcode weights via the tunable parameter α (0 = purely transcriptome, 1 = purely clone), followed by Louvain community detection. It is used to generate hybrid clusters that reveal clonally driven transcriptomic reorganization while keeping cluster counts comparable to standard transcriptome-only clustering.

## When to use

Use when single-cell transcriptome data has paired clone barcode assignments and you want hybrid clusters weighting clonal origin against transcriptomic similarity, e.g. barcoded melanoma/iPSC/HSC datasets.

## When NOT to use

- No clone barcode data are available for the cells — pure transcriptome tools like Seurat or scanpy remain the appropriate choice.
- Data are full-lineage tracing (CRISPR-Cas9 genome editing or imaging-based family trees); the article notes ClonoCluster was not designed for these, though it could be adapted to weight by phylogenetic distance.
- You only need 2D visualization rather than clustering — the complementary Warp Factor tool warps UMAPs instead of producing cluster assignments.

## Inputs

- normalized or scaled count matrix of barcoded cells (gene-by-cell, loadable from a Seurat object)
- table of unique cell IDs and their assigned clone barcodes

## Outputs

- hybrid cluster assignments per cell at each α level (e.g. *_cluster_assignments.txt)
- cluster count vs α curve used to select high and low α
- Sankey/alluvial plots comparing transcriptome, high-α, and low-α clusterings

## How to apply

Load the normalized/scaled count matrix and barcode table (data.table::fread, then dt2m to get a cell-by-gene matrix), compute a 25-component PCA with irlba_wrap(cm, npc = 25), and run clonocluster over an α grid (alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1), where edges between cells sharing a barcode get a clone weight of 1 (normalized by barcode cell count) and transcriptome and clone weights are linearly combined by α. First fix the resolution by identifying the res value that returns the desired number of transcriptome-only clusters, then plot the number of clusters returned with increasing α to pick the maximum tolerable 'high α' just before the inflection point where cluster counts rise sharply; set 'low α' to half the high α (reported range 0.4–0.7 across samples). Confirm the chosen α with marker fidelity analysis (Find_Markers_ROC, Plot_alluvia_track) or comparison to ground-truth data so hybrid clusters remain biologically meaningful.

## Related tools

- **ClonoCluster** (R package performing α-weighted hybrid clustering: combines transcriptome NN edge weights with clone-barcode weights and runs Louvain community detection) — https://github.com/leeprichman/ClonoCluster
- **Warp Factor** (Complementary tool to incorporate clone information into UMAP visualization of the hybrid clustering results) — https://github.com/leeprichman/ClonoCluster
- **data.table** (Loads the count matrix and barcode table with fread in the paper's analysis scripts)
- **irlba** (Provides irlba_wrap for the 25-component PCA used as transcriptome input to clustering)
- **magrittr** (Pipe operator (%<>%) used in the paper's processing workflow (cm %<>% dt2m))

## Examples

```
clust <- clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1)
```

## Evaluation signals

- Recovered high-α values fall within the reported 0.4–0.7 range across the nine deposited samples
- Cluster counts at α = 0 match transcriptome-only clustering at the chosen resolution
- High α sits just before the inflection point of the cluster-number vs α curve, where counts begin rising sharply
- Marker fidelity of hybrid clusters is equivalent to transcriptome-only clusters (ROC-derived AUCs via Find_Markers_ROC), even though markers differ
- Barcode-to-cluster concordance (cast_confusion, Cohen's kappa vs α) confirms increasing clonal influence with rising α

## Limitations

- The appropriate relative weighting of transcriptome vs clone information may differ between endpoint and transitional systems and requires case-by-case evaluation.
- The study did not reveal a reliable method to identify which experimental systems have intrinsic determinants of cell fate and would thus benefit from clonal information.
- Full-lineage tracing data (CRISPR genome editing, imaging-based family trees) were not addressed, though ClonoCluster could be adapted to weight by phylogenetic distance.
- Further testing of the homogeneity of functional properties within clusters is required to determine which clustering method best matches biological distinctions.
- Only R version >= 3.6 is specified; version numbers for other dependencies are not stated.

## Evidence

- [intro] We linearly combined transcriptome and clone weights using α such that it returned purely transcriptome weights for α = 0 and purely clone weights for α = 1: "We linearly combined transcriptome and clone weights using α such that it returned purely transcriptome weights for α = 0 and purely clone weights for α = 1"
- [methods] clonocluster call with alpha grid: "clust <- clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1)"
- [discussion] resolution and alpha selection procedure: "first identifying the resolution value that returns the desired number of clusters from standard transcriptome-only clustering"
- [discussion] tutorial location: "Example code to determine the optimal α value and an example protocol is provided in the ClonoCluster GitHub repository"
- [readme] README installation and Docker: "ClonoCluster is easily installed with the help of the `devtools` package"
