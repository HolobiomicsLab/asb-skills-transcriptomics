---
name: f-measure-precision-recall-computation
description: Use when ranking gene-prediction tools by F0.2-measure against curated
  driver gene lists (CGC 616 genes, NCG 711 genes, PMH 501 genes) over multiple TCGA
  cancer-type prediction sets.
when_to_use_negative:
- When the analysis goal favors recall over precision (e.g. discovering novel candidate
  drivers), since the beta^2 = 0.2 setting deliberately underweights recall
- When the candidate predictions are continuous confidence scores rather than gene
  sets — precision/recall as defined here requires binary gene lists and does not
  use thresholds or AUC-style ranking
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_2640
tools:
- name: WMDS.net
  role: Produces the driver gene prediction lists (via Matlab R2020b intlinprog) that
    are scored with the F0.2-measure; also released as the comparison repository
  repo: https://github.com/chaofen123/WMDS.net
- name: Matlab R2020b
  role: Runs the WMDS.net binary integer-programming step (intlinprog) whose gene
    lists feed the F0.2 benchmarking
- name: DawnRank
  role: Source of the gene interaction network (Gene_Net hub network) underlying WMDS.net
    predictions; its predictions are also among the 11 competitors from DriverDBv2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad071
    title: 'WMDS.net: a network control framework for identifying key players in transcriptome
      programs'
  source_content_hash: sha256:4b2964ad57973153c3f21231449c169cfc7841025d33f4ff25e2dca706fc2fb8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chaofen123/WMDS.net
  tools:
  - DawnRank
  - Matlab R2020b
  - WMDS.net
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# f-measure-precision-recall-computation

## Summary

Compute precision, recall, and the F_beta-measure (with beta^2 = 0.2, plus an F1 robustness check) to score cancer driver gene predictions against the CGC, NCG and PMH benchmark gene lists across 14 TCGA datasets, as used to benchmark WMDS.net/WMDS.netP against 11 competitor tools.

## When to use

Use when ranking gene-prediction tools by F0.2-measure against curated driver gene lists (CGC 616 genes, NCG 711 genes, PMH 501 genes) over multiple TCGA cancer-type prediction sets.

## When NOT to use

- When the analysis goal favors recall over precision (e.g. discovering novel candidate drivers), since the beta^2 = 0.2 setting deliberately underweights recall
- When the candidate predictions are continuous confidence scores rather than gene sets — precision/recall as defined here requires binary gene lists and does not use thresholds or AUC-style ranking

## Inputs

- Predicted driver gene lists per tool per cancer type (WMDS.net and WMDS.netP from the Matlab R2020b intlinprog WMDS step; 11 competitor predictions retrieved from DriverDBv2)
- Benchmark gene lists: 616 genes from CGC (COSMIC), 711 genes from NCG, and the 501-gene PMH set (merged from Pan-cancer drivers, Mut-drivers and Hiconf drivers)
- 14 TCGA cancer dataset identifiers defining which prediction sets are scored

## Outputs

- Per-tool, per-dataset precision, recall, and F0.2-measure values against each of the CGC, NCG and PMH benchmarks
- A ranked table with columns for tool, cancer type, benchmark and F0.2 score, assembled for Figures 3B-3D
- Supplementary F1 (beta^2 = 1) recomputation confirming unchanged relative rankings

## How to apply

For each tool's predicted driver list on each TCGA dataset, classify predictions against a benchmark gene list (CGC 616, NCG 711, or PMH 501 genes) as TP or FP, and count the benchmark genes not recovered as FN. Compute precision = TP/(TP+FP), recall = TP/(TP+FN), and the F-measure F_beta = (1+beta^2)*P*R/(beta^2*P+R) with beta^2 = 0.2, which weights precision more heavily than recall. Use the beta^2 = 0.2 setting to favor precision because driver gene lists are dominated by false positives; recompute with beta^2 = 1 (F1) to confirm relative tool rankings are unchanged. Rank all 12 tools (WMDS.net, WMDS.netP, DrGap, OncoDriveFM, MutSigCV, DriverML, PNC, DawnRank, SCS, DriverNet, ActiveDriver, iPAC, MSEA predictions sourced from DriverDBv2) per benchmark and assemble tool/cancer-type/benchmark/F0.2 rows for Figures 3B-3D.

## Related tools

- **WMDS.net** (Produces the driver gene prediction lists (via Matlab R2020b intlinprog) that are scored with the F0.2-measure; also released as the comparison repository) — https://github.com/chaofen123/WMDS.net
- **Matlab R2020b** (Runs the WMDS.net binary integer-programming step (intlinprog) whose gene lists feed the F0.2 benchmarking)
- **DawnRank** (Source of the gene interaction network (Gene_Net hub network) underlying WMDS.net predictions; its predictions are also among the 11 competitors from DriverDBv2)

## Examples

```
F = (1+0.2**2)*P*R/(0.2**2*P+R)  # P=TP/(TP+FP), R=TP/(TP+FN) vs CGC/NCG/PMH benchmark gene lists
```

## Evaluation signals

- WMDS.netP and WMDS.net hold the two highest F0.2 values on the CGC benchmark, about 30% higher than third-ranked PNC
- WMDS.netP and WMDS.net rank top-two on the NCG benchmark and within the top three (with ActiveDriver) on the PMH benchmark across the 14 TCGA datasets
- Recomputing with beta^2 = 1 (F1) leaves the relative tool rankings unchanged
- Precision and recall values satisfy precision = TP/(TP+FP) and recall = TP/(TP+FN) with F0.2 = (1+0.2)*P*R/(0.2*P+R) recomputed identically per tool/dataset/benchmark cell

## Limitations

- Scores depend entirely on the chosen benchmark gene lists (CGC, NCG, PMH); different curation of driver genes will change the ordering
- The F0.2-measure deliberately overweights precision, so tools with balanced precision-recall tradeoffs can be penalized relative to high-precision/low-recall predictions
- Competitor predictions are taken from DriverDBv2, so results reflect that database's versions of the 11 tools rather than freshly rerun software

## Evidence

- [other] F-measure definition with beta^2 = 0.2: [evidence span withheld — anchor status: not_found]
- [results] Expected ranking on CGC benchmark: [evidence span withheld — anchor status: not_found]
- [other] Robustness check with F1: [evidence span withheld — anchor status: not_found]
- [readme] Outperformed other tools: "WMDS.net is robust and powerful among various cancer datasets and outperformed the other top-tier tools with a better balance between precision and recall"
