---
name: cluster-number-inflection-detection
description: Use when tuning the α parameter of hybrid clone/transcriptome clustering
  (e.g., ClonoCluster) and you need a data-driven cutoff for how much clone information
  can be injected before cluster number balloons beyond the transcriptome-only baseline.
when_to_use_negative:
- You have no clonal barcode data and are only choosing a Seurat/scanpy resolution
  — standard resolution tuning applies, not α inflection scanning
- Your goal is to force a specific number of hybrid clusters regardless of transcriptome-cluster
  comparability — the inflection rule intentionally caps α to keep cluster counts
  comparable to transcriptome-only clustering
- You need full-lineage/phylogenetic weighting (CRISPR-Cas9 genome editing or imaging-based
  family trees), which the paper explicitly did not address
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: ClonoCluster
  role: Runs Louvain-based hybrid clustering at each α on the α grid, returning cluster
    assignments whose counts are tabulated to locate the inflection point
  repo: https://github.com/leeprichman/ClonoCluster
- name: R (version >= 3.6)
  role: Analysis environment for running the α sweep, aggregating cluster counts,
    and plotting cluster number versus α
  repo: https://github.com/leeprichman/ClonoCluster
- name: data.table
  role: Loads count matrices and barcode tables (fread) and aggregates per-α cluster
    assignments into count tables
- name: irlba
  role: Provides the 25-component PCA (irlba_wrap) that is the clustering input held
    fixed across the α sweep
- name: magrittr
  role: Piping (cm %<>% dt2m) used in the paper's analysis scripts
  repo: https://github.com/arjunrajlaboratory/ClonoCluster_paper
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.xgen.2022.100247
    title: 'ClonoCluster: A method for using clonal origin to inform transcriptome
      clustering'
  source_content_hash: sha256:b7b653221de85bf267b758f9a494fb607e8b621687f1968f1c4a056b0eac1b22
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leeprichman/ClonoCluster
  tools:
  - ClonoCluster
  - R (version >= 3.6)
  - R (version >=)
  - data.table
  - irlba
  - magrittr
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# cluster-number-inflection-detection

## Summary

A calibration procedure from the ClonoCluster paper that determines the maximum ('high') α weighting of clone-versus-transcriptome edges by plotting the number of clusters returned at each α and selecting the value just before cluster counts inflect upward, keeping hybrid clusters comparable to transcriptome-only cluster counts.

## When to use

Use when tuning the α parameter of hybrid clone/transcriptome clustering (e.g., ClonoCluster) and you need a data-driven cutoff for how much clone information can be injected before cluster number balloons beyond the transcriptome-only baseline.

## When NOT to use

- You have no clonal barcode data and are only choosing a Seurat/scanpy resolution — standard resolution tuning applies, not α inflection scanning
- Your goal is to force a specific number of hybrid clusters regardless of transcriptome-cluster comparability — the inflection rule intentionally caps α to keep cluster counts comparable to transcriptome-only clustering
- You need full-lineage/phylogenetic weighting (CRISPR-Cas9 genome editing or imaging-based family trees), which the paper explicitly did not address

## Inputs

- Normalized/scaled count matrix in gene-by-cell format (e.g., Data_genes/ tables loaded with data.table::fread and converted via dt2m)
- Barcode assignment table of unique cell IDs and their assigned barcodes (Data_barcodes/)
- 25-component PCA of the cell-by-gene matrix (irlba_wrap(cm, npc = 25))
- Chosen transcriptome-only clustering resolution (res = 1 in the paper)

## Outputs

- Table of cluster counts per α value for each deposited sample
- Line plots of cluster number versus α (Figure 1A / Figure S3 style, cluster_size_plots.R)
- Selected 'high α' value per sample (observed range 0.4–0.7) and 'low α' = high α / 2
- Per-α cluster assignments (e.g., *_cluster_assignments.txt) used for downstream alluvia and marker-fidelity checks

## How to apply

First fix a resolution (res = 1 in the paper) and identify the transcriptome-only cluster count at α = 0 as the reference. Then run ClonoCluster over an α grid — alpha = seq(0, 1, by = 0.1) with beta = 0.1 — aggregating, for each sample, a table of cluster counts per α value. Plot cluster number versus α (as in cluster_size_plots.R, Figure 1A and Figure S3) and set 'high α' just before the inflection point where cluster counts rise sharply, i.e., the maximum α whose cluster number remains within the transcriptome-cluster range; set 'low α' to half of high α. The high α value is sample-dependent and ranged from 0.4 to 0.7 across the nine replicates analyzed. Confirm the choice downstream with marker-fidelity analysis (Find_Markers_ROC, Plot_alluvia_track) or comparison to ground truth, since a purely cluster-count criterion does not guarantee biologically meaningful hybrid clusters.

## Related tools

- **ClonoCluster** (Runs Louvain-based hybrid clustering at each α on the α grid, returning cluster assignments whose counts are tabulated to locate the inflection point) — https://github.com/leeprichman/ClonoCluster
- **R (version >= 3.6)** (Analysis environment for running the α sweep, aggregating cluster counts, and plotting cluster number versus α) — https://github.com/leeprichman/ClonoCluster
- **data.table** (Loads count matrices and barcode tables (fread) and aggregates per-α cluster assignments into count tables)
- **irlba** (Provides the 25-component PCA (irlba_wrap) that is the clustering input held fixed across the α sweep)
- **magrittr** (Piping (cm %<>% dt2m) used in the paper's analysis scripts) — https://github.com/arjunrajlaboratory/ClonoCluster_paper

## Examples

```
clust <- clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1)
```

## Evaluation signals

- Recovered high-α values fall within the reported 0.4–0.7 range across samples
- Cluster counts at α = 0 match transcriptome-only clustering at the chosen resolution (res = 1)
- At the selected high α, the number of hybrid clusters is still comparable to standard transcriptome cluster counts (counts rise sharply only at the next α step)
- Low α equals exactly half of the identified high α for each sample
- Marker-fidelity analysis (Find_Markers_ROC / Plot_alluvia_track Sankeys) confirms hybrid clusters at the chosen α have marker AUC comparable to transcriptome-only clusters

## Limitations

- The appropriate relative weighting of transcriptome vs clone information may differ between endpoint and transitional systems and requires case-by-case evaluation
- Cluster-number comparability alone does not establish biological meaningfulness; confirm via marker fidelity or ground-truth comparison
- No reliable method was found to identify which experimental systems have intrinsic determinants of cell fate and would benefit from clonal information, so the calibrated α may not transfer across systems
- The high α value varies between samples (0.4–0.7), so a single fixed α cannot be reused across datasets

## Evidence

- [discussion] Plot the number of clusters returned with increasing α to determine the maximum tolerable α value: "then plotting the number of clusters returned with increasing α"
- [discussion] Identify the resolution that returns the desired transcriptome-only cluster count first: "first identifying the resolution value that returns the desired number of clusters from standard transcriptome-only clustering"
- [discussion] Confirm α using marker fidelity or ground truth: "Analysis of marker fidelity or comparison to ground truth data can be used to confirm that the α value produces hybrid clusters that are biologically meaningful"
- [methods] Concrete clonocluster call over the α grid: "clust <- clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1)"
- [intro] α sweep endpoints definition: "We linearly combined transcriptome and clone weights using α such that it returned purely transcriptome weights for α = 0 and purely clone weights for α = 1"
- [readme] Analysis scripts exist for alpha vs cluster number plots: [evidence span withheld — anchor status: not_found]
