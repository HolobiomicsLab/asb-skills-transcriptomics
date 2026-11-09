---
name: gsva-score-differential-analysis-limma
description: Use when RNA-seq gene-set (pathway) activity must be compared between
  sample groups — e.g. glycolysis GSVA score between hypoxia-high and hypoxia-low
  TCGA-LUAD samples — and the test variable is continuous decimal scores.
when_to_use_negative:
- Testing differential expression on the raw counts matrix — ARMT uses edgeR for DEGs,
  not limma-on-GSVA-scores
- The input is already a per-sample gene-set score table produced elsewhere and no
  grouping/cut-off step is needed — go straight to statistical testing
- The question is prognostic (survival/Cox) rather than group comparison — use ARMT's
  survival analysis and Cox proportional hazards regression module instead
edam_operation: http://edamontology.org/operation_3225
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ARMT
  role: Shiny GUI that performs TPM normalization, GSVA, high/low sample grouping
    at cut-off t, and limma-based differential analysis of GSVA scores
  repo: https://github.com/Dulab2020/ARMT
- name: GSVA
  role: Computes per-sample gene set variation analysis scores from the log2(TPM+1)
    matrix for the supplied .gmt gene sets
- name: limma
  role: Statistical test for differential GSVA scores between high/low groups, chosen
    over edgeR because GSVA scores are decimal values
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.009
    title: 'ARMT: An automatic RNA-seq data mining tool based on comprehensive and
      integrative analysis in cancer research'
  source_content_hash: sha256:13ee68f81f2afa83d75a36c463fd97d6d26761306f4ddf93028401face0e8b9a
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
  edam_operation: http://edamontology.org/operation_3225
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# gsva-score-differential-analysis-limma

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Differentially tests gene-set activity between sample groups by first converting gene sets (e.g. 22-gene glycolysis, 14-gene hypoxia signatures) to per-sample GSVA scores, then applying limma on those scores in ARMT.

## When to use

Use when RNA-seq gene-set (pathway) activity must be compared between sample groups — e.g. glycolysis GSVA score between hypoxia-high and hypoxia-low TCGA-LUAD samples — and the test variable is continuous decimal scores.

## When NOT to use

- Testing differential expression on the raw counts matrix — ARMT uses edgeR for DEGs, not limma-on-GSVA-scores
- The input is already a per-sample gene-set score table produced elsewhere and no grouping/cut-off step is needed — go straight to statistical testing
- The question is prognostic (survival/Cox) rather than group comparison — use ARMT's survival analysis and Cox proportional hazards regression module instead

## Inputs

- Raw counts matrix as .csv (rows = Ensembl gene IDs, columns = sample IDs)
- Gene set file: .csv with geneset and gene columns, convertible to a .gmt (e.g. 22-gene glycolysis and 14-gene hypoxia signatures)
- GSVA score matrix as .csv (rows = sample IDs, columns = geneset names) from the Normalization&GSVA page
- Integrated data table with a Group Column containing the numeric grouping factor

## Outputs

- GSVA score matrix .csv (sample ID rows, geneset name columns)
- Differential GSVA analysis result table (.csv) with logFC, p-value and FDR (adj.p) columns, filtered by adjustable thresholds

## How to apply

Normalize the counts matrix (rows = Ensembl IDs, columns = samples) to TPM and transform to log2(TPM + 1), converting Ensembl IDs to gene symbols as ARMT's Normalization&GSVA module does. Compute GSVA scores for the gene sets supplied as a .gmt (built from a two-column geneset/gene .csv) and export the score matrix as a .csv with sample IDs as rows and geneset names as columns. In the Integration&Analysis page, choose the numeric GSVA-score factor (e.g. hypoxia score); ARMT splits samples into high and low groups at an adjustable cut-off t (upper and lower t*100%, e.g. top and bottom 30%), treating the low group as control. Run the differential analysis with limma — edgeR is not applicable because GSVA scores are decimal values while edgeR requires integer counts — then filter results by adjustable logFC, p-value and FDR (adj.p) thresholds and save the table as CSV. Validate by reproducing the reported case: glycolysis GSVA score differential between hypoxia-high and -low LUAD groups with adjusted p = 2.45e-128.

## Related tools

- **ARMT** (Shiny GUI that performs TPM normalization, GSVA, high/low sample grouping at cut-off t, and limma-based differential analysis of GSVA scores) — https://github.com/Dulab2020/ARMT
- **GSVA** (Computes per-sample gene set variation analysis scores from the log2(TPM+1) matrix for the supplied .gmt gene sets)
- **limma** (Statistical test for differential GSVA scores between high/low groups, chosen over edgeR because GSVA scores are decimal values)

## Examples

```
if (!requireNamespace("devtools", quietly = TRUE)) install.packages("devtools"); devtools::install_github("Dulab2020/ARMT"); ARMT::run_app()
```

## Evaluation signals

- Differential analysis of glycolysis GSVA score between hypoxia-high and hypoxia-low LUAD groups reproduces the reported adjusted p-value of 2.45e-128
- Grouping uses top and bottom t*100% of samples by the numeric GSVA score (e.g. 30%), with the low group defined as the control group
- Exported GSVA score matrix has sample IDs as rows and geneset names as columns and can be reloaded through a .csv file in the Integration&Analysis page
- Differential result table contains filterable logFC, p-value and FDR (adj.p) columns and is saved as CSV
- Expression matrix used for GSVA is log2(TPM + 1) with gene symbols, not raw Ensembl-ID counts

## Limitations

- edgeR cannot be used on GSVA scores because they are decimal values incompatible with edgeR's integer input requirement, so limma is mandatory for this step
- No software version numbers for ARMT, GSVA or limma are stated in the article or README
- When the difference factor is non-numeric, the high/low cut-off logic does not apply — two values must be selected for experiment vs control groups

## Evidence
