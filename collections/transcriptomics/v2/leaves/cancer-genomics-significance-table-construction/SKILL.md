---
name: cancer-genomics-significance-table-construction
description: Use when a MAF-format mutation dataset must be tested per-gene between
  two sample groups defined by a continuous factor (e.g., hypoxia GSVA score strata)
  to yield a p-value/FDR significance table.
when_to_use_negative:
- The goal is mutation visualization only (oncoplots, VAF, oncogenic pathway summary)
  rather than between-group significance testing — use ARMT's Mutant mapping page
  instead
- Input mutation data is not a standard .maf file (ARMT requires TCGA-like mutation
  annotation format for the differential mutation analysis)
- The comparison involves more than two groups or a continuous association rather
  than a high-vs-low split of a numeric factor
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: ARMT
  role: Runs the differential gene-mutation analysis between hypoxia-high and hypoxia-low
    groups via its shiny GUI and Integration&Analysis page
  repo: https://github.com/Dulab2020/ARMT
- name: maftools
  role: Backs the reading of the .maf file and the differential mutation testing/forest
    plot output
- name: GSVA
  role: Produces the hypoxia GSVA scores used to stratify LUAD samples into high and
    low mutation-analysis groups
- name: R
  role: Runtime environment for ARMT (requires R >= 4.0.2 installed via devtools/remotes)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.009
    title: 'ARMT: An automatic RNA-seq data mining tool based on comprehensive and
      integrative analysis in cancer research'
  source_content_hash: sha256:9d45abe23471d23f79348d13aa31aab5f69d5130b15d2a3e87c9e676137e9d6e
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# cancer-genomics-significance-table-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds a gene-level significance table for differentially mutated genes between two phenotype strata (e.g., hypoxia-high vs hypoxia-low TCGA-LUAD) using ARMT's maftools-backed differential mutation analysis, filtered at p < 0.05 and FDR < 0.01.

## When to use

Use when a MAF-format mutation dataset must be tested per-gene between two sample groups defined by a continuous factor (e.g., hypoxia GSVA score strata) to yield a p-value/FDR significance table.

## When NOT to use

- The goal is mutation visualization only (oncoplots, VAF, oncogenic pathway summary) rather than between-group significance testing — use ARMT's Mutant mapping page instead
- Input mutation data is not a standard .maf file (ARMT requires TCGA-like mutation annotation format for the differential mutation analysis)
- The comparison involves more than two groups or a continuous association rather than a high-vs-low split of a numeric factor

## Inputs

- TCGA-LUAD mutation data in MAF (mutation annotation format) file
- Pre-computed hypoxia GSVA scores per sample (.csv from the Normalization&GSVA page)
- TCGA clinical data with a group column for sample stratification
- TPM expression matrix (.csv, optional, for data integration)

## Outputs

- Filtered gene-level significance table (gene, p-value, FDR) written to hypoxia_high_mutated_genes.csv
- Forest plot of differential mutation genes between hypoxia-high and hypoxia-low groups (Fig. 3 B equivalent)

## How to apply

Stratify samples by their hypoxia GSVA score into hypoxia-high and hypoxia-low groups — the case study used top and bottom 30% of the GSVA score, while ARMT exposes an adjustable group cutoff 't' (upper and lower t*100%) for continuous factors. Integrate clinical data, GSVA score, TPM expression, and mutation profile through common samples, entering the mutation data as a standard TCGA-like .maf file. Run ARMT's differential analysis of gene mutation (maftools-based) between the hypoxia-high and hypoxia-low groups, treating the low group as control. Filter per-gene results with p < 0.05 and FDR < 0.01, write the gene/p-value/FDR table to hypoxia_high_mutated_genes.csv, and visualize the differential mutation genes as a forest plot (Fig. 3 B equivalent). The rationale is that the paper reports exactly this analysis reproduced five significantly SNV-mutated genes in the hypoxia-high LUAD group.

## Related tools

- **ARMT** (Runs the differential gene-mutation analysis between hypoxia-high and hypoxia-low groups via its shiny GUI and Integration&Analysis page) — https://github.com/Dulab2020/ARMT
- **maftools** (Backs the reading of the .maf file and the differential mutation testing/forest plot output)
- **GSVA** (Produces the hypoxia GSVA scores used to stratify LUAD samples into high and low mutation-analysis groups)
- **R** (Runtime environment for ARMT (requires R >= 4.0.2 installed via devtools/remotes))

## Examples

```
devtools::install_github("Dulab2020/ARMT"); ARMT::run_app()
```

## Evaluation signals

- The output table contains the five reported genes TP53, CSMD3, TTN, CD163L1, and OR4C15 as significantly mutated in the hypoxia-high group
- All retained genes satisfy both thresholds p < 0.05 and FDR < 0.01
- The CSV schema has the expected columns (gene, p-value, FDR) and is saved as hypoxia_high_mutated_genes.csv
- Sample grouping corresponds to the intended strata of the hypoxia GSVA score (top/bottom 30% in the case study) rather than an arbitrary split
- The forest plot matches the paper's Fig. 3 B equivalent in gene content and direction (mutation enrichment in the hypoxia-high group)

## Limitations

- Significance is group-specific: TP53, CSMD3 and TTN were also mutated in glycolysis score high samples, so hits are not exclusive to the hypoxia-high stratum
- The high/low grouping cutoff is adjustable ('t' in ARMT, with top/bottom 50% as the documented behavior); changing the cutoff (e.g., to the case study's 30%) can change which genes pass significance
- Different statistical tools based on different principles may lead to different results, so the five-gene result is tool-dependent

## Evidence

- [methods] five genes with SNV including TP53, CSMD3, TTN, CD163L1, OR4C15 were significantly mutated in hypoxia-high groups: "five genes with SNV including TP53, CSMD3, TTN, CD163L1, OR4C15 were significantly mutated in hypoxia-high groups"
- [methods] clinical data, GSVA score, gene expression profile data (TPM) and mutation profile data can be merged through common samples: "clinical data, GSVA score, gene expression profile data (TPM) and mutation profile data can be merged through common samples"
- [readme] To analyze difference of gene mutation, the mutation information of samples in integrated data should be entered through .maf file.: "To analyze difference of gene mutation, the mutation information of samples in integrated data should be entered through .maf file."
- [readme] The group cut off value 't' is a adjustable parameter in ARMT(upper and lower t*100%).: "The group cut off value 't' is a adjustable parameter in ARMT(upper and lower t*100%)."
- [intro] Based on different statistical principles, different tools may lead to different results: "Based on different statistical principles, different tools may lead to different results"
