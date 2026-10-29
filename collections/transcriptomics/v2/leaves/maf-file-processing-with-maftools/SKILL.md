---
name: maf-file-processing-with-maftools
description: Use when mutation information is provided as a standard .maf file (e.g.,
  TCGA LUAD) and you need per-gene mutation significance between groups (p-value,
  FDR) or mutation visualization such as forest plots and oncoplots.
when_to_use_negative:
- Mutation data is not in standard .maf format — ARMT explicitly requires 'standard
  .maf file like the mutation annotation format file in TCGA'
- The goal is differential gene expression or GSVA score analysis rather than somatic
  mutation analysis (use the differential analysis of gene expression or GSVA instead)
- Samples cannot be integrated across clinical, GSVA, TPM and mutation data through
  common samples
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: ARMT
  role: Shiny-based GUI wrapper providing Mutant mapping (Maftools Summary and Self-defined
    modes) and differential gene mutation analysis on integrated .maf data
  repo: https://github.com/Dulab2020/ARMT
- name: maftools
  role: Underlying Bioconductor package with which ARMT visualizes gene mutations
    in .maf files (summary, oncoplot, pathway enrichment, VAF)
- name: GSVA
  role: Provides hypoxia gene set scores used to stratify samples into hypoxia-high/low
    groups before differential mutation testing
- name: R
  role: Runtime environment (R >= 4.0.2) in which ARMT and its Bioconductor dependencies
    are installed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.009
    title: 'ARMT: An automatic RNA-seq data mining tool based on comprehensive and
      integrative analysis in cancer research'
  source_content_hash: sha256:d4560e0a91fb7bc56f19a1fdb13ee51816765d3bcc20c7f9370948db5c11d9ca
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
  - maftools
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# maf-file-processing-with-maftools

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Process and visualize somatic mutation data in standard .maf format (e.g., TCGA mutation annotation files) through maftools as integrated into ARMT, supporting mutation summary plots, oncoplots, and differential gene mutation analysis between sample groups.

## When to use

Use when mutation information is provided as a standard .maf file (e.g., TCGA LUAD) and you need per-gene mutation significance between groups (p-value, FDR) or mutation visualization such as forest plots and oncoplots.

## When NOT to use

- Mutation data is not in standard .maf format — ARMT explicitly requires 'standard .maf file like the mutation annotation format file in TCGA'
- The goal is differential gene expression or GSVA score analysis rather than somatic mutation analysis (use the differential analysis of gene expression or GSVA instead)
- Samples cannot be integrated across clinical, GSVA, TPM and mutation data through common samples

## Inputs

- Standard .maf mutation annotation format file (e.g., TCGA LUAD MAF)
- Integrated data table (.csv) with clinical and GSVA score data merged via common samples
- Optional .gmt geneset file for oncoplots of specific genesets (Self-defined mode)

## Outputs

- Per-gene differential mutation significance table (gene, p-value, FDR/adj.p) filtered by p < 0.05 and FDR < 0.01
- Forest plot of differential mutation genes between groups
- MAF summary plots, oncogenic signaling pathway enrichment, oncoplots, and gene-level VAF plots

## How to apply

Load a standard .maf file (e.g., TCGA mutation annotation format) into ARMT, which wraps maftools for all mutation operations. In the Integration&Analysis page, merge the mutation data with clinical, GSVA score, and TPM data through common samples, then run differential gene mutation analysis between two sample groups (e.g., hypoxia-high vs hypoxia-low LUAD); the numeric difference factor is split into high/low with adjustable cutoff t (upper and lower t*100%), with the low group as control. Screen results by p-value and FDR (adj.p) — in the LUAD hypoxia case study, thresholds of p < 0.05 and FDR < 0.01 identified five significant SNV-mutated genes — and display the result as a forest plot. For visualization, use the Mutant mapping page (Maftools Summary or Self-defined modes) to draw summaries, oncoplots of interested pathways or genesets, and specify mutation types (deletion, insert, SNV) and the number of top mutant genes.

## Related tools

- **ARMT** (Shiny-based GUI wrapper providing Mutant mapping (Maftools Summary and Self-defined modes) and differential gene mutation analysis on integrated .maf data) — https://github.com/Dulab2020/ARMT
- **maftools** (Underlying Bioconductor package with which ARMT visualizes gene mutations in .maf files (summary, oncoplot, pathway enrichment, VAF))
- **GSVA** (Provides hypoxia gene set scores used to stratify samples into hypoxia-high/low groups before differential mutation testing)
- **R** (Runtime environment (R >= 4.0.2) in which ARMT and its Bioconductor dependencies are installed)

## Examples

```
if (!requireNamespace("devtools", quietly=TRUE)) install.packages("devtools"); devtools::install_github("Dulab2020/ARMT"); ARMT::run_app()
```

## Evaluation signals

- Output gene table contains exactly the five reported significant genes (TP53, CSMD3, TTN, CD163L1, OR4C15) for hypoxia-high vs hypoxia-low TCGA-LUAD, each with p < 0.05 and FDR < 0.01
- Input .maf parses as a standard TCGA mutation annotation format file without ID-mapping errors
- Differential mutation results are rendered as a forest plot equivalent to Fig. 3 B
- All merged rows share common samples across clinical, GSVA, TPM and mutation data
- Self-defined mutant mapping correctly restricts plots to the specified mutation types (deletion, insert, SNV) and top-N mutant genes

## Limitations

- No software version numbers for maftools or other dependencies are stated (only R >= 4.0.2)
- ARMT requires mutation input as a standard .maf file; other variant call formats are not supported
- No ARMT version number is stated in the article or README, so exact reproducibility of mutation results may vary with repository state

## Evidence
