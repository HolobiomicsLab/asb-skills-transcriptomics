---
name: tcga-luad-mutation-data-acquisition
description: Use when a TCGA-LUAD SNV dataset in MAF format must be obtained and staged
  so ARMT can merge mutation profiles with clinical, TPM and GSVA score data by common
  samples before differential mutation testing.
when_to_use_negative:
- Mutation data are not in a standard .maf format (ARMT requires the TCGA-style mutation
  annotation format) or come from a non-TCGA cohort without mappable sample IDs for
  integration
- The goal is differential expression or GSVA-score analysis of RNA-seq counts rather
  than SNV mutation significance testing (use ARMT's differential expression or GSVA
  differential modules instead)
- Sample-level integration is impossible because clinical, GSVA and mutation data
  do not share common sample identifiers
edam_topics:
- http://edamontology.org/topic_2648
tools:
- name: ARMT
  role: Automatic RNA-seq data mining tool providing TCGA clinical data, data integration
    across clinical/GSVA/TPM/MAF inputs, and the GUI used to run differential mutation
    analysis
  repo: https://github.com/Dulab2020/ARMT
- name: maftools
  role: R/Bioconductor package used by ARMT to read the .maf file, compute per-gene
    mutation significance and render forest/oncoplot visualizations
  repo: https://github.com/Dulab2020/ARMT
- name: GSVA
  role: Produces the hypoxia GSVA scores in ARMT that define the high/low sample grouping
    used as the difference factor in the mutation analysis
  repo: https://github.com/Dulab2020/ARMT
- name: R
  role: Runtime environment (>= 4.0.2) hosting ARMT and its Bioconductor dependencies;
    ARMT is installed via devtools and launched with ARMT::run_app()
  repo: https://github.com/Dulab2020/ARMT
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.009
    title: 'ARMT: An automatic RNA-seq data mining tool based on comprehensive and
      integrative analysis in cancer research'
  source_content_hash: sha256:180999f99846e155e2cfdde449792b0e8aff95e61897e5c006cbf043defc68fa
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2648
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# tcga-luad-mutation-data-acquisition

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Acquires the TCGA lung adenocarcinoma (LUAD) mutation data in standard MAF format and stages it for integration with clinical, TPM and GSVA-score data inside ARMT, so that downstream differential mutation analysis (e.g. hypoxia-high vs hypoxia-low) can be run with maftools-backed plotting.

## When to use

Use when a TCGA-LUAD SNV dataset in MAF format must be obtained and staged so ARMT can merge mutation profiles with clinical, TPM and GSVA score data by common samples before differential mutation testing.

## When NOT to use

- Mutation data are not in a standard .maf format (ARMT requires the TCGA-style mutation annotation format) or come from a non-TCGA cohort without mappable sample IDs for integration
- The goal is differential expression or GSVA-score analysis of RNA-seq counts rather than SNV mutation significance testing (use ARMT's differential expression or GSVA differential modules instead)
- Sample-level integration is impossible because clinical, GSVA and mutation data do not share common sample identifiers

## Inputs

- TCGA-LUAD mutation data in standard MAF format (.maf)
- TCGA-LUAD clinical data (built into ARMT, obtainable from the Data page)
- Hypoxia GSVA score matrix (.csv) computed by ARMT from the LUAD counts matrix
- TPM gene expression matrix (.csv, normalized from counts via ARMT)

## Outputs

- Integrated data table merging clinical, GSVA, TPM and mutation profile data by common samples
- Gene-level differential mutation result table (gene, p-value, FDR/adj.p)
- Forest plot of differential mutation genes (Fig. 3 B equivalent)
- hypoxia_high_mutated_genes.csv containing significantly mutated genes (p < 0.05, FDR < 0.01)

## How to apply

Download the TCGA-LUAD mutation annotation file (standard .maf) as specified in the ARMT methods data-preparation step, since ARMT requires 'standard .maf file like the mutation annotation format file in TCGA'. Launch ARMT via its shiny GUI and use the Data page to pull built-in TCGA clinical data for the 33 cancer types, then on the Integration&Analysis page input the clinical, GSVA and TPM data as .csv files and enter the mutation data through the .maf file so all data are merged through common samples. Select the hypoxia GSVA score as the grouping/difference factor, letting ARMT split samples into high and low groups (the case study used top and bottom 30% of the hypoxia GSVA score), with the low group as control. Finally run ARMT's differential analysis of gene mutation, which computes per-gene mutation p-values and FDR-adjusted values and renders a forest plot, screening results by p-value and FDR (the LUAD hypoxia case used p < 0.05 and FDR < 0.01).

## Related tools

- **ARMT** (Automatic RNA-seq data mining tool providing TCGA clinical data, data integration across clinical/GSVA/TPM/MAF inputs, and the GUI used to run differential mutation analysis) — https://github.com/Dulab2020/ARMT
- **maftools** (R/Bioconductor package used by ARMT to read the .maf file, compute per-gene mutation significance and render forest/oncoplot visualizations) — https://github.com/Dulab2020/ARMT
- **GSVA** (Produces the hypoxia GSVA scores in ARMT that define the high/low sample grouping used as the difference factor in the mutation analysis) — https://github.com/Dulab2020/ARMT
- **R** (Runtime environment (>= 4.0.2) hosting ARMT and its Bioconductor dependencies; ARMT is installed via devtools and launched with ARMT::run_app()) — https://github.com/Dulab2020/ARMT

## Examples

```
devtools::install_github("Dulab2020/ARMT"); ARMT::run_app()  # then load TCGA-LUAD .maf + hypoxia GSVA .csv on the Integration&Analysis page
```

## Evaluation signals

- The integrated data table contains the union of samples shared between the clinical, GSVA, TPM and .maf inputs — no sample IDs present in the MAF are dropped during integration
- Filtering at p < 0.05 and FDR < 0.01 yields exactly five significantly SNV-mutated genes: TP53, CSMD3, TTN, CD163L1 and OR4C15, matching the published hypoxia-high LUAD result
- The hypoxia-high/hypoxia-low grouping corresponds to the intended top/bottom fraction of the hypoxia GSVA score distribution (case-study Step 1 used 30%)
- The resulting forest plot reproduces the differential mutation visualization (Fig. 3 B equivalent) with the five reported genes
- hypoxia_high_mutated_genes.csv is written with columns gene, p-value and FDR, and every listed gene satisfies both thresholds

## Limitations

- ARMT requires a standard .maf file; non-standard or reformatted mutation annotation files may fail to load in the Mutant mapping / differential mutation module
- No software or ARMT version numbers are stated in the article or README, so exact reproducibility of versioned results is not guaranteed
- ARMT provides built-in TCGA clinical data for only 33 cancer types; other cohorts must be supplied by the user in the same format

## Evidence
