---
name: driver-gene-frequency-stratification
description: Use when you have per-patient personalized driver gene lists from sample-specific
  networks (e.g., WMDS.netP on TCGA RNA-seq) and need to tabulate how often each driver
  gene recurs across patients by cancer type.
when_to_use_negative:
- Your driver calls come from a global (pan-cohort) method like WMDS.net rather than
  per-patient WMDS.netP lists — recurrence stratification requires patient-level driver
  sets, and the global list has no per-patient frequency (e.g., CDKN2A is in the LUAD
  WMDS.netP list but absent from the global WMDS.net list)
- You only have a single cohort without per-patient sample-specific networks — with
  no per-patient driver sets there is no recurrence frequency to bin
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: WMDS.net
  role: Source of the per-patient (WMDS.netP) personalized driver gene lists whose
    recurrence frequencies are binned
  repo: github.com/chaofen123/WMDS.net
- name: Matlab R2020b
  role: Runs intlinprog (Optimization ToolBox) to solve the weighted minimum dominating
    set binary integer program that yields each patient's driver genes
- name: DawnRank
  role: Provides the gene interaction network used to construct Gene_Net, the network
    backbone for driver discovery
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad071
    title: 'WMDS.net: a network control framework for identifying key players in transcriptome
      programs'
  source_content_hash: sha256:c8eb442afe4f5e44f8aac94ff1f4c6c220ea1602da782d444e32d6ea0a76144c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DawnRank
  - Matlab R2020b
  - WMDS.net
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# driver-gene-frequency-stratification

## Summary

Stratify personalized driver genes predicted per-patient by WMDS.net (WMDS.netP) into high (>=0.6), medium (0.3-0.6), and low (<0.3) recurrence-frequency classes per cancer type, in order to quantify within-cancer-type tumor heterogeneity.

## When to use

Use when you have per-patient personalized driver gene lists from sample-specific networks (e.g., WMDS.netP on TCGA RNA-seq) and need to tabulate how often each driver gene recurs across patients by cancer type.

## When NOT to use

- Your driver calls come from a global (pan-cohort) method like WMDS.net rather than per-patient WMDS.netP lists — recurrence stratification requires patient-level driver sets, and the global list has no per-patient frequency (e.g., CDKN2A is in the LUAD WMDS.netP list but absent from the global WMDS.net list)
- You only have a single cohort without per-patient sample-specific networks — with no per-patient driver sets there is no recurrence frequency to bin

## Inputs

- Per-patient personalized driver gene lists (WMDS.netP output) for each of 14 TCGA cancer types
- TCGA RNA-seq expression matrices (rows = gene symbols, columns = sample ids; e.g., TCGA_BLCAtumor and TCGA_BLCAnormal)
- Gene_Net hub network (intersection of the DawnRank gene interaction network and TCGA genes)

## Outputs

- personalized_driver_frequency_table.csv: per-cancer-type distribution of high-frequency (>=0.6), medium-frequency (0.3-0.6), and low-frequency (<0.3) personalized driver genes
- Cancer-type-specific recurrence frequency values for each personalized driver gene

## How to apply

For each of the 14 cancer types, compute the recurrence frequency of every personalized driver gene across patients, where each gene's driver call comes from solving the weighted minimum dominating set (WMDS.net) as a binary integer program with Matlab R2020b's intlinprog on the patient's sample-specific network built against the TCGA normal-sample reference co-expression network (edges retained at Delta-r Z-test P-value < 0.05; node weight wi = (sum over adjacent j of -log10(p_ij))^(-gamma), gamma = 0.01). Bin each gene as high-frequency (frequency >= 0.6), medium-frequency (0.3 <= f < 0.6), or low-frequency (f < 0.3) using these fixed cutoffs. Tabulate the per-type counts/proportions of the three classes and write them to personalized_driver_frequency_table.csv. The rationale is that recurrence frequency across patients directly measures whether driver genes are shared or patient-specific, so the class proportions quantify tumor heterogeneity within a cancer type.

## Related tools

- **WMDS.net** (Source of the per-patient (WMDS.netP) personalized driver gene lists whose recurrence frequencies are binned) — github.com/chaofen123/WMDS.net
- **Matlab R2020b** (Runs intlinprog (Optimization ToolBox) to solve the weighted minimum dominating set binary integer program that yields each patient's driver genes)
- **DawnRank** (Provides the gene interaction network used to construct Gene_Net, the network backbone for driver discovery)

## Examples

```
binned = pd.cut(driver_freq.frequency, bins=[0, 0.3, 0.6, 1.0], labels=['low', 'medium', 'high']); pd.crosstab(driver_freq.cancer_type, binned).to_csv('personalized_driver_frequency_table.csv')
```

## Evaluation signals

- In every cancer type, the low-frequency class (<0.3) must account for the largest proportion of personalized driver genes, consistent with Figure 5 of the paper
- CDKN2A must appear in the LUAD WMDS.netP personalized driver list while being absent from the global WMDS.net driver list
- For each cancer type, the sum of high + medium + low class counts must equal the total number of distinct personalized driver genes for that type
- Every binned gene must fall in exactly one class under the fixed cutoffs (>=0.6, 0.3<=f<0.6, <0.3); no unclassified or double-binned genes
- The number of cancer types tabulated must be 14, matching the paper's TCGA analysis

## Limitations

- The finding that low-frequency personalized driver genes dominate across all cancer types indicates each patient carries a unique set of driver genes, so class-level summaries cannot be interpreted as cohort-shared driver signatures
- The README documents the tested data only as TCGA_BLCAtumor and TCGA_BLCAnormal; applying to other cancer types requires assembling equivalent tumor/normal expression matrices per type
- No discussion-section content is available in the extracted article (only a changelog placeholder), so caveats beyond the main-text findings are not documented

## Evidence

- [abstract] WMDS.net applied to TCGA driver discovery: "we applied WMDS.net to the discovery of cancer driver genes in RNA-seq datasets from The Cancer Genome Atlas"
- [readme] Matlab intlinprog implementation: "solve the binary integer-programming problem using function "intlinprog" which is available in the Optimization ToolBox of MatLab version R2020b"
- [intro] Gene_Net provenance: "Gene_Net is the hub network of intersection of gene interaction network constructed in Dawnrank[1] and the genes in TCGA."
- [readme] Expected expression matrix format: "The rows of the expression matrices represent gene expression, and the columns represent samples. The row and column names of the expression matrices are gene symbols and sample ids, respectively."
