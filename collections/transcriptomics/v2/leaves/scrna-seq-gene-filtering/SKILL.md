---
name: scrna-seq-gene-filtering
description: Use when scRNA-seq counts from multiple subjects (e.g., iPSC dopaminergic
  differentiation time points) must be converted into per-donor gene co-expression
  adjacency matrices as input for MuDCoD or similar personalized dynamic community
  detection.
when_to_use_negative:
- The input is already a set of adjacency matrices or a co-expression network — the
  filtering step is only needed upstream of community detection.
- 'The goal is gene centrality analysis rather than producing networks for MuDCoD
  — the article explicitly stops before Dozer''s ''Section 4: Gene Centrality Analysis''.'
- Data are not population-scale scRNA-seq from multiple subjects; Dozer is designed
  for subject-specific co-expression networks with phenotypic subject information.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: Dozer
  role: R package used to filter genes and construct personalized weighted gene co-expression
    networks from population-scale scRNA-seq data; vignette followed up to (not including)
    Section 4
  repo: https://github.com/shanlu01/Dozer
- name: MuDCoD
  role: 'Downstream consumer: takes the collected numpy array of T x S adjacency matrices
    as its input network argument for multi-subject community detection'
  repo: https://github.com/bo1929/MuDCoD
- name: numpy
  role: Holds the T x S adjacency matrices as a multi-dimensional array, which is
    the input format MuDCoD expects
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad592
    title: 'MuDCoD: multi-subject community detection in personalized dynamic gene
      networks from single-cell RNA sequencing'
  source_content_hash: sha256:a77a3425b4fb4bd80081092b7b48fdf4b61f9c63b65c6154b2158327e98d979b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shanlu01/Dozer
  tools:
  - Dozer
  - MuDCoD
  - PisCES
  - numpy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# scrna-seq-gene-filtering

## Summary

Filter genes and construct personalized weighted gene co-expression networks from population-scale scRNA-seq using Dozer, selecting top 3000 highly variable genes with Scanpy and SCTransform normalization, then thresholding each network at its top-5% edges to produce n-by-n adjacency matrices for multi-subject dynamic community detection.

## When to use

Use when scRNA-seq counts from multiple subjects (e.g., iPSC dopaminergic differentiation time points) must be converted into per-donor gene co-expression adjacency matrices as input for MuDCoD or similar personalized dynamic community detection.

## When NOT to use

- The input is already a set of adjacency matrices or a co-expression network — the filtering step is only needed upstream of community detection.
- The goal is gene centrality analysis rather than producing networks for MuDCoD — the article explicitly stops before Dozer's 'Section 4: Gene Centrality Analysis'.
- Data are not population-scale scRNA-seq from multiple subjects; Dozer is designed for subject-specific co-expression networks with phenotypic subject information.

## Inputs

- scRNA-seq count matrix and cell metadata from multiple subjects and time points (e.g., Jerber et al. 2021 dopaminergic neuron differentiation dataset)
- Cell-type labels for filtering (DA, Sert, Epen1)
- Dozer vignette pipeline (installed via devtools::install_github('shanlu01/Dozer'))

## Outputs

- T x S multi-dimensional numpy array of n-by-n adjacency matrices (subjects on axis 0, time points on axis 1), one thresholded top-5% network per donor per time point
- Filtered gene set (top 3000 highly variable genes)

## How to apply

Download the scRNA-seq counts and cell metadata (e.g., Jerber et al. 2021 dopaminergic neuron differentiation), restrict to the target time points and cell types (day-30 and day-52; DA, Sert, and Epen1 per Section 3.2.1). Select the top 3000 highly variable genes with Scanpy and apply SCTransform normalization, then use Dozer to filter genes and construct robust personalized weighted co-expression networks per subject and time point. Threshold each weighted network at its top-5% edges to obtain one n-by-n adjacency matrix per donor per time point (T x S matrices), following the Dozer vignette up to but not including 'Section 4: Gene Centrality Analysis'. Collect the T x S adjacency matrices into a multi-dimensional numpy array (subjects on the first axis, time points on the second) so MuDCoD's fit_predict can consume them directly.

## Related tools

- **Dozer** (R package used to filter genes and construct personalized weighted gene co-expression networks from population-scale scRNA-seq data; vignette followed up to (not including) Section 4) — https://github.com/shanlu01/Dozer
- **MuDCoD** (Downstream consumer: takes the collected numpy array of T x S adjacency matrices as its input network argument for multi-subject community detection) — https://github.com/bo1929/MuDCoD
- **numpy** (Holds the T x S adjacency matrices as a multi-dimensional array, which is the input format MuDCoD expects)

## Examples

```
devtools::install_github("shanlu01/Dozer", build_vignettes = FALSE); load(system.file("extdata", "Jerber_demo.rda", package = "Dozer"))
```

## Evaluation signals

- Number of genes retained equals 3000 (top highly variable genes selected with Scanpy).
- Each network is thresholded at its top-5% edges, yielding one n-by-n adjacency matrix per donor per time point (T x S matrices total).
- Adjacency matrices are symmetric, weighted, and assembled with subjects on the first axis and time points on the second of the numpy array.
- Downstream MuDCoD fit_predict accepts the array without reshaping errors (MuDCoD takes multi-dimensional numpy arrays as the network input).
- Dozer vignette steps completed up to but not including 'Section 4: Gene Centrality Analysis'.

## Limitations

- Dozer requires phenotypic information of each subject; it is designed for population-scale scRNA-seq, not single-subject datasets.
- The vignette build requires many dependencies (ggpubr, ggplot2, ggrepel, Rtsne, Matrix, knitr, dplyr, enrichR, limma, cluster, foreach, doParallel) and takes up to 10 minutes.
- lcmix must be installed from R-Forge and Dozer via devtools::install_github, which can be fragile.
- The article reports no numeric ARI/NMI values in its methods text for the simulation path, so correctness must be judged by reproducing the downstream Wilcoxon comparison (P = 0.0003 at day-52 vs PisCES) rather than checking printed numbers.

## Evidence

- [methods] We suggest to use Dozer [3] to filter genes and construct robust networks: "We suggest to use Dozer [3] to filter genes and construct robust networks"
- [methods] It is sufficient to follow until 'Section 4: Gene Centrality Analysis' (not included) for our purposes.: "It is sufficient to follow until "Section 4: Gene Centrality Analysis" (not included) for our purposes."
- [methods] MuDCoD takes multi-dimensional numpy arrays as the input network argument: "MuDCoD takes multi-dimensional `numpy` arrays as the input network argument"
- [readme] Dozer is a lightweight R package for personalized co-expression network analysis from population-scale single cell RNA-sequencing datasets.: [evidence span withheld — anchor status: external_doc]
- [readme] devtools::install_github('shanlu01/Dozer', build_vignettes = FALSE): [evidence span withheld — anchor status: external_doc]
