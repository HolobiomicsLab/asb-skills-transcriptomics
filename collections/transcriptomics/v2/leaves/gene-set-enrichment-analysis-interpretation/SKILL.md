---
name: gene-set-enrichment-analysis-interpretation
description: Use when a gene list of temporal DEGs from multi-timepoint scRNA-seq
  (e.g., CASi tDEGs at p < 0.05) needs pathway-level interpretation (Hallmark/Reactome)
  and key-gene trajectory visualization.
when_to_use_negative:
- The input is a single-timepoint cross-sectional scRNA-seq dataset with no Time variable
  — CASi's temporal DEG framework and tDEG-based enrichment do not apply
- The goal is unsupervised clustering or dimension reduction rather than interpreting
  a specific differentially expressed gene list
- Time/condition effects are expected to be nonlinear — the GLM behind FindTemporalDEGs
  only detects linear time and time-by-group interaction effects, so enrichment on
  its tDEGs would miss nonlinear dynamics
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: CASi
  role: Generates the tDEG gene set via FindTemporalDEGs (p.cutoff = 0.05, pct.cutoff
    = 0.3) that is the input to enrichment; CASiList and metadata are lazy-loaded
    demo-style inputs
  repo: github.com/yizhuo-wang/CASi
- name: Seurat
  role: Produces a comparator DEG list on the same annotated MCL data to quantify
    overlap with CASi tDEGs (75 DEGs, 21 shared)
- name: ggplot2
  role: Plots mean expression trajectories of tDEG genes (IFITM2, IFITM3) over time,
    split by responders vs non-responders
- name: dplyr
  role: Data manipulation for preparing gene lists and trajectory summary data
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-024-58566-x
    title: 'CASi: A framework for cross-timepoint analysis of single-cell RNA sequencing
      data'
  source_content_hash: sha256:3812190f27173751114fd19cc7b4192273541ba08ad82b03f49c21120555af71
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CASi
  - Seurat
  - dplyr
  - ggplot2
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# gene-set-enrichment-analysis-interpretation

## Summary

Interpret temporal differentially expressed genes (tDEGs) identified from cross-timepoint scRNA-seq (e.g., CASi FindTemporalDEGs output) by performing Hallmark and Reactome pathway enrichment, and validate biological signals via trajectory plots of key genes split by condition.

## When to use

Use when a gene list of temporal DEGs from multi-timepoint scRNA-seq (e.g., CASi tDEGs at p < 0.05) needs pathway-level interpretation (Hallmark/Reactome) and key-gene trajectory visualization.

## When NOT to use

- The input is a single-timepoint cross-sectional scRNA-seq dataset with no Time variable — CASi's temporal DEG framework and tDEG-based enrichment do not apply
- The goal is unsupervised clustering or dimension reduction rather than interpreting a specific differentially expressed gene list
- Time/condition effects are expected to be nonlinear — the GLM behind FindTemporalDEGs only detects linear time and time-by-group interaction effects, so enrichment on its tDEGs would miss nonlinear dynamics

## Inputs

- Annotated CASiList object for the MCL dataset with metadata (Group as binary factor, Time as numerical/categorical; row count equal to total cells at t0 + t1)
- tDEG gene list from CASi FindTemporalDEGs (t-test p < 0.05, saved as tDEG_results.csv)
- Seurat DEG list from the same annotated MCL data for overlap comparison
- Hallmark and Reactome pathway gene-set databases

## Outputs

- Hallmark and Reactome pathway enrichment tables for the tDEG set (highlighting MYC and oxidative phosphorylation)
- tDEG_results.csv with genes passing p < 0.05
- Overlap counts between CASi tDEGs and Seurat DEGs (expected 21 shared genes)
- ggplot2 trajectory plots of mean expression vs time for genes like IFITM2 and IFITM3, split by responders vs non-responders

## How to apply

First obtain the tDEG set by running CASi's FindTemporalDEGs on the annotated CASiList (Group as binary factor, Time numeric/categorical) with p.cutoff = 0.05 and pct.cutoff = 0.3; on the real-world MCL dataset this yields 268 tDEGs among 1469 genes (vs 75 Seurat DEGs, 21 overlapping). Then run Hallmark and Reactome gene-set enrichment on the tDEG list, checking for expected drivers such as MYC and oxidative phosphorylation, and save the enrichment tables. Complement the enrichment with ggplot2 mean-expression trajectory plots of individual genes (e.g., IFITM2 and IFITM3) over time, split by responders vs non-responders, to confirm directionality of the biological story. Correctness is judged by reproducing the reported tDEG/overlap counts and seeing the expected enriched pathways (MYC, oxidative phosphorylation) in the tables.

## Related tools

- **CASi** (Generates the tDEG gene set via FindTemporalDEGs (p.cutoff = 0.05, pct.cutoff = 0.3) that is the input to enrichment; CASiList and metadata are lazy-loaded demo-style inputs) — github.com/yizhuo-wang/CASi
- **Seurat** (Produces a comparator DEG list on the same annotated MCL data to quantify overlap with CASi tDEGs (75 DEGs, 21 shared))
- **ggplot2** (Plots mean expression trajectories of tDEG genes (IFITM2, IFITM3) over time, split by responders vs non-responders)
- **dplyr** (Data manipulation for preparing gene lists and trajectory summary data)

## Evaluation signals

- tDEG count at p = 0.05 matches the reported 268 genes (out of 1469 total) for the MCL dataset
- Seurat comparison yields 75 DEGs with exactly 21 genes overlapping the CASi tDEG set
- Enrichment tables contain the expected highlighted terms: MYC and oxidative phosphorylation (Hallmark and/or Reactome)
- IFITM2 and IFITM3 trajectory plots show opposite directions between responders and non-responders
- Enrichment tables saved as files and Group metadata verified as a binary factor with Time row count equal to t0 + t1 cells

## Limitations

- The GLM framework behind the tDEGs only detects linear effects (linear change of expression by time or time-by-condition interaction), so enrichment will not capture nonlinear temporal patterns
- Low overlap between methods (21 of 268 CASi tDEGs vs 75 Seurat DEGs) means enrichment results are method-dependent; run enrichment on the tDEG list rather than assuming concordance with Seurat DEGs
- Version numbers of comparison tools (e.g., Seurat) are not stated, complicating exact reproduction of the DEG overlap

## Evidence

- [methods] Below are the input parameters of the FindTemporalDEGs function: "Below are the input parameters of the FindTemporalDEGs function"
- [readme] CASi is a comprehensive R package for analyzing multi-timepoint scRNA-seq data: "CASi is a comprehensive R package for analyzing multi-timepoint scRNA-seq data, which provides users with: (1) cross-time points cell annotation"
- [other] At p = 0.05, among a total of 1469 genes, CASi identifies 268 tDEGs and Seurat identifies 75 DEGs, with 21 genes overlapping: [evidence span withheld — anchor status: not_found]
- [discussion] the current GLM framework can only detect linear effect: "the current GLM framework can only detect linear effect, i.e., the linear change of gene expression by time or the interaction of time and condition"
