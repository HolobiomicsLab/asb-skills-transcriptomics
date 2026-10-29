---
name: multiple-testing-fdr-correction
description: Use when a per-gene test over hundreds/thousands of genes (mutation enrichment,
  DEG, or GSVA score difference) has produced raw p-values and you must report a defensible
  significant set, e.g. filtering mutation results with p < 0.05 and FDR < 0.01.
when_to_use_negative:
- You have a single pre-specified hypothesis test on one gene/geneset — no multiplicity
  correction is needed and FDR would only reduce power
- Input is already an FDR-adjusted table (adj.p column present) — re-correcting adjusted
  values would be invalid
- You need family-wise error control for a strict confirmatory claim rather than exploratory
  discovery-rate control
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ARMT
  role: Shiny GUI that performs the per-gene differential tests (mutation via maftools,
    DEG via edgeR, GSVA score) and exposes p-value, logFC and FDR(adj.p) screening
    parameters
  repo: https://github.com/Dulab2020/ARMT
- name: maftools
  role: Backend computing per-gene mutation difference p-values between sample groups
    from the .maf file
  repo: https://github.com/Dulab2020/ARMT
- name: edgeR
  role: Package ARMT employs to detect differentially expressed genes whose p-values
    are subsequently FDR-screened
  repo: https://github.com/Dulab2020/ARMT
- name: GSVA
  role: Produces the gene-set score matrix whose group differences are tested and
    FDR-filtered
  repo: https://github.com/Dulab2020/ARMT
- name: R
  role: Runtime environment (>=4.0.2) in which ARMT and all correction workflows execute
  repo: https://github.com/Dulab2020/ARMT
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.009
    title: 'ARMT: An automatic RNA-seq data mining tool based on comprehensive and
      integrative analysis in cancer research'
  source_content_hash: sha256:9c9aea785e6c43f0d518358febe1e9ff2fce0531bf5298202e5ca521d787614d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Dulab2020/ARMT
  repo_url: https://github.com/Dulab2020/ARMT
  tools:
  - ARMT
  - GSVA
  - R
  - edgeR
  - maftools
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# multiple-testing-fdr-correction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies false-discovery-rate (FDR) adjustment to per-gene p-values from ARMT differential analyses (gene mutation via maftools, differential expression via edgeR, GSVA score differences) so that significance calls control the expected proportion of false positives. Essential when screening thousands of genes simultaneously, e.g. identifying the five SNV-significantly mutated genes (TP53, CSMD3, TTN, CD163L1, OR4C15) in hypoxia-high TCGA-LUAD.

## When to use

Use when a per-gene test over hundreds/thousands of genes (mutation enrichment, DEG, or GSVA score difference) has produced raw p-values and you must report a defensible significant set, e.g. filtering mutation results with p < 0.05 and FDR < 0.01.

## When NOT to use

- You have a single pre-specified hypothesis test on one gene/geneset — no multiplicity correction is needed and FDR would only reduce power
- Input is already an FDR-adjusted table (adj.p column present) — re-correcting adjusted values would be invalid
- You need family-wise error control for a strict confirmatory claim rather than exploratory discovery-rate control

## Inputs

- Per-gene raw p-values from ARMT differential mutation analysis (.maf input via maftools)
- Per-gene raw p-values from ARMT differential expression analysis (edgeR on counts matrix .csv)
- Per-geneset p-values from differential GSVA score analysis (GSVA score matrix .csv)
- Grouping factor column from integrated ARMT data (e.g. hypoxia-high vs hypoxia-low)

## Outputs

- Filtered gene-level significance table (gene, p-value, FDR) e.g. hypoxia_high_mutated_genes.csv
- Forest plot of differential mutation genes (Fig. 3 B equivalent)
- Screened DEG list (p-value, logFC, FDR filters) for downstream GO/KEGG enrichment

## How to apply

Run the underlying per-gene test first: for differential mutation analysis, load the TCGA-LUAD .maf file in ARMT (maftools-backed 'Mutant mapping'/'differential analysis of gene mutation' module) and compute per-gene mutation p-values between hypoxia-high and hypoxia-low groups; for DEG, edgeR computes the p-values inside ARMT; for GSVA score differences, enter the GSVA score .csv produced by the 'Normalization&GSVA' page. After analysis, adjust p-values for multiple testing (ARMT exposes this as the FDR / adj.p column) and screen results by p-value, logFC and FDR(adj.p) as adjustable cutoffs in the GUI. For the hypoxia-high LUAD case study, apply the thresholds p < 0.05 and FDR < 0.01 and keep only genes passing both, which recovers exactly five SNV genes: TP53, CSMD3, TTN, CD163L1, OR4C15. Export the filtered gene-level table (gene, p-value, FDR) and visualize the differential mutation genes as a forest plot (Fig. 3 B equivalent). Rationale: different tools based on different statistical principles can yield different results, so the reported significant set should always be defined by an explicit, dual p/FDR cutoff.

## Related tools

- **ARMT** (Shiny GUI that performs the per-gene differential tests (mutation via maftools, DEG via edgeR, GSVA score) and exposes p-value, logFC and FDR(adj.p) screening parameters) — https://github.com/Dulab2020/ARMT
- **maftools** (Backend computing per-gene mutation difference p-values between sample groups from the .maf file) — https://github.com/Dulab2020/ARMT
- **edgeR** (Package ARMT employs to detect differentially expressed genes whose p-values are subsequently FDR-screened) — https://github.com/Dulab2020/ARMT
- **GSVA** (Produces the gene-set score matrix whose group differences are tested and FDR-filtered) — https://github.com/Dulab2020/ARMT
- **R** (Runtime environment (>=4.0.2) in which ARMT and all correction workflows execute) — https://github.com/Dulab2020/ARMT

## Examples

```
devtools::install_github("Dulab2020/ARMT"); ARMT::run_app()  # then in Integration&Analysis > Differential analysis (gene mutation), set p<0.05 and FDR<0.01
```

## Evaluation signals

- Validation table for hypoxia-high TCGA-LUAD must contain exactly the five reported SNV genes (TP53, CSMD3, TTN, CD163L1, OR4C15), each with p < 0.05 and FDR < 0.01
- FDR column is monotonically non-decreasing when rows are sorted by raw p-value, and FDR >= p for every gene
- Genes passing the dual cutoff form a strict subset of all tested genes, and the count of significant genes is far smaller than the tested universe (5 significant SNV genes in the case study)
- Forest plot of differential mutation genes renders only the filtered set and matches the paper's Fig. 3 B equivalent
- Re-running the analysis with the same grouping (top/bottom hypoxia GSVA score strata) reproduces the same significant-gene list

## Limitations

- Different differential-analysis tools based on different statistical principles may lead to different results, so the significant set is pipeline-dependent
- No software version numbers for the tools are stated, hindering exact reproducibility of p-value/FDR computations
- Continuous grouping factors are dichotomized by a top/bottom fraction cutoff (adjustable upper and lower t*100%, with the low part as control), so the tested groups and hence adjusted p-values depend on that parameter
- No ARMT version number is stated for the released tool

## Evidence
