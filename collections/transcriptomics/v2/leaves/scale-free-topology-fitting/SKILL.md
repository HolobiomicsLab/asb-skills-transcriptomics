---
name: scale-free-topology-fitting
description: Use when you have a constructed gene co-expression or differential co-expression
  network (adjacency or edge list) and need to validate that its degree distribution
  approximately follows a scale-free topology before downstream driver-node analysis.
when_to_use_negative:
- Input is still raw TCGA expression matrices (e.g. TCGA_BLCAtumor / TCGA_BLCAnormal)
  rather than a constructed co-expression network — network construction must happen
  first
- The goal is to rank key nodes / driver genes in the network — that is WMDS.net's
  controllability (intlinprog) step, not topology validation
- The network is small or sparse such that a degree-frequency histogram is too discontinuous
  for a meaningful log-log regression
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: Matlab R2020b
  role: Environment for computing degree distributions and fitting the log2(degree
    frequency) vs log2(degree) linear regression to obtain R2 per cancer type
- name: WMDS.net
  role: Upstream framework whose differential gene co-expression networks over TCGA
    RNA-seq datasets are the objects being validated for scale-free topology
  repo: https://github.com/chaofen123/WMDS.net
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad071
    title: 'WMDS.net: a network control framework for identifying key players in transcriptome
      programs'
  source_content_hash: sha256:fdcffb67b814d3ad3be7b53cb0162d3a466721399ab9810beb5aeaa72f4fde87
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chaofen123/WMDS.net
  tools:
  - Matlab R2020b
  - WMDS.net
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# scale-free-topology-fitting

## Summary

Assesses whether a differential co-expression network approximates a scale-free topology by fitting log2(degree frequency) versus log2(degree) with linear regression and reporting the R-square, following the Zhang and Horvath criterion (R2 > 0.80) used to validate the 14 TCGA cancer-type networks in WMDS.net.

## When to use

Use when you have a constructed gene co-expression or differential co-expression network (adjacency or edge list) and need to validate that its degree distribution approximately follows a scale-free topology before downstream driver-node analysis.

## When NOT to use

- Input is still raw TCGA expression matrices (e.g. TCGA_BLCAtumor / TCGA_BLCAnormal) rather than a constructed co-expression network — network construction must happen first
- The goal is to rank key nodes / driver genes in the network — that is WMDS.net's controllability (intlinprog) step, not topology validation
- The network is small or sparse such that a degree-frequency histogram is too discontinuous for a meaningful log-log regression

## Inputs

- Differential co-expression network (adjacency matrix or edge list) per cancer type, from the upstream WMDS.net network-construction task
- Node degree sequence derived from each network's connectivity

## Outputs

- scale_free_r2.csv with columns for cancer type and R2 of the log2(degree frequency) vs log2(degree) linear fit
- Log-log degree-distribution plots with fitted regression lines per cancer type

## How to apply

Load each differential co-expression network (e.g. the 14 TCGA cancer-type networks: BLCA, BRCA, COAD, HNSC, KICH, KIRC, KIRP, LIHC, LUAD, LUSC, PRAD, STAD, THCA, UCEC) produced by the upstream network-construction step. For each network, compute the node degree distribution, i.e. the frequency (count) of each distinct degree value. Fit log2(numbers of degree) versus log2(degree) using a linear regression model, as in the paper's scale-free topology check, implemented in Matlab R2020b. Record the R-square of each fit and apply the acceptance criterion R2 > 0.80 (Zhang and Horvath); the paper reported R2 > 0.9 on most of the 14 TCGA datasets (Figure 2C). Write per-cancer-type R2 values to scale_free_r2.csv.

## Related tools

- **Matlab R2020b** (Environment for computing degree distributions and fitting the log2(degree frequency) vs log2(degree) linear regression to obtain R2 per cancer type)
- **WMDS.net** (Upstream framework whose differential gene co-expression networks over TCGA RNA-seq datasets are the objects being validated for scale-free topology) — https://github.com/chaofen123/WMDS.net

## Examples

```
p = polyfit(log2(degrees), log2(degfreq), 1); r2 = corr(log2(degfreq), polyval(p, log2(degrees)))^2;
```

## Evaluation signals

- Every reported R2 exceeds the paper's acceptance threshold of 0.80 (Zhang and Horvath), with most of the 14 TCGA datasets exceeding 0.9 as in Figure 2C
- scale_free_r2.csv contains exactly 14 rows (one per TCGA cancer type: BLCA, BRCA, COAD, HNSC, KICH, KIRC, KIRP, LIHC, LUAD, LUSC, PRAD, STAD, THCA, UCEC) with cancer-type and R2 columns
- Degree frequencies summed over all degree values equal the number of nodes in each input network, and all degrees lie in [0, N-1]
- Regression is fit on log2-transformed degree counts versus log2-transformed degrees (not raw values), reproducing the paper's scale-free check
- Any cancer type with R2 <= 0.80 is flagged as failing the approximate scale-free criterion rather than silently passed

## Limitations

- R2 > 0.80 is an approximation criterion, not proof of exact scale-free behavior; borderline fits may be accepted
- The check depends on the quality of the upstream differential co-expression networks, which were built from TCGA RNA-seq tumor/normal expression matrices
- Sparse or low-degree networks can yield noisy log-log degree-frequency points, degrading the linear fit

## Evidence

- [abstract] WMDS.net network model context: "we developed a weighted MDS network model (WMDS.net) to find the driver nodes of differential gene co-expression networks"
- [intro] TCGA RNA-seq application scope: "we applied WMDS.net to the discovery of cancer driver genes in RNA-seq datasets from The Cancer Genome Atlas."
- [intro] Matlab implementation environment: "solve the binary integer-programming problem using function “intlinprog” which is available in the Optimization ToolBox of MatLab version R2020b"
- [readme] Expression matrix layout for the tested networks: "The rows of the expression matrices represent gene expression, and the columns represent samples. The row and column names of the expression matrices are gene symbols and sample ids, respectively."
