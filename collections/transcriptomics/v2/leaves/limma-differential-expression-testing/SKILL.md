---
name: limma-differential-expression-testing
description: Use when the analysis target is a GSVA score matrix (samples x genesets)
  and the goal is testing differential gene-set activity between two sample groups
  in RNA-seq/cancer data.
when_to_use_negative:
- The input is a raw counts matrix for gene-level differential expression (DEG) —
  ARMT uses edgeR for that case, not limma-on-GSVA.
- The difference factor is a non-numeric variable — instead select two explicit values
  to group samples into experiment and control groups.
- No gene-set-level (GSVA) scores exist and the question concerns single-gene expression
  or mutation differences rather than pathway activity.
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: limma
  role: Performs the differential testing on continuous GSVA scores between high/low
    sample groups
  repo: https://github.com/Dulab2020/ARMT
- name: GSVA
  role: Converts the log2(TPM+1) expression matrix into a per-sample gene set (GSVA)
    score matrix that limma then tests
  repo: https://github.com/Dulab2020/ARMT
- name: ARMT
  role: Shiny-based GUI that orchestrates normalization, GSVA computation, high/low
    grouping with cut-off t, and limma-based differential analysis with logFC/p-value/FDR
    filtering
  repo: https://github.com/Dulab2020/ARMT
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.009
    title: 'ARMT: An automatic RNA-seq data mining tool based on comprehensive and
      integrative analysis in cancer research'
  source_content_hash: sha256:10e0f1f23ae8eb607aaa0358f7da137bd5e061b7f5c35981b2101dab652e6b79
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
  - limma
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# limma-differential-expression-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Differential testing of gene-set-level GSVA scores with limma, as implemented in ARMT: samples are grouped high/low by a GSVA score and limma linear models test score differences between groups, with results filtered by logFC, p-value, and FDR.

## When to use

Use when the analysis target is a GSVA score matrix (samples x genesets) and the goal is testing differential gene-set activity between two sample groups in RNA-seq/cancer data.

## When NOT to use

- The input is a raw counts matrix for gene-level differential expression (DEG) — ARMT uses edgeR for that case, not limma-on-GSVA.
- The difference factor is a non-numeric variable — instead select two explicit values to group samples into experiment and control groups.
- No gene-set-level (GSVA) scores exist and the question concerns single-gene expression or mutation differences rather than pathway activity.

## Inputs

- Counts matrix as .csv (rows = gene Ensembl IDs, columns = sample IDs)
- Gene set file (.csv with geneset and gene columns, convertible to .gmt), e.g. hypoxia and glycolysis signatures
- GSVA score matrix as .csv (rows = sample IDs, columns = geneset names) produced by the Normalization&GSVA page
- Integrated data table with a numeric Group Column / difference factor (the selected GSVA score)

## Outputs

- Differential GSVA results table (.csv) with logFC, p-value, and FDR (adj.p) per geneset after threshold filtering
- Visualization of the differential GSVA score result in the ARMT main panel

## How to apply

First obtain GSVA scores by normalizing the Ensembl-ID counts matrix to TPM, transforming to log2(TPM + 1) with Ensembl-to-symbol conversion (ARMT Normalization&GSVA module), and running GSVA with the gene sets from a .gmt file to yield a samples-by-genesets score matrix saved as CSV. Then select a numeric difference factor (a GSVA score column) in the integrated data; ARMT splits samples into high and low groups using the adjustable cut-off t (upper and lower t*100%, e.g. top/bottom 30% in the LUAD hypoxia case), treating the low group as control. Run the differential analysis with limma rather than edgeR because GSVA scores are decimal/continuous values, which suits limma's linear-model framework rather than edgeR's count model. Finally screen the output by adjustable logFC, p-value, and FDR (adj.p) thresholds and export the differential GSVA results table as CSV.

## Related tools

- **limma** (Performs the differential testing on continuous GSVA scores between high/low sample groups) — https://github.com/Dulab2020/ARMT
- **GSVA** (Converts the log2(TPM+1) expression matrix into a per-sample gene set (GSVA) score matrix that limma then tests) — https://github.com/Dulab2020/ARMT
- **ARMT** (Shiny-based GUI that orchestrates normalization, GSVA computation, high/low grouping with cut-off t, and limma-based differential analysis with logFC/p-value/FDR filtering) — https://github.com/Dulab2020/ARMT

## Examples

```
devtools::install_github("Dulab2020/ARMT"); ARMT::run_app()
```

## Evaluation signals

- The GSVA score matrix has rows = sample IDs and columns = geneset names and is saved as CSV before differential analysis.
- Sample grouping yields high/low sets sized per the cut-off t (e.g. top and bottom 30% for t=0.3), with the low group treated as control.
- The differential GSVA results table contains logFC, p-value, and FDR columns and passes the chosen filtering thresholds.
- Validation benchmark: in TCGA-LUAD, the glycolysis GSVA score difference between hypoxia-high and hypoxia-low groups is reproduced as highly significant (paper reports adj.p = 2.45*10^-128).
- limma (not edgeR) is confirmed as the engine for the GSVA-score differential step, consistent with GSVA scores being decimal values.

## Limitations

- No software version numbers for the underlying R packages are stated in the article or README.
- The cut-off t for numeric grouping is user-adjustable, so results depend on the chosen upper/lower percentage of samples.
- Different differential analysis tools based on different statistical principles may lead to different results (article intro caveat).

## Evidence

- [other] run differential analysis of the GSVA scores with limma (used instead of edgeR because GSVA scores are decimal values).: [evidence span withheld — anchor status: not_found]
- [other] Filter the differential results by the adjustable thresholds (logFC, p-value, FDR) and save the differential GSVA results table as a CSV file.: [evidence span withheld — anchor status: not_found]
