---
name: benchmark-gene-list-mapping
description: Use when you have driver gene prediction lists for multiple tools (e.g.
  from DriverDBv2) and need to rank them against curated benchmark sets (CGC 616 genes,
  NCG 711 genes, PMH 501 genes) with an F-measure.
when_to_use_negative:
- The candidate gene list is not a driver-gene prediction set but raw expression or
  co-expression data — this skill only scores already-predicted gene lists against
  benchmarks
- Recall is the priority of the evaluation (beta > 1); the skill's headline metric
  deliberately weights precision (beta^2 = 0.2), so F0.2 rankings would misrepresent
  recall-oriented goals
- No curated benchmark list (CGC/NCG/PMH or equivalent) exists for the cancer type,
  since TP/FP/FN cannot be defined without a reference gene set
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0085
tools:
- name: WMDS.net
  role: Method under evaluation whose predicted driver nodes are scored against the
    CGC/NCG/PMH benchmarks alongside 11 competitor tools
  repo: https://github.com/chaofen123/WMDS.net
- name: DawnRank
  role: Source of the gene interaction network whose hub intersection with TCGA genes
    defines Gene_Net, the network backbone of the evaluated predictions
- name: Matlab R2020b
  role: Environment producing the evaluated WMDS.net driver lists via the intlinprog
    binary integer-programming function
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad071
    title: 'WMDS.net: a network control framework for identifying key players in transcriptome
      programs'
  source_content_hash: sha256:9f886de640e02ddec0954f1e6010e8da42b1a3a68c4872fb043471a41fb4f6f6
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
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# benchmark-gene-list-mapping

## Summary

Scores cancer driver gene predictions against curated benchmark gene lists (CGC, NCG, PMH) using precision/recall and the precision-weighted F0.2 measure, then ranks competing tools per benchmark. It grounds tool-performance claims (e.g. WMDS.net vs 11 DriverDBv2 tools) in a reproducible benchmark table.

## When to use

Use when you have driver gene prediction lists for multiple tools (e.g. from DriverDBv2) and need to rank them against curated benchmark sets (CGC 616 genes, NCG 711 genes, PMH 501 genes) with an F-measure.

## When NOT to use

- The candidate gene list is not a driver-gene prediction set but raw expression or co-expression data — this skill only scores already-predicted gene lists against benchmarks
- Recall is the priority of the evaluation (beta > 1); the skill's headline metric deliberately weights precision (beta^2 = 0.2), so F0.2 rankings would misrepresent recall-oriented goals
- No curated benchmark list (CGC/NCG/PMH or equivalent) exists for the cancer type, since TP/FP/FN cannot be defined without a reference gene set

## Inputs

- Predicted driver gene lists per tool and per TCGA cancer type (e.g. the 11 comparison tools DrGap, OncoDriveFM, MutSigCV, DriverML, PNC, DawnRank, SCS, DriverNet, ActiveDriver, iPAC, MSEA retrieved from DriverDBv2, plus WMDS.net/WMDS.netP outputs from the Matlab R2020b intlinprog step)
- Benchmark gene lists: 616-gene CGC (COSMIC), 711-gene NCG, 501-gene PMH (merged Pan-cancer drivers + Mut-drivers + Hiconf drivers)
- Cancer-type labels for the 14 TCGA datasets

## Outputs

- Ranked table of F0.2 scores with columns tool, cancer type, benchmark, F0.2 score (assembled for Figures 3B-3D)
- Per-tool precision and recall values against each benchmark
- F1-measure sensitivity check confirming unchanged relative rankings

## How to apply

Load each tool's predicted driver gene list for each TCGA cancer type and the benchmark lists (616 genes from CGC/COSMIC, 711 genes from NCG, and a 501-gene PMH set merged from Pan-cancer drivers, Mut-drivers and Hiconf drivers). For every tool x cancer type x benchmark, compute TP/FP/FN against the benchmark set, then precision = TP/(TP+FP), recall = TP/(TP+FN), and the F-measure F_beta = (1+beta^2)*P*R/(beta^2*P+R) with beta^2 = 0.2 to weight precision over recall. Rank all tools per benchmark by F0.2 and assemble a table (tool, cancer type, benchmark, F0.2 score). Recompute with beta^2 = 1 (F1) as a sensitivity check that relative rankings are unchanged before accepting the result.

## Related tools

- **WMDS.net** (Method under evaluation whose predicted driver nodes are scored against the CGC/NCG/PMH benchmarks alongside 11 competitor tools) — https://github.com/chaofen123/WMDS.net
- **DawnRank** (Source of the gene interaction network whose hub intersection with TCGA genes defines Gene_Net, the network backbone of the evaluated predictions)
- **Matlab R2020b** (Environment producing the evaluated WMDS.net driver lists via the intlinprog binary integer-programming function)

## Evaluation signals

- WMDS.netP and WMDS.net hold the two highest F0.2 values on the CGC benchmark, ~30% above third-ranked PNC
- WMDS.netP and WMDS.net rank top-two on the NCG benchmark and within the top three (with ActiveDriver) on the PMH benchmark across the 14 TCGA datasets
- Benchmark list sizes match the article: 616 (CGC), 711 (NCG), 501 (PMH)
- F0.2 values reproduce the ordering shown in Figures 3B-3D when tabulated per tool, cancer type, and benchmark
- Recomputing with beta^2 = 1 (F1) leaves the relative tool rankings unchanged

## Limitations

- Benchmark scoring is limited to the three reference lists used (CGC, NCG, PMH); results on PMH are shared with ActiveDriver rather than dominated, so claims of superiority must be qualified per benchmark
- Predictions for the 11 comparison tools depend on DriverDBv2 availability; missing tool x cancer-type cells break the complete ranking table
- F0.2 is precision-dominant (beta^2 = 0.2), so the ranking emphasizes precision over recall by design

## Evidence

- [abstract] outperformed top-tier tools claim: "WMDS.net is powerful among various cancer datasets and outperformed the other top-tier tools with a better balance between precision and recall"
- [intro] TCGA application scope: "we applied WMDS.net to the discovery of cancer driver genes in RNA-seq datasets from The Cancer Genome Atlas."
- [readme] Gene_Net construction: "Gene_Net is the hub network of intersection of gene interaction network constructed in Dawnrank[1] and the genes in TCGA."
- [readme] input format for driver prediction: "The inputs to the main program are the expression matrices of the two conditions (which are TCGA_BLCAtumor and TCGA_BLCAnormal in the tested data)"
