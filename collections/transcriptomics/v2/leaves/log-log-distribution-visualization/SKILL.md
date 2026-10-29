---
name: log-log-distribution-visualization
description: Use when you have an adjacency network (e.g., a differential gene co-expression
  network) and must verify its degree distribution follows an approximate scale-free
  topology before downstream node-control analysis.
when_to_use_negative:
- The input network was not constructed from the paper's differential co-expression
  workflow (e.g., it is an unvalidated random graph or a directed interaction network
  such as the DawnRank-derived Gene_Net hub network), so the scale-free assumption
  does not apply
- You need to test a different topology hypothesis (e.g., small-world or power-law
  via maximum-likelihood fitting) rather than the paper's log2-log2 linear-regression
  R-square criterion
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0602
tools:
- name: Matlab R2020b
  role: Computing node degree distributions and the log2-transformed linear regression
    fits (intlinprog from the Optimization ToolBox supports the surrounding WMDS.net
    integer-programming steps)
- name: WMDS.net
  role: Upstream framework whose differential gene co-expression networks (from TCGA
    RNA-seq, e.g., TCGA_BLCAtumor vs TCGA_BLCAnormal) are the objects being checked
    for scale-free topology
  repo: https://github.com/chaofen123/WMDS.net
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad071
    title: 'WMDS.net: a network control framework for identifying key players in transcriptome
      programs'
  source_content_hash: sha256:cae281257b1ee0e41c8cc565a22be8f0854688d5fc2c9df999ef067141fc0293
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chaofen123/WMDS.net
  tools:
  - Matlab R2020b
  - WMDS.net
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# log-log-distribution-visualization

## Summary

Assesses whether a differential co-expression network approximates a scale-free topology by fitting log2(degree frequency) versus log2(degree) with linear regression and checking the R-square, as done for the 14 TCGA cancer-type networks in the WMDS.net framework.

## When to use

Use when you have an adjacency network (e.g., a differential gene co-expression network) and must verify its degree distribution follows an approximate scale-free topology before downstream node-control analysis.

## When NOT to use

- The input network was not constructed from the paper's differential co-expression workflow (e.g., it is an unvalidated random graph or a directed interaction network such as the DawnRank-derived Gene_Net hub network), so the scale-free assumption does not apply
- You need to test a different topology hypothesis (e.g., small-world or power-law via maximum-likelihood fitting) rather than the paper's log2-log2 linear-regression R-square criterion

## Inputs

- Differential co-expression network (adjacency/edge list) per TCGA cancer type (14 types: BLCA, BRCA, COAD, HNSC, KICH, KIRC, KIRP, LIHC, LUAD, LUSC, PRAD, STAD, THCA, UCEC)
- Node degree vector derived from each network's adjacency structure

## Outputs

- Per-cancer-type R-square values of the log2(degree frequency) vs log2(degree) linear regression, written to scale_free_r2.csv (columns: cancer type, R2)
- Log-log degree-distribution scatter plots with fitted regression line (as in paper Figure 2C)

## How to apply

Load each pre-built network (here: the 14 TCGA cancer types BLCA, BRCA, COAD, HNSC, KICH, KIRC, KIRP, LIHC, LUAD, LUSC, PRAD, STAD, THCA, UCEC) and compute the frequency of each node degree value. Transform both axes with log2 and fit a linear regression of log2(numbers of degree) versus log2(degree), mirroring the paper's scale-free check (Figure 2C) and the Zhang and Horvath criterion. Record the R-square per cancer type and write the table to scale_free_r2.csv with columns for cancer type and R2. Acceptance threshold is R2 > 0.80 per cancer type; the paper reports R2 > 0.9 on most of the 14 TCGA differential co-expression networks, so values substantially below 0.8 indicate the topology check failed and upstream network construction should be revisited. The WMDS.net pipeline implements this in Matlab (R2020b), where intlinprog from the Optimization ToolBox is used in related integer-programming steps.

## Related tools

- **Matlab R2020b** (Computing node degree distributions and the log2-transformed linear regression fits (intlinprog from the Optimization ToolBox supports the surrounding WMDS.net integer-programming steps))
- **WMDS.net** (Upstream framework whose differential gene co-expression networks (from TCGA RNA-seq, e.g., TCGA_BLCAtumor vs TCGA_BLCAnormal) are the objects being checked for scale-free topology) — https://github.com/chaofen123/WMDS.net

## Examples

```
mdl = fitlm(log2(degree), log2(degreeFrequency)); r2 = mdl.Rsquared.Ordinary; % per cancer type, collect into scale_free_r2.csv
```

## Evaluation signals

- scale_free_r2.csv exists with exactly 14 rows and columns for cancer type and R2
- Every reported R2 exceeds the paper's acceptance threshold of 0.80 (Zhang and Horvath criterion)
- Most (expected: the majority) of the 14 TCGA cancer-type R2 values exceed 0.9, reproducing Figure 2C of the paper
- Degree frequencies are recomputed from the adjacency structure (sum over rows/columns) and match the network edge counts
- The linear fit is performed on log2-transformed degree frequencies vs log2-transformed degrees, not on raw counts

## Limitations

- The R-square of the log-log regression is an approximate goodness-of-fit criterion, not a formal power-law test; networks failing R2 > 0.80 signal that upstream network construction may need adjustment
- The check is specific to the undirected node-degree distribution of the differential co-expression networks; it does not validate edge weights or the significance of gene co-expression differences integrated by WMDS.net

## Evidence

- [intro] Matlab implementation of WMDS.net pipeline: "solve the binary integer-programming problem using function “intlinprog” which is available in the Optimization ToolBox of MatLab version R2020b"
- [intro] Network objects being assessed: "We developed a weighted MDS network model (WMDS.net) to find the drive nodes of differential gene co-expression networks."
- [readme] Degree and co-expression-difference signals integrated per node: "WMDS.net integrates the degree of nodes in the network and the significance of gene co-expression difference between two physiological states into the measurement of node controllability"
- [readme] TCGA input matrices defining the networks: "The inputs to the main program are the expression matrices of the two conditions (which are TCGA_BLCAtumor and TCGA_BLCAnormal in the tested data)"
