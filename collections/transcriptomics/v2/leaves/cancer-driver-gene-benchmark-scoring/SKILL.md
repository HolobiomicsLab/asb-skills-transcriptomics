---
name: cancer-driver-gene-benchmark-scoring
description: Use when ranking driver-gene prediction lists (your own or tools like
  PNC, DawnRank, MutSigCV) against curated CGC, NCG or PMH benchmark gene sets with
  precision, recall and a precision-weighted F0.2 score across TCGA cancer types.
when_to_use_negative:
- Input is still raw expression matrices (e.g. TCGA_BLCAtumor vs TCGA_BLCAnormal)
  rather than gene-symbol driver lists — the upstream WMDS.net integer-programming
  prediction step must run first.
- The cancer type or study has no meaningful overlap with the CGC, NCG or PMH curated
  driver sets, so TP/FP/FN counts and F0.2 values would be uninformative.
- The goal is to run driver discovery itself (constructing Gene_Net / differential
  co-expression networks and solving the integer program) rather than to score already-produced
  gene lists.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: WMDS.net
  role: 'Predictor under evaluation: its (and WMDS.netP''s) driver gene lists for
    14 TCGA cancer types are scored with F0.2 against CGC, NCG and PMH benchmarks'
  repo: https://github.com/chaofen123/WMDS.net
- name: Matlab R2020b
  role: 'Implementation environment: runs the upstream ''intlinprog'' binary integer
    programming that produces the WMDS.net driver lists, and computes precision/recall/F_beta
    for the benchmark table'
- name: DawnRank
  role: One of the 11 comparison tools scored from DriverDBv2 predictions; its gene
    interaction network intersected with TCGA genes is also the source of the Gene_Net
    hub network used upstream
- name: DriverDBv2
  role: Database from which the 11 comparison tools' driver gene predictions (DrGap,
    OncoDriveFM, MutSigCV, DriverML, PNC, DawnRank, SCS, DriverNet, ActiveDriver,
    iPAC, MSEA) are retrieved for scoring
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad071
    title: 'WMDS.net: a network control framework for identifying key players in transcriptome
      programs'
  source_content_hash: sha256:88975283146920e1eb80512d7d2c1fb5c08191ea687555e351cfc6a691651d0a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chaofen123/WMDS.net
  tools:
  - DawnRank
  - DriverDBv2
  - Matlab R2020b
  - WMDS.net
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# cancer-driver-gene-benchmark-scoring

## Summary

Scores cancer driver gene prediction lists against curated benchmark gene sets — 616-gene CGC (COSMIC), 711-gene NCG and the 501-gene PMH set — computing precision, recall and the precision-weighted F_beta measure (beta^2 = 0.2) per tool and TCGA cancer type, in order to rank predictors such as WMDS.net, WMDS.netP and 11 comparison tools.

## When to use

Use when ranking driver-gene prediction lists (your own or tools like PNC, DawnRank, MutSigCV) against curated CGC, NCG or PMH benchmark gene sets with precision, recall and a precision-weighted F0.2 score across TCGA cancer types.

## When NOT to use

- Input is still raw expression matrices (e.g. TCGA_BLCAtumor vs TCGA_BLCAnormal) rather than gene-symbol driver lists — the upstream WMDS.net integer-programming prediction step must run first.
- The cancer type or study has no meaningful overlap with the CGC, NCG or PMH curated driver sets, so TP/FP/FN counts and F0.2 values would be uninformative.
- The goal is to run driver discovery itself (constructing Gene_Net / differential co-expression networks and solving the integer program) rather than to score already-produced gene lists.

## Inputs

