---
name: differential-mutation-frequency-analysis
description: Use when you have a TCGA-style MAF mutation file and sample groups defined
  by a continuous factor (e.g. top/bottom hypoxia GSVA score) and need per-gene mutation
  p-values and FDR between groups.
when_to_use_negative:
- You only need mutation summary/oncoplot visualization of a single cohort without
  group comparison — use ARMT's 'Mutant mapping' / maftools Summary mode instead
- Input is somatic copy-number variation or CNV burden rather than SNV; the validated
  case study screens SNVs in a standard .maf file
- Groups cannot be defined through a shared sample column between MAF and phenotype/GSVA
  data (no common-sample integration possible)
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3272
- http://edamontology.org/topic_3474
tools:
- name: ARMT
  role: Shiny-based GUI that integrates clinical, GSVA, TPM and MAF data by common
    samples and runs the gene-mutation differential analysis with adjustable group
    cutoff t and p-value/FDR/logFC screening
  repo: https://github.com/Dulab2020/ARMT
- name: maftools
  role: Bioconductor dependency underlying ARMT's mutant mapping and mutation differential
    analysis of .maf files
- name: GSVA
  role: Computes the hypoxia gene-set scores used to stratify LUAD samples into high/low
    mutation-comparison groups
- name: R
  role: Runtime environment (>= 4.0.2) hosting ARMT and its Bioconductor dependencies
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.009
    title: 'ARMT: An automatic RNA-seq data mining tool based on comprehensive and
      integrative analysis in cancer research'
  source_content_hash: sha256:d31dc39e841efc54dc1534f63f1d545ce652207aed764381cab2ef12301846b2
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3272
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# differential-mutation-frequency-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identify genes whose SNV mutation frequency differs significantly between two sample strata (e.g. hypoxia-high vs hypoxia-low TCGA-LUAD) by running ARMT's mutation differential analysis, which wraps maftools on a merged MAF-plus-phenotype dataset and visualizes results as a forest plot.

## When to use

Use when you have a TCGA-style MAF mutation file and sample groups defined by a continuous factor (e.g. top/bottom hypoxia GSVA score) and need per-gene mutation p-values and FDR between groups.

## When NOT to use

- You only need mutation summary/oncoplot visualization of a single cohort without group comparison — use ARMT's 'Mutant mapping' / maftools Summary mode instead
- Input is somatic copy-number variation or CNV burden rather than SNV; the validated case study screens SNVs in a standard .maf file
- Groups cannot be defined through a shared sample column between MAF and phenotype/GSVA data (no common-sample integration possible)

## Inputs

- Standard .maf mutation annotation format file (e.g. from TCGA-LUAD)
- Hypoxia (or other gene set) GSVA score matrix in .csv
- Clinical data table for TCGA samples
- Gene expression TPM matrix in .csv (row names = gene symbols, columns = sample IDs) for integration
- Group definition: continuous GSVA score column split into high/low strata

## Outputs

- Per-gene differential mutation table (gene, p-value, FDR/adj.p)
- Forest plot of differentially mutated genes (Fig. 3B equivalent)
- Filtered list of significantly mutated genes (e.g. TP53, CSMD3, TTN, CD163L1, OR4C15 in hypoxia-high LUAD)

## How to apply

First stratify samples by the continuous factor (hypoxia GSVA score) into high/low groups; in ARMT's GUI the group cutoff 't' is adjustable (upper and lower t*100%). Integrate clinical, GSVA score, TPM and MAF mutation data through common samples in the 'Integration&Analysis' page, using the 'Integrate Maf' button to merge mutation information. Then run ARMT's differential analysis on gene mutation via edgeR-backed comparisons implemented through maftools, which computes per-gene mutation differences between the experiment (hypoxia-high) and control (hypoxia-low) groups. Screen the resulting table by p-value and FDR (adj.p); the paper's case study validated genes at p < 0.05 and FDR < 0.01. Render the surviving genes as a forest plot (the paper's Fig. 3B equivalent) to display the differentially mutated genes such as TP53, CSMD3, TTN, CD163L1 and OR4C15.

## Related tools

- **ARMT** (Shiny-based GUI that integrates clinical, GSVA, TPM and MAF data by common samples and runs the gene-mutation differential analysis with adjustable group cutoff t and p-value/FDR/logFC screening) — https://github.com/Dulab2020/ARMT
- **maftools** (Bioconductor dependency underlying ARMT's mutant mapping and mutation differential analysis of .maf files)
- **GSVA** (Computes the hypoxia gene-set scores used to stratify LUAD samples into high/low mutation-comparison groups)
- **R** (Runtime environment (>= 4.0.2) hosting ARMT and its Bioconductor dependencies)

## Examples

```
if (!requireNamespace("devtools", quietly=TRUE)) install.packages("devtools"); devtools::install_github("Dulab2020/ARMT"); ARMT::run_app()
```

## Evaluation signals

- The filtered significance table contains exactly the five reported genes (TP53, CSMD3, TTN, CD163L1, OR4C15) at p < 0.05 and FDR < 0.01 for hypoxia-high LUAD
- The result is rendered as a forest plot, matching the ARMT Figure 8 behavior for gene-mutation differential analysis
- Integrated data contain the same common samples across clinical, GSVA and MAF sources before analysis
- TP53, CSMD3 and TTN additionally overlap the reported glycolysis-score-high mutated gene set, serving as a cross-check
- The .maf input parses cleanly in maftools/ARMT (standard TCGA mutation annotation format)

## Limitations

- No ARMT version number or tool version numbers are stated; results may vary with dependency versions
- Grouping cutoff for continuous factors defaults to top and bottom 50% (the case study used top and bottom 30%), so cutoff choice must be documented to reproduce the paper
- Different statistical principles across tools can lead to different results, so findings are tool-dependent

## Evidence
