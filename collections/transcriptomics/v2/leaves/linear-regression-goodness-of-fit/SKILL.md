---
name: linear-regression-goodness-of-fit
description: Use when you have a constructed co-expression network and must verify
  it satisfies the approximate scale-free topology criterion (R2 > 0.80 per Zhang
  and Horvath) before downstream node-controllability analysis.
when_to_use_negative:
- The network has not yet been constructed from TCGA tumor/normal expression matrices
  (e.g. only raw count matrices are available)
- The goal is driver-node identification itself — use the WMDS.net controllability
  model, not the topology check
- The network type is not a differential gene co-expression network (e.g. protein
  interaction or hub networks like Gene_Net)
tools:
- name: Matlab R2020b
  role: Implementation environment for the linear regression fit of log2(degree frequency)
    versus log2(degree) in the WMDS.net pipeline
  repo: https://github.com/chaofen123/WMDS.net
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad071
    title: 'WMDS.net: a network control framework for identifying key players in transcriptome
      programs'
  source_content_hash: sha256:d87a6d2172067ba09899338f2cd0b9676a5bbc43d3195095243ce6f0808548aa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chaofen123/WMDS.net
  tools:
  - Matlab R2020b
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# linear-regression-goodness-of-fit

## Summary

Assesses approximate scale-free topology of the 14 TCGA differential co-expression networks (BLCA through UCEC) by fitting log2(degree frequency) versus log2(degree) with linear regression and checking the R-square goodness-of-fit.

## When to use

Use when you have a constructed co-expression network and must verify it satisfies the approximate scale-free topology criterion (R2 > 0.80 per Zhang and Horvath) before downstream node-controllability analysis.

## When NOT to use

- The network has not yet been constructed from TCGA tumor/normal expression matrices (e.g. only raw count matrices are available)
- The goal is driver-node identification itself — use the WMDS.net controllability model, not the topology check
- The network type is not a differential gene co-expression network (e.g. protein interaction or hub networks like Gene_Net)

## Inputs

- 14 TCGA cancer-type differential co-expression networks (BLCA, BRCA, COAD, HNSC, KICH, KIRC, KIRP, LIHC, LUAD, LUSC, PRAD, STAD, THCA, UCEC)
- Node degree values for each network

## Outputs

- scale_free_r2.csv with columns for cancer type and R2
- Per-network log2(degree frequency) vs log2(degree) plots with fitted linear regression

## How to apply

Load each of the 14 TCGA cancer-type differential co-expression networks (BLCA, BRCA, COAD, HNSC, KICH, KIRC, KIRP, LIHC, LUAD, LUSC, PRAD, STAD, THCA, UCEC) produced by upstream network construction. For each network, compute the node degree distribution (frequency of each degree value), then fit log2(numbers of degree) versus log2(degree) with a linear regression model, replicating the paper's scale-free topology check. Record the R-square of each fit, applying the paper's acceptance threshold R2 > 0.80, with most datasets reported to exceed 0.9 (Figure 2C). The analysis is implemented in MatLab R2020b, the same environment used for the WMDS.net workflow. Write per-cancer-type values to scale_free_r2.csv with columns for cancer type and R2.

## Related tools

- **Matlab R2020b** (Implementation environment for the linear regression fit of log2(degree frequency) versus log2(degree) in the WMDS.net pipeline) — https://github.com/chaofen123/WMDS.net

## Examples

```
p = polyfit(log2(degree), log2(freq), 1); R2 = corr(log2(degree), log2(freq))^2; % Matlab R2020b, per TCGA cancer-type network
```

## Evaluation signals

- Each cancer type's R-square exceeds the paper's acceptance threshold R2 > 0.80 (Zhang and Horvath)
- Most of the 14 TCGA differential co-expression networks report R2 > 0.9, matching Figure 2C of the paper
- scale_free_r2.csv contains exactly 14 rows, one per TCGA cancer type, with cancer-type and R2 columns populated
- Fitted line is computed on log2-transformed axes: log2(numbers of degree) versus log2(degree), not raw degree counts

## Limitations

- The R-square criterion is an approximate scale-free check, not a strict power-law test; passing R2 > 0.80 does not guarantee exact scale-free behavior
- Sensitivity depends on binning of degree frequencies; degree-1 or very high-degree nodes may distort the log2 fit on small networks
- Findings are specific to the 14 TCGA differential co-expression networks; no claim is made for other network types

## Evidence

- [methods] fit log2(numbers of degree) versus log2(degree) using a linear regression model: [evidence span withheld — anchor status: not_found]
- [results] R-square values exceed 0.9 on most of the 14 networks: [evidence span withheld — anchor status: not_found]
- [readme] Matlab R2020b implementation environment: "solve the binary integer-programming problem using function "intlinprog" which is available in the Optimization ToolBox of MatLab version R2020b"