- Driver gene prediction lists per tool per TCGA cancer type (WMDS.net and WMDS.netP from the Matlab R2020b intlinprog step; 11 comparison tools from DriverDBv2)
- CGC benchmark gene list (616 genes, downloaded from COSMIC, https://cancer.sanger.ac.uk/cosmic)
- NCG benchmark gene list (711 genes)
- PMH benchmark gene set (501 genes merged from Pan-cancer drivers, Mut-drivers and Hiconf drivers)
- TCGA somatic SNV data for the 14 cancer types from GDC (https://portal.gdc.cancer.gov), for the optional low/high mutation-frequency classification

## Outputs

- Table of F0.2 scores with columns tool, cancer type, benchmark (CGC/NCG/PMH) — the data behind Figures 3B-3D
- Per-benchmark ranking of the 13 scored predictors (WMDS.net, WMDS.netP plus the 11 DriverDBv2 tools)
- Robustness table recomputed at beta^2 = 1 (F1) showing unchanged relative tool rankings
- Optional per-cancer-type fractions of CGC-overlapping predicted drivers in the low (<0.05) versus high (>0.05) SNV mutation-frequency classes, with flagged low-frequency drivers such as ERBB3 in LUAD

## How to apply

Load the driver gene lists for the 14 TCGA cancer types (BLCA, BRCA, COAD, HNSC, KICH, KIRC, KIRP, LIHC, LUAD, LUSC, PRAD, STAD, THCA, UCEC): WMDS.net/WMDS.netP lists from the upstream Matlab R2020b 'intlinprog' binary integer-programming step, plus the 11 comparison tools (DrGap, OncoDriveFM, MutSigCV, DriverML, PNC, DawnRank, SCS, DriverNet, ActiveDriver, iPAC, MSEA) retrieved from DriverDBv2. Load the three benchmark sets: 616 CGC genes from COSMIC (https://cancer.sanger.ac.uk/cosmic), 711 NCG genes, and the 501-gene PMH set merged from Pan-cancer drivers, Mut-drivers and Hiconf drivers. For each tool x cancer type x benchmark, count TP/FP/FN and compute precision = TP/(TP+FP), recall = TP/(TP+FN) and F_beta = (1+beta^2)*P*R/(beta^2*P+R) with beta^2 = 0.2, a precision-weighted setting that penalises false-positive driver calls. Rank all tools per benchmark and assemble the table of tool, cancer type, benchmark and F0.2 score behind Figures 3B-3D. Robustness-check by recomputing at beta^2 = 1 (F1) and confirming relative rankings are unchanged. Optionally (per the low-frequency analysis), intersect predicted drivers with the CGC list per cancer type and classify SNV mutation frequencies from GDC TCGA data as low (<0.05) or high (>0.05), expecting most CGC-overlapping predicted drivers to be rarely mutated (e.g. ERBB3 as a low-frequency LUAD driver).

## Related tools

- **WMDS.net** (Predictor under evaluation: its (and WMDS.netP's) driver gene lists for 14 TCGA cancer types are scored with F0.2 against CGC, NCG and PMH benchmarks) — https://github.com/chaofen123/WMDS.net
- **Matlab R2020b** (Implementation environment: runs the upstream 'intlinprog' binary integer programming that produces the WMDS.net driver lists, and computes precision/recall/F_beta for the benchmark table)
- **DawnRank** (One of the 11 comparison tools scored from DriverDBv2 predictions; its gene interaction network intersected with TCGA genes is also the source of the Gene_Net hub network used upstream)
- **DriverDBv2** (Database from which the 11 comparison tools' driver gene predictions (DrGap, OncoDriveFM, MutSigCV, DriverML, PNC, DawnRank, SCS, DriverNet, ActiveDriver, iPAC, MSEA) are retrieved for scoring)

## Examples

```
P = TP/(TP+FP); R = TP/(TP+FN); beta2 = 0.2; F02 = (1+beta2)*P*R/(beta2*P+R); % Matlab R2020b, F0.2 per tool x cancer type x benchmark
```

## Evaluation signals

- The F0.2 table reproduces the Figure 3B-3D ordering: WMDS.netP and WMDS.net hold the two highest F0.2 values on the CGC and NCG benchmarks, about 30% above third-ranked PNC.
- On the PMH benchmark, WMDS.netP and WMDS.net rank within the top three (together with ActiveDriver) across the 14 TCGA datasets.
- Recomputing with beta^2 = 1 (F1) leaves the relative tool rankings unchanged on all three benchmarks.
- Schema/consistency check: one row per tool x cancer type x benchmark; TP is bounded by both the prediction list size and the benchmark size (616 CGC, 711 NCG, 501 PMH genes), and precision = TP/(TP+FP), recall = TP/(TP+FN) hold exactly.
- Downstream validity: most CGC-overlapping predicted drivers per cancer type fall in the low-frequency class (SNV mutation frequency < 0.05), and ERBB3 is present as a low-frequency LUAD driver consistent with COSMIC records.

## Limitations

- Ranking is benchmark-dependent: WMDS.netP/WMDS.net rank top-two on CGC and NCG but only top-three (with ActiveDriver) on PMH, so conclusions change with the chosen benchmark list.
- Comparison coverage is limited to the 11 tools whose predictions are available in DriverDBv2 and to the 14 TCGA datasets analyzed in the study.
- Benchmark lists are fixed snapshots (616 CGC genes from COSMIC, 711 NCG, 501 PMH); later COSMIC/NCG updates change TP/FP/FN counts and F0.2 values, and the 0.05 mutation-frequency cutoff makes genes near the threshold unstable.

## Evidence

- [abstract] WMDS.net is powerful among various cancer datasets and outperformed the other top-tier tools with a better balance between precision and recall: "WMDS.net is powerful among various cancer datasets and outperformed the other top-tier tools with a better balance between precision and recall"
- [intro] To confirm its validity, we applied WMDS.net to the discovery of cancer driver genes in RNA-seq datasets from The Cancer Genome Atlas.: "To confirm its validity, we applied WMDS.net to the discovery of cancer driver genes in RNA-seq datasets from The Cancer Genome Atlas."
- [readme] solve the binary integer-programming problem using function “intlinprog” which is available in the Optimization ToolBox of MatLab version R2020b: "solve the binary integer-programming problem using function “intlinprog” which is available in the Optimization ToolBox of MatLab version R2020b"
- [readme] Gene_Net is the hub network of intersection of gene interaction network constructed in Dawnrank[1] and the genes in TCGA.: "Gene_Net is the hub network of intersection of gene interaction network constructed in Dawnrank[1] and the genes in TCGA."
- [other] compute precision = TP/(TP+FP), recall = TP/(TP+FN) and the F-measure F_beta = (1+beta^2)*P*R/(beta^2*P+R) with beta^2 = 0.2 against each of the CGC, NCG and PMH benchmarks: [evidence span withheld — anchor status: not_found]
- [other] On the CGC benchmark, WMDS.netP and WMDS.net achieved the two highest F0.2-measure values, about 30% higher than PNC which ranked third: [evidence span withheld — anchor status: not_found]
