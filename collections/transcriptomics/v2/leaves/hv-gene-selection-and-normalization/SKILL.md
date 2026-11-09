---
name: hv-gene-selection-and-normalization
description: Use when preparing population-scale multi-subject scRNA-seq counts (e.g.,
  the Jerber et al. 2021 iPSC dopaminergic differentiation dataset) for personalized
  network construction, where you must choose a gene subset and normalization before
  Dozer/MuDCoD.
when_to_use_negative:
- Input is already a set of per-subject adjacency matrices or community labels — gene
  selection and normalization are already done; go straight to MuDCoD fit_predict.
- The dataset is not multi-subject single-cell RNA-seq (e.g., bulk RNA-seq or a single
  donor/time point) — the personalized network setting that motivates Dozer-based
  filtering does not apply.
- 'You need gene centrality or module-trait association analyses — the paper explicitly
  stops the Dozer workflow before ''Section 4: Gene Centrality Analysis'' because
  those outputs are unnecessary for network generation.'
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Dozer
  role: 'R package used to filter genes and construct robust personalized weighted
    gene co-expression networks; vignette followed up to (not including) ''Section
    4: Gene Centrality Analysis'''
  repo: https://github.com/shanlu01/Dozer
- name: Scanpy
  role: Python library used to select the top 3000 highly variable genes from the
    scRNA-seq count matrix
- name: SCTransform
  role: normalization applied to the expression data before network construction
- name: MuDCoD
  role: 'downstream consumer: expects the normalized, filtered T x S adjacency matrices
    as a multi-dimensional numpy array input'
  repo: https://github.com/bo1929/MuDCoD
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad592
    title: 'MuDCoD: multi-subject community detection in personalized dynamic gene
      networks from single-cell RNA sequencing'
  source_content_hash: sha256:59d588e10231f3b4ab20a356b3df61d2a10c3b052e96c136823dfe8c174e427f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shanlu01/Dozer
  tools:
  - Dozer
  - MuDCoD
  - PisCES
  - SCTransform
  - Scanpy
  - numpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# hv-gene-selection-and-normalization

## Summary

Gene filtering, highly-variable-gene selection, and normalization for personalized dynamic gene co-expression networks: select the top 3000 HVGs with Scanpy, apply SCTransform normalization, then use Dozer to build subject-specific weighted co-expression networks for MuDCoD community detection.

## When to use

Use when preparing population-scale multi-subject scRNA-seq counts (e.g., the Jerber et al. 2021 iPSC dopaminergic differentiation dataset) for personalized network construction, where you must choose a gene subset and normalization before Dozer/MuDCoD.

## When NOT to use

- Input is already a set of per-subject adjacency matrices or community labels — gene selection and normalization are already done; go straight to MuDCoD fit_predict.
- The dataset is not multi-subject single-cell RNA-seq (e.g., bulk RNA-seq or a single donor/time point) — the personalized network setting that motivates Dozer-based filtering does not apply.
- You need gene centrality or module-trait association analyses — the paper explicitly stops the Dozer workflow before 'Section 4: Gene Centrality Analysis' because those outputs are unnecessary for network generation.

## Inputs

- population-scale multi-subject scRNA-seq count matrix with cell metadata (e.g., Jerber et al. 2021 dopaminergic neuron differentiation dataset, DA/Sert/Epen1 cell types)
- subject/donor phenotypic information required by Dozer for personalized network construction
- top-3000-HVG-restricted expression matrix (post-Scanpy/SCTransform filtering)

## Outputs

- set of top 3000 highly variable genes selected with Scanpy
- SCTransform-normalized expression matrix
- one n-by-n adjacency matrix per donor per time point (T x S matrices), each thresholded at its top-5% edges
- a multi-dimensional numpy array (subjects on axis 0, time points on axis 1) saved to disk as MuDCoD input

## How to apply

Download the scRNA-seq counts and cell metadata, restricting to the samples and cell types of interest (in the paper: day-30 and day-52 DA, Sert, and Epen1 cells). Select the top 3000 highly variable genes with Scanpy and apply SCTransform normalization, as in the task workflow. Then follow the Dozer vignette to filter genes and construct robust personalized weighted gene co-expression networks — the paper states it is sufficient to follow the vignette up to but not including 'Section 4: Gene Centrality Analysis'. Threshold each weighted network at its top-5% edges to obtain one n-by-n adjacency matrix per donor per time point (T x S matrices), and collect them into a numpy array with subjects on the first axis and time points on the second, which is the input format MuDCoD's fit_predict expects.

## Related tools

- **Dozer** (R package used to filter genes and construct robust personalized weighted gene co-expression networks; vignette followed up to (not including) 'Section 4: Gene Centrality Analysis') — https://github.com/shanlu01/Dozer
- **Scanpy** (Python library used to select the top 3000 highly variable genes from the scRNA-seq count matrix)
- **SCTransform** (normalization applied to the expression data before network construction)
- **MuDCoD** (downstream consumer: expects the normalized, filtered T x S adjacency matrices as a multi-dimensional numpy array input) — https://github.com/bo1929/MuDCoD

## Examples

```
import scanpy as sc; sc.pp.highly_variable_genes(adata, n_top_genes=3000)
```

## Evaluation signals

- Exactly 3000 highly variable genes are selected (n_top_genes=3000 with Scanpy) and the normalized matrix is restricted to those genes.
- The Dozer vignette is followed only up to 'Section 4: Gene Centrality Analysis' (not included), and networks are output (e.g., in an .rda file) before centrality analysis.
- Each weighted network is thresholded at its top-5% edges, yielding one n-by-n adjacency matrix per donor per time point (T x S total).
- The stacked networks form a multi-dimensional numpy array with subjects on the first axis and time points on the second, matching MuDCoD's expected input format.
- Downstream MuDCoD run converges (monitor_convergence=True) and reproduces the expected donor-similarity behavior (e.g., Wilcoxon P = 0.0003 for MuDCoD vs. PisCES NMI at day-52 in the paper's task).

## Limitations

- Dozer installation requires lcmix from R-Forge and building the vignette takes up to 10 minutes with a long dependency list (ggpubr, ggplot2, ggrepel, Rtsne, Matrix, knitr, dplyr, enrichR, limma, cluster, foreach, doParallel).
- The paper reports no concrete numeric normalization/filtering sensitivity results (e.g., no ARI output values appear in the text), so parameter choices like the top-5% edge threshold are inherited from the vignette rather than validated in-document.
- MuDCoD downstream analysis requires Python 3.9 (installed via conda or pyenv) and poetry-managed dependencies, which constrains the environment the prepared networks can be used in.

## Evidence

- [methods] We suggest to use Dozer [3] to filter genes and construct robust networks: "We suggest to use Dozer [3] to filter genes and construct robust networks"
- [methods] It is sufficient to follow until "Section 4: Gene Centrality Analysis" (not included) for our purposes.: "It is sufficient to follow until "Section 4: Gene Centrality Analysis" (not included) for our purposes."
- [methods] MuDCoD takes multi-dimensional `numpy` arrays as the input network argument: "MuDCoD takes multi-dimensional `numpy` arrays as the input network argument"
- [readme] Dozer is a lightweight R package for personalized co-expression network analysis from population-scale single cell RNA-sequencing datasets.: [evidence span withheld — anchor status: external_doc]
