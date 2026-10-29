---
name: gene-set-enrichment-analysis-hallmark-pathways
description: Use when scMDC cluster assignments for a CITE-seq dataset (e.g., BMNC
  GSE128639) exist and you need per-cluster Hallmark pathway enrichment from differential
  gene rankings to interpret cluster biology.
when_to_use_negative:
- You only need cluster labels without biological interpretation — scMDC clustering
  alone suffices and GSEA is unnecessary.
- Input is raw unnormalized counts without scMDC cluster assignments or a derived
  gene rank — GSEA on an arbitrary rank list will not reproduce the paper's analysis.
- Data is not paired multimodal CITE-seq/SMAGE-seq single-cell data with defined groups
  for the minimal-perturbation comparison.
edam_operation: http://edamontology.org/operation_3939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: scMDC
  role: Provides cluster assignments and the LRP minimal-perturbation deep differential
    analysis that produces the directed gene ranks fed to GSEA
  repo: https://github.com/xianglin226/scMDC
- name: fgsea
  role: R package (v1.19.4) that performs gene set enrichment analysis on each directed
    gene rank against Hallmark gene sets
- name: msigdbr
  role: R package (v7.4.1) supplying the Hallmark gene sets used by fgsea
- name: Scanpy
  role: Python package used to preprocess and normalize the mRNA and ADT count data
    prior to clustering and downstream GSEA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-35031-9
    title: Clustering of single-cell multi-omics data with a multimodal deep learning
      method
  source_content_hash: sha256:8faaf77fc3a3f5a35a61c6081785540c5cbc3bdd5b3bcf2d09bb6e1f6d684a1e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xianglin226/scMDC
  tools:
  - Scanpy
  - fgsea
  - h5py
  - msigdbr
  - scMDC
  edam_operation: http://edamontology.org/operation_3939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# gene-set-enrichment-analysis-hallmark-pathways

## Summary

Runs Hallmark gene set enrichment analysis (GSEA) on cluster-level directed gene ranks derived from scMDC's minimal-perturbation deep differential analysis, using fgsea (v1.19.4) with msigdbr (v7.4.1) gene sets in R, to identify enriched pathways (e.g., complement system in CD14 monocytes) per BMNC cluster.

## When to use

Use when scMDC cluster assignments for a CITE-seq dataset (e.g., BMNC GSE128639) exist and you need per-cluster Hallmark pathway enrichment from differential gene rankings to interpret cluster biology.

## When NOT to use

- You only need cluster labels without biological interpretation — scMDC clustering alone suffices and GSEA is unnecessary.
- Input is raw unnormalized counts without scMDC cluster assignments or a derived gene rank — GSEA on an arbitrary rank list will not reproduce the paper's analysis.
- Data is not paired multimodal CITE-seq/SMAGE-seq single-cell data with defined groups for the minimal-perturbation comparison.

## Inputs

- Preprocessed BMNC (GSE128639 / bmcite) h5 file with RNA counts X1 and ADT counts X2 prepared with Scanpy
- Well-trained scMDC model weights (--ae_weights)
- scMDC cluster assignments on the BMNC dataset
- Hallmark gene sets from msigdbr (v7.4.1)

## Outputs

- fgsea_hallmark_gsea_results.csv with per-cluster NES and padj fields
- Fig. 8-style dot plots of enriched Hallmark pathways per cluster
- Pathway heatmap of Hallmark enrichment across clusters
- Per-cluster directed gene ranks from LRP deep differential analysis

## How to apply

First derive a directed gene rank per cluster by running the LRP deep differential analysis (run_LRP.py), which finds the minimal perturbation that alters the group assignment from a source group to a target group, combining gene importances with log-fold-change direction vectors. Export each directed, signed gene rank to R and run fgsea (v1.19.4) with Hallmark gene sets retrieved via msigdbr (v7.4.1), one run per cluster against the rest. Collect the normalized enrichment scores (NES) and adjusted p-values (padj) per cluster and write them to fgsea_hallmark_gsea_results.csv. Finally, render Fig. 8-style dot plots of enriched Hallmark pathways per cluster and a pathway heatmap; interpret top hits such as the complement system pathway having the highest enrichment score in CD14 monocytes.

## Related tools

- **scMDC** (Provides cluster assignments and the LRP minimal-perturbation deep differential analysis that produces the directed gene ranks fed to GSEA) — https://github.com/xianglin226/scMDC
- **fgsea** (R package (v1.19.4) that performs gene set enrichment analysis on each directed gene rank against Hallmark gene sets)
- **msigdbr** (R package (v7.4.1) supplying the Hallmark gene sets used by fgsea)
- **Scanpy** (Python package used to preprocess and normalize the mRNA and ADT count data prior to clustering and downstream GSEA)

## Examples

```
python run_LRP.py --ae_weights scMDC_BMNC_weights.h5 --data BMNC.h5; Rscript fgsea_hallmark.R --ranks lrp_ranks/ --output fgsea_hallmark_gsea_results.csv
```

## Evaluation signals

- fgsea_hallmark_gsea_results.csv exists and contains per-cluster NES and adjusted p-value (padj) columns
- Dot plots and heatmap render enriched Hallmark pathways consistent with the published Fig. 8
- The complement system pathway ranks as the highest-enrichment Hallmark hit in the CD14 monocyte cluster
- Each cluster yields a signed, directed gene rank (importance weighted by log-fold-change direction) before GSEA is run
- fgsea v1.19.4 and msigdbr v7.4.1 versions match those used in the paper for reproducibility

## Limitations

- The number of clusters (K) must be specified by the user; no automatic estimation is provided, and wrong K changes the grouping used for per-cluster GSEA.
- All upstream scMDC experiments were conducted on Nvidia Tesla P100 (16 G) GPU, which may limit reproducibility on CPU-only systems.
- Gene set results depend on msigdbr v7.4.1 Hallmark definitions; other versions of fgsea or msigdbr may not reproduce the published NES/padj values.

## Evidence

- [discussion] this algorithm will find the minimal perturbation that alters the group assignment from a source group (s) to the target group(s) (t): "this algorithm will find the minimal perturbation that alters the group assignment from a source group (s) to the target group(s) (t)"
- [discussion] Based on the directed gene rank, gene set enrichment analysis (GSEA) is performed by the package fgsea (v1.19.4) and msigdbr (v7.4.1) in R: "Based on the directed gene rank, gene set enrichment analysis (GSEA) is performed by the package fgsea (v1.19.4) and msigdbr (v7.4.1) in R"
- [results] In the Hallmark GSEA of the BMNC clusters, the complement system pathway has the highest enrichment score in CD14 monocytes.: [evidence span withheld — anchor status: not_found]
- [discussion] The raw CITE-seq data is preprocessed and normalized by the Python package SCANPY48. mRNA and ADT data are normalized separately but using the same method.: "The raw CITE-seq data is preprocessed and normalized by the Python package SCANPY48. mRNA and ADT data are normalized separately but using the same method."
- [discussion] After pretraining, the users need to specify the number of clusters (K).: "After pretraining, the users need to specify the number of clusters (K)."
