---
name: tcga-dataset-performance-evaluation
description: Use when comparing cancer driver gene prediction tools on TCGA RNA-seq
  datasets against curated benchmark gene lists (CGC/COSMIC, NCG, PMH) using precision-recall
  F_beta scoring.
when_to_use_negative:
- Input is a single gene list without a curated benchmark (CGC/NCG/PMH) to score against
  — no TP/FP/FN can be defined.
- The aim is discovery of driver nodes rather than head-to-head benchmarking of prediction
  tools (use the WMDS.net network control workflow instead).
edam_operation: http://edamontology.org/operation_3791
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0622
tools:
- name: WMDS.net
  role: Produces the driver gene lists under evaluation on the 14 TCGA cancer datasets
  repo: https://github.com/chaofen123/WMDS.net
- name: Matlab R2020b
  role: Runs the upstream binary integer-programming step via 'intlinprog' from the
    Optimization ToolBox to generate WMDS.net driver lists
- name: DawnRank
  role: Source of the Gene_Net hub interaction network intersected with TCGA genes;
    also one of the 11 compared prediction tools
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad071
    title: 'WMDS.net: a network control framework for identifying key players in transcriptome
      programs'
  source_content_hash: sha256:075f5f2776489ff9cc8ade5491edf69c0f8b5868caaa1a8b4493037360efef68
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chaofen123/WMDS.net
  tools:
  - DawnRank
  - Matlab R2020b
  - WMDS.net
  edam_operation: http://edamontology.org/operation_3791
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# tcga-dataset-performance-evaluation

## Summary

Evaluate driver gene prediction tools (e.g. WMDS.net, WMDS.netP and 11 comparison tools) across 14 TCGA cancer datasets by computing precision, recall and the F_beta measure (beta^2=0.2) against the CGC, NCG and PMH benchmark gene lists, then rank tools per benchmark.

## When to use

Use when comparing cancer driver gene prediction tools on TCGA RNA-seq datasets against curated benchmark gene lists (CGC/COSMIC, NCG, PMH) using precision-recall F_beta scoring.

## When NOT to use

- Input is a single gene list without a curated benchmark (CGC/NCG/PMH) to score against — no TP/FP/FN can be defined.
- The aim is discovery of driver nodes rather than head-to-head benchmarking of prediction tools (use the WMDS.net network control workflow instead).

## Inputs

- Driver gene lists per tool per TCGA cancer type (e.g. from WMDS.net / WMDS.netP integer-programming output)
- Benchmark gene lists: 616 CGC (COSMIC) genes, 711 NCG genes, 501-gene PMH set (Pan-cancer + Mut-drivers + Hiconf drivers)
- Predictions of 11 comparison tools retrieved from DriverDBv2
- 14 TCGA cancer dataset gene lists (tumor vs normal, e.g. TCGA_BLCAtumor / TCGA_BLCAnormal)

## Outputs

- Table with columns tool, cancer type, benchmark (CGC/NCG/PMH) and F0.2 score (assembled for Figures 3B-3D)
- Per-tool precision and recall values per TCGA dataset per benchmark
- Tool rankings per benchmark, plus an F1 (beta^2=1) sensitivity check

## How to apply

Load the driver gene lists for the 14 TCGA cancer types produced by the upstream WMDS binary integer-programming step (solved with Matlab R2020b 'intlinprog'). Retrieve the three benchmark gene lists: 616 genes from CGC (COSMIC), 711 genes from NCG, and the 501-gene PMH set merged from Pan-cancer drivers, Mut-drivers and Hiconf drivers. Pull predictions of the 11 comparison tools (DrGap, OncoDriveFM, MutSigCV, DriverML, PNC, DawnRank, SCS, DriverNet, ActiveDriver, iPAC, MSEA) from the DriverDBv2 database. For each tool and dataset, compute precision = TP/(TP+FP), recall = TP/(TP+FN) and F_beta = (1+beta^2)*P*R/(beta^2*P+R) with beta^2=0.2 against each benchmark. Rank all tools per benchmark by F0.2-measure (as in Figures 3B-3D), then recompute with beta^2=1 (F1) to confirm the relative rankings are unchanged.

## Related tools

- **WMDS.net** (Produces the driver gene lists under evaluation on the 14 TCGA cancer datasets) — https://github.com/chaofen123/WMDS.net
- **Matlab R2020b** (Runs the upstream binary integer-programming step via 'intlinprog' from the Optimization ToolBox to generate WMDS.net driver lists)
- **DawnRank** (Source of the Gene_Net hub interaction network intersected with TCGA genes; also one of the 11 compared prediction tools)

## Examples

```
matlab -batch "intlinprog(...) % upstream WMDS step; then compute F0.2 = (1+0.2)*P.*R./(0.2*P+R) per tool per TCGA dataset vs CGC/NCG/PMH lists"
```

## Evaluation signals

- WMDS.netP and WMDS.net hold the two highest F0.2 values on the CGC benchmark, about 30% higher than third-ranked PNC.
- WMDS.netP and WMDS.net rank top-two on the NCG benchmark.
- WMDS.netP and WMDS.net rank within the top three (with ActiveDriver) on the PMH benchmark across the 14 TCGA datasets.
- Recomputing with beta^2=1 (F1) leaves the relative rankings unchanged.
- Reproduced ordering matches Figures 3B-3D when the table is assembled per benchmark.

## Limitations

- Benchmark coverage differs between lists (616 CGC, 711 NCG, 501 PMH genes), so scores are not directly comparable across benchmarks.
- Comparison tool predictions are restricted to those deposited in DriverDBv2; tools absent from that database cannot be scored in this framework.
- F0.2 weighting (beta^2=0.2) emphasizes precision over recall; conclusions should be checked with F1 as a robustness measure.

## Evidence

- [other] On the CGC benchmark, WMDS.netP and WMDS.net achieved the two highest F0.2-measure values, about 30% higher than PNC which ranked third: [evidence span withheld — anchor status: not_found]
- [other] compute precision = TP/(TP+FP), recall = TP/(TP+FN) and the F-measure F_beta: [evidence span withheld — anchor status: not_found]
- [readme] outperformed the other top-tier tools with a better balance between precision and recall: "WMDS.net is robust and powerful among various cancer datasets and outperformed the other top-tier tools with a better balance between precision and recall."
- [readme] solve the binary integer-programming problem using function "intlinprog" which is available in the Optimization ToolBox of MatLab version R2020b: "solve the binary integer-programming problem using function “intlinprog” which is available in the Optimization ToolBox of MatLab version R2020b"
