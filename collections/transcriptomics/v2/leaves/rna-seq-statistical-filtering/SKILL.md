---
name: rna-seq-statistical-filtering
description: Use when a counts matrix plus a two-group factor (e.g. hypoxia-high vs
  hypoxia-low TCGA-LUAD samples) must be tested for differentially expressed genes
  and filtered at explicit p/logFC/FDR cut-offs.
when_to_use_negative:
- Input is already a TPM/expression matrix normalized elsewhere and you only need
  correlation or survival analysis rather than count-based DE testing
- The goal is differential analysis of GSVA scores or mutation data rather than gene-level
  differential expression
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_3308
tools:
- name: edgeR
  role: Primary package used to detect differentially expressed genes with TMM normalization
  repo: https://github.com/Dulab2020/ARMT
- name: limma
  role: Secondary DE package used to verify/confirm edgeR DEG results
  repo: https://github.com/Dulab2020/ARMT
- name: ARMT
  role: Shiny GUI that runs the differential analysis and provides p-value/logFC/FDR
    result screening
  repo: https://github.com/Dulab2020/ARMT
- name: R
  role: Runtime environment (R >= 4.0.2) hosting the Bioconductor DE packages
  repo: https://github.com/Dulab2020/ARMT
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.009
    title: 'ARMT: An automatic RNA-seq data mining tool based on comprehensive and
      integrative analysis in cancer research'
  source_content_hash: sha256:016d45a62fd394de140957d8402561c03c6f76620b8286c339911a2183f20c7b
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
  - R
  - edgeR
  - limma
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# rna-seq-statistical-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Detect differentially expressed genes from RNA-seq counts using edgeR (TMM normalization), with limma as a cross-check, and filter results by p-value, logFC and FDR thresholds to extract DEG and up-regulated gene lists in ARMT-based workflows.

## When to use

Use when a counts matrix plus a two-group factor (e.g. hypoxia-high vs hypoxia-low TCGA-LUAD samples) must be tested for differentially expressed genes and filtered at explicit p/logFC/FDR cut-offs.

## When NOT to use

- Input is already a TPM/expression matrix normalized elsewhere and you only need correlation or survival analysis rather than count-based DE testing
- The goal is differential analysis of GSVA scores or mutation data rather than gene-level differential expression

## Inputs

- Raw RNA-seq counts matrix (.csv; rows = Ensembl gene IDs, columns = sample IDs)
- Group/difference factor from integrated data (e.g. hypoxia GSVA score groups, tumor vs normal)
- Optional second counts matrix or method run (limma) for cross-method verification

## Outputs

- Filtered DEG table (e.g. deg_results.csv) with p-value, logFC and FDR columns
- Up-regulated gene list extracted under the stringent criterion
- Volcano plot visualization of DEG results

## How to apply

Normalize the raw counts matrix to TPM and transform to log2(TPM + 1) following the ARMT convention, then group samples by the difference factor (numeric factors are split into high/low with an adjustable cut-off t, low group as control). Run differential analysis with edgeR, which ARMT employs 'for high efficiency', and optionally repeat with limma to confirm consistency since both are generally acknowledged R packages for RNA-seq DE analysis. Screen results by p-value, logFC and FDR(adj.p) — the source workflow uses p < 0.05, -log10(FDR) > 2, logFC > 1 for DEGs and a stringent p < 0.05, -log10(FDR) > 20, logFC > 1 for up-regulated genes. Write the filtered table (e.g. deg_results.csv) and compare DEG/up-regulated gene counts against expected values (e.g. 6684 DEGs, 1174 up-regulated) to judge correctness.

## Related tools

- **edgeR** (Primary package used to detect differentially expressed genes with TMM normalization) — https://github.com/Dulab2020/ARMT
- **limma** (Secondary DE package used to verify/confirm edgeR DEG results) — https://github.com/Dulab2020/ARMT
- **ARMT** (Shiny GUI that runs the differential analysis and provides p-value/logFC/FDR result screening) — https://github.com/Dulab2020/ARMT
- **R** (Runtime environment (R >= 4.0.2) hosting the Bioconductor DE packages) — https://github.com/Dulab2020/ARMT

## Examples

```
if (!requireNamespace("devtools", quietly = TRUE)) install.packages("devtools"); devtools::install_github("Dulab2020/ARMT")
```

## Evaluation signals

- Filtered DEG counts match reported values (6684 DEGs; 1174 up-regulated genes at the stringent threshold)
- DEG lists from edgeR and limma are consistent, as expected for acknowledged RNA-seq DE packages
- Every retained gene satisfies the filter thresholds: p < 0.05, -log10(FDR) > 2, |logFC| > 1 (stringent: -log10(FDR) > 20)
- Counts matrix was TPM-normalized and log2(TPM + 1) transformed before analysis, per the ARMT convention
- Output table is written in the expected schema (e.g. deg_results.csv with p, FDR, logFC columns)

## Limitations

- Different DE tools based on different statistical principles may lead to different results, so cross-tool (edgeR vs limma) verification is advised
- No software version numbers for edgeR/limma are stated, so exact reproducibility is not guaranteed
- ARMT groups numeric difference factors into high/low with an adjustable cut-off t (upper and lower t*100%), and the low part is treated as control — cut-off choice affects results

## Evidence
