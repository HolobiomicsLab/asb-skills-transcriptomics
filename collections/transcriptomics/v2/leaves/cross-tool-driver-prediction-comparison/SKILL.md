---
name: cross-tool-driver-prediction-comparison
description: Use when comparing multiple driver-gene prediction tools' gene lists
  against known driver sets (e.g. CGC 616 genes, NCG 711 genes, PMH 501 genes) over
  matched datasets, and recall-weighted performance must be quantified and ranked.
when_to_use_negative:
- When the goal is to discover driver genes de novo rather than to score and rank
  already-produced prediction lists against a benchmark
- When precision should be weighted more heavily than recall — the beta^2 = 0.2 setting
  deliberately weights recall five times more; use balanced F1 or another beta if
  the goal does not prioritize recall
- When no trusted curated gene set (CGC/NCG/PMH-like) is available to serve as ground
  truth for TP/FP/FN counting
tools:
- name: WMDS.net
  role: Subject tool whose driver-gene lists are benchmarked against the 11 comparators
    via F0.2 scores
  repo: https://github.com/chaofen123/WMDS.net
- name: Matlab R2020b
  role: Environment running the upstream intlinprog binary integer-programming step
    that produces the WMDS.net driver-gene lists being scored
  repo: https://github.com/chaofen123/WMDS.net
- name: DawnRank
  role: Source of the Gene_Net interaction network used to construct the WMDS.net
    network on which driver nodes are identified and then benchmarked
  repo: https://github.com/chaofen123/WMDS.net
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad071
    title: 'WMDS.net: a network control framework for identifying key players in transcriptome
      programs'
  source_content_hash: sha256:1f9fae000206403cf91fa8bb0979f96815269f6c0522afa1c14bd451b278725e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chaofen123/WMDS.net
  tools:
  - DawnRank
  - Matlab R2020b
  - WMDS.net
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# cross-tool-driver-prediction-comparison

## Summary

Benchmark driver-gene prediction tool outputs against curated gene sets (CGC, NCG, PMH) by computing precision, recall and the recall-weighted F0.2-measure per cancer type, then ranking all tools by F0.2 score. This reproduces the WMDS.net evaluation scheme, which weights recall far more heavily than precision.

## When to use

Use when comparing multiple driver-gene prediction tools' gene lists against known driver sets (e.g. CGC 616 genes, NCG 711 genes, PMH 501 genes) over matched datasets, and recall-weighted performance must be quantified and ranked.

## When NOT to use

- When the goal is to discover driver genes de novo rather than to score and rank already-produced prediction lists against a benchmark
- When precision should be weighted more heavily than recall — the beta^2 = 0.2 setting deliberately weights recall five times more; use balanced F1 or another beta if the goal does not prioritize recall
- When no trusted curated gene set (CGC/NCG/PMH-like) is available to serve as ground truth for TP/FP/FN counting

## Inputs

- Predicted driver-gene lists per tool per TCGA cancer dataset (e.g. WMDS.net / WMDS.netP outputs from the Matlab R2020b intlinprog step; 11 comparator lists from DriverDBv2)
- Benchmark gene lists: 616 genes from CGC (COSMIC), 711 genes from NCG, 501-gene PMH set (Pan-cancer drivers + Mut-drivers + Hiconf drivers merged)
- Expression matrices (rows = gene symbols, columns = sample ids) for tumor and normal conditions, e.g. TCGA_BLCAtumor / TCGA_BLCAnormal

## Outputs

- Table with columns tool, cancer type, benchmark, F0.2 score for Figures 3B-3D
- Per-tool precision, recall and F0.2-measure values for each of the CGC, NCG and PMH benchmarks across the 14 TCGA datasets
- Tool rankings per benchmark (with F1 robustness recheck)

## How to apply

Load each tool's predicted driver-gene lists for the 14 TCGA cancer datasets (e.g. the WMDS.net/WMDS.netP lists from the upstream Matlab R2020b binary integer-programming step using 'intlinprog'), and retrieve comparator predictions (DrGap, OncoDriveFM, MutSigCV, DriverML, PNC, DawnRank, SCS, DriverNet, ActiveDriver, iPAC, MSEA) from DriverDBv2. Treat each benchmark gene list (616-gene CGC/COSMIC set, 711-gene NCG set, and the 501-gene PMH set merged from Pan-cancer drivers, Mut-drivers and Hiconf drivers) as ground truth. For each tool, cancer type and benchmark, compute precision = TP/(TP+FP), recall = TP/(TP+FN), and the recall-weighted F-measure F_beta = (1+beta^2)*P*R/(beta^2*P+R) with beta^2 = 0.2. Rank all tools per benchmark by F0.2-measure and assemble tool x cancer type x benchmark x F0.2-score into a single table for figures (here Figures 3B-3D). As a robustness check, recompute the comparison with beta^2 = 1 (F1) and confirm the relative tool rankings are unchanged.

## Related tools

- **WMDS.net** (Subject tool whose driver-gene lists are benchmarked against the 11 comparators via F0.2 scores) — https://github.com/chaofen123/WMDS.net
- **Matlab R2020b** (Environment running the upstream intlinprog binary integer-programming step that produces the WMDS.net driver-gene lists being scored) — https://github.com/chaofen123/WMDS.net
- **DawnRank** (Source of the Gene_Net interaction network used to construct the WMDS.net network on which driver nodes are identified and then benchmarked) — https://github.com/chaofen123/WMDS.net

## Examples

```
% Matlab R2020b: upstream driver-node list then F0.2 benchmark scoring
[x,fval] = intlinprog(f,intcon,A,b,Aeq,beq,lb,ub); % P,R = TP/(TP+FP), TP/(TP+FN); F02 = 1.2*P*R/(0.2*P+R)
```

## Evaluation signals

- WMDS.netP and WMDS.net hold the two highest F0.2 values on the CGC benchmark, about 30% above third-ranked PNC
- WMDS.netP and WMDS.net also rank best on the NCG benchmark and within the top three (with ActiveDriver) on the PMH benchmark across the 14 TCGA datasets
- Recomputing with beta^2 = 1 (F1) leaves the relative tool rankings unchanged
- Every tool x cancer type x benchmark cell carries a valid F0.2 in [0, 1] consistent with its precision and recall via F_beta = (1+beta^2)*P*R/(beta^2*P+R)
- The assembled table reproduces the ordering shown in Figures 3B-3D

## Limitations

- The comparison depends on the quality and completeness of DriverDBv2 predictions for the 11 comparator tools; missing per-dataset predictions limit per-cancer-type ranking
- Benchmark truth is anchored to three specific gene sets (616 CGC, 711 NCG, 501 PMH); results may shift under different or updated driver catalogs
- The beta^2 = 0.2 weighting is a deliberate recall-heavy design choice tied to the driver-discovery objective, not a neutral performance metric

## Evidence

- [abstract] outperformed top-tier tools on benchmarks: "WMDS.net is powerful among various cancer datasets and outperformed the other top-tier tools with a better balance between precision and recall"
- [readme] TCGA validation scope: "To confirm its validity, we applied WMDS.net to the discovery of cancer driver genes in RNA-seq datasets from The Cancer Genome Atlas."
- [intro] Matlab implementation of upstream step: "solve the binary integer-programming problem using function “intlinprog” which is available in the Optimization ToolBox of MatLab version R2020b"
- [readme] input data format: "The rows of the expression matrices represent gene expression, and the columns represent samples. The row and column names of the expression matrices are gene symbols and sample ids, respectively."
