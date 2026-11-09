---
name: cross-method-deg-list-comparison
description: Use when an annotated multi-timepoint scRNA-seq dataset (binary Group
  factor plus a Time variable) needs DE genes identified by both a temporal method
  (CASi FindTemporalDEGs) and a conventional tool (Seurat) so their result lists can
  be intersected and compared.
when_to_use_negative:
- The goal is cross-timepoint cell annotation or novel cell detection — use CASi's
  CrossTimeAnnotation/DetectNovelCells functions instead, not DE comparison.
- The dataset has no multi-timepoint structure (single time point, cross-sectional
  only) — CASi's GLM models time and time-by-group interaction effects, which are
  undefined here.
- The expected temporal effect is nonlinear — CASi's GLM framework only detects linear
  changes of expression by time or time-by-condition interaction, so a single-method
  comparison built on it would miss nonlinear dynamics.
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: CASi
  role: FindTemporalDEGs fits a GLM with iterative feature selection for time and
    time-by-group interaction effects to identify tDEGs
  repo: github.com/yizhuo-wang/CASi
- name: Seurat
  role: Runs conventional differential expression on the same annotated MCL data as
    the comparator method
- name: ggplot2
  role: Plots mean expression trajectories of IFITM2 and IFITM3 over time split by
    responders vs non-responders
- name: dplyr
  role: Compares and intersects the CASi and Seurat gene lists
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-024-58566-x
    title: 'CASi: A framework for cross-timepoint analysis of single-cell RNA sequencing
      data'
  source_content_hash: sha256:81e35855772047ebdea7fec5c756c71824fef7001f70e4d0c36642cfa50cae81
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CASi
  - Seurat
  - dplyr
  - ggplot2
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# cross-method-deg-list-comparison

## Summary

Benchmark CASi's temporal differential expression (tDEG) calls against a standard single-timepoint DE tool (Seurat) on the same annotated multi-timepoint scRNA-seq dataset, quantifying per-method gene counts at a shared significance cutoff and their overlap. In the MCL case study this reproduces 268 CASi tDEGs vs 75 Seurat DEGs (p = 0.05) with 21 shared genes.

## When to use

Use when an annotated multi-timepoint scRNA-seq dataset (binary Group factor plus a Time variable) needs DE genes identified by both a temporal method (CASi FindTemporalDEGs) and a conventional tool (Seurat) so their result lists can be intersected and compared.

## When NOT to use

- The goal is cross-timepoint cell annotation or novel cell detection — use CASi's CrossTimeAnnotation/DetectNovelCells functions instead, not DE comparison.
- The dataset has no multi-timepoint structure (single time point, cross-sectional only) — CASi's GLM models time and time-by-group interaction effects, which are undefined here.
- The expected temporal effect is nonlinear — CASi's GLM framework only detects linear changes of expression by time or time-by-condition interaction, so a single-method comparison built on it would miss nonlinear dynamics.

## Inputs

- Annotated CASiList (gene expression matrices for t0 and t1) for the MCL dataset
- CASiList metadata with Group (binary factor) and Time (numerical or categorical) columns, one row per cell (t0 + t1)
- The same annotated MCL data in a Seurat-compatible object for Seurat DE analysis

## Outputs

- tDEG_results.csv: CASi tDEGs with t-test p < 0.05 (p.cutoff = 0.05, pct.cutoff = 0.3)
- Seurat DEG table at p = 0.05
- Intersection/overlap count of CASi tDEGs and Seurat DEGs (expected: 21 shared genes)
- ggplot2 mean-expression trajectory plots (e.g. IFITM2 and IFITM3 split by responders vs non-responders)
- Hallmark and Reactome pathway enrichment tables for the tDEG set

## How to apply

Load the annotated CASiList and its metadata, verifying that Group is a binary factor, Time is numerical or categorical, and the metadata row count equals total cells (t0 + t1). Run CASi's FindTemporalDEGs with p.cutoff = 0.05 and pct.cutoff = 0.3; the function fits a generalized linear model with iterative feature selection for time effects and time-by-group interaction effects, and genes with t-test p < 0.05 are saved as tDEG_results.csv. Run Seurat differential expression on the same annotated data with the same p-value cutoff, then intersect the two gene lists (e.g. with dplyr) to count shared genes. Validate by confirming the counts match the reported 268 CASi tDEGs, 75 Seurat DEGs, and 21 overlapping genes from the 1469-gene MCL dataset, then support the comparison with mean-expression trajectories (IFITM2, IFITM3 by responders vs non-responders in ggplot2) and Hallmark/Reactome enrichment (MYC, oxidative phosphorylation).

## Related tools

- **CASi** (FindTemporalDEGs fits a GLM with iterative feature selection for time and time-by-group interaction effects to identify tDEGs) — github.com/yizhuo-wang/CASi
- **Seurat** (Runs conventional differential expression on the same annotated MCL data as the comparator method)
- **ggplot2** (Plots mean expression trajectories of IFITM2 and IFITM3 over time split by responders vs non-responders)
- **dplyr** (Compares and intersects the CASi and Seurat gene lists)

## Examples

```
require(CASi); tdegs <- FindTemporalDEGs(CASiList_tDEG_demo, metadata_tDEG_demo, p.cutoff = 0.05, pct.cutoff = 0.3); write.csv(tdegs, "tDEG_results.csv")
```

## Evaluation signals

- CASi tDEG count at p = 0.05 equals 268 out of 1469 total genes
- Seurat DEG count at p = 0.05 equals 75
- CASi-Seurat overlap equals 21 shared genes
- IFITM2 and IFITM3 trajectory plots show opposite directions between responders and non-responders
- Enrichment tables highlight MYC and oxidative phosphorylation among the tDEGs

## Limitations

- CASi's current GLM framework only detects linear effects (linear change of expression by time or time-by-condition interaction), not nonlinear changes
- Version numbers of comparison methods such as Seurat, BPSC, MAST, and Monocle are not stated in the article
- Detailed simulation comparison results (ARI for three scenarios) are only available in supplementary Figure 11, not the main text

## Evidence

- [methods] workflow p.cutoff and pct.cutoff: [evidence span withheld — anchor status: not_found]
- [results] reported counts and overlap: [evidence span withheld — anchor status: not_found]
- [discussion] GLM linearity limitation: "the current GLM framework can only detect linear effect, i.e., the linear change of gene expression by time or the interaction of time and condition"
- [methods] input structure requirements: [evidence span withheld — anchor status: not_found]
- [readme] installation command: [evidence span withheld — anchor status: external_doc]
