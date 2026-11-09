---
name: gene-coexpression-network-degree-analysis
description: Use when you have constructed gene co-expression (or differential co-expression)
  networks and need to validate their topology as approximately scale-free before
  downstream hub/driver-node analyses.
when_to_use_negative:
- Input is not a co-expression network but an already-validated scale-free network
  where topology checking is redundant
- The goal is driver-node identification (e.g., running WMDS.net's intlinprog-based
  controllability optimization) rather than topology validation
- The network is too small for a meaningful degree distribution fit (few degree values
  make the log-log regression unstable)
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3366
tools:
- name: Matlab R2020b
  role: Implement the degree-distribution computation, log2-log2 linear regression
    fit, and R-square calculation for each cancer-type network
- name: WMDS.net
  role: Upstream framework whose differential co-expression networks (built from TCGA
    RNA-seq tumor/normal expression matrices) are the input networks whose topology
    is assessed
  repo: https://github.com/chaofen123/WMDS.net
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad071
    title: 'WMDS.net: a network control framework for identifying key players in transcriptome
      programs'
  source_content_hash: sha256:c0e0241309389b3e50bb1a293c2b60d1b22d004d4ee79de6088555e2a54d2865
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chaofen123/WMDS.net
  tools:
  - Matlab R2020b
  - WMDS.net
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# gene-coexpression-network-degree-analysis

## Summary

Assess whether differential gene co-expression networks from TCGA cancer types satisfy an approximate scale-free topology by fitting log2(degree frequency) vs log2(degree) with linear regression and checking the R-square against Zhang and Horvath's R2 > 0.80 criterion.

## When to use

Use when you have constructed gene co-expression (or differential co-expression) networks and need to validate their topology as approximately scale-free before downstream hub/driver-node analyses.

## When NOT to use

- Input is not a co-expression network but an already-validated scale-free network where topology checking is redundant
- The goal is driver-node identification (e.g., running WMDS.net's intlinprog-based controllability optimization) rather than topology validation
- The network is too small for a meaningful degree distribution fit (few degree values make the log-log regression unstable)

## Inputs

- Differential co-expression networks for 14 TCGA cancer types (BLCA, BRCA, COAD, HNSC, KICH, KIRC, KIRP, LIHC, LUAD, LUSC, PRAD, STAD, THCA, UCEC) from the upstream network-construction task
- Gene_Net hub network (intersection of the DawnRank gene interaction network and TCGA genes) as the network background, if needed for node-degree context

## Outputs

- scale_free_r2.csv with columns for cancer type and R2
- Per-cancer-type log2(degree frequency) vs log2(degree) scatter plots with fitted linear regression lines

## How to apply

Load each differential co-expression network (e.g., the 14 TCGA cancer types: BLCA, BRCA, COAD, HNSC, KICH, KIRC, KIRP, LIHC, LUAD, LUSC, PRAD, STAD, THCA, UCEC) produced by the upstream network-construction step. For each network, compute the node degree distribution as the frequency of each degree value. Plot and fit log2(numbers of degree) versus log2(degree) using a linear regression model, replicating the paper's scale-free topology check (as in Figure 2C). Record the R-square of each fit, applying the Zhang and Horvath acceptance threshold of R2 > 0.80 for an adequate scale-free approximation; the paper reported R2 > 0.9 on most of the 14 TCGA datasets. Write per-cancer-type R-square values to scale_free_r2.csv with columns for cancer type and R2.

## Related tools

- **Matlab R2020b** (Implement the degree-distribution computation, log2-log2 linear regression fit, and R-square calculation for each cancer-type network)
- **WMDS.net** (Upstream framework whose differential co-expression networks (built from TCGA RNA-seq tumor/normal expression matrices) are the input networks whose topology is assessed) — https://github.com/chaofen123/WMDS.net

## Examples

```
R = polyfit(log2(degrees), log2(degreeFreq), 1); r2 = 1 - sum((log2(degreeFreq)-polyval(R,log2(degrees))).^2)/sum((log2(degreeFreq)-mean(log2(degreeFreq))).^2)
```

## Evaluation signals

- Each reported R-square exceeds the Zhang and Horvath acceptance threshold of 0.80
- Most (expected >0.9) of the 14 TCGA cancer-type networks yield R-square values above 0.9, matching the paper's Figure 2C report
- scale_free_r2.csv contains exactly 14 rows (one per TCGA cancer type) with cancer type and R2 columns populated
- Degree frequencies are computed as counts of nodes per degree value before log2 transformation; verify no zero-frequency log2 entries are fed to the regression

## Limitations

- The R2 > 0.80 threshold is an approximate-scale-free criterion (Zhang and Horvath); values below it indicate the network does not adequately satisfy scale-free topology and downstream hub-based conclusions may be unreliable
- The assessment depends on the upstream differential co-expression networks constructed from TCGA tumor/normal expression matrices; errors in those networks propagate to the topology check
- The paper's pipeline is implemented in Matlab R2020b (function 'intlinprog' from the Optimization ToolBox); porting to other languages requires reproducing the same degree-frequency and regression steps

## Evidence

- [results] R-square values of the log2(degree frequency) vs log2(degree) linear regression exceed 0.9 on most of the 14 TCGA differential co-expression networks: [evidence span withheld — anchor status: not_found]
- [readme] Matlab toolchain quote: "we implement the algorithm using the Matlab program language and solve the binary integer-programming problem using function "intlinprog" which is available in the Optimization ToolBox of MatLab"
- [intro] WMDS.net scope quote: "We developed a weighted MDS network model (WMDS.net) to find the drive nodes of differential gene co-expression networks."
- [readme] Gene_Net definition quote: "Gene_Net is the hub network of intersection of gene interaction network constructed in Dawnrank[1] and the genes in TCGA."
