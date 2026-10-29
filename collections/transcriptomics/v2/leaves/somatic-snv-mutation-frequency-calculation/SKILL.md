---
name: somatic-snv-mutation-frequency-calculation
description: Use when you have a list of predicted driver genes plus TCGA somatic
  SNV calls and need to classify genes as rarely- vs frequently-mutated per cancer
  type, e.g. to validate a driver-discovery method against CGC.
when_to_use_negative:
- Input is raw TCGA RNA-seq expression matrices (tumor/normal) — that is the input
  to WMDS.net driver-node discovery, not to this mutation-frequency characterization
  step
- Goal is to discover driver genes de novo; this skill only characterizes the mutation
  frequency of an existing predicted-driver list and does not rank or predict drivers
- Only germline variants or variant calls without non-synonymous consequence annotation
  are available, since the frequency is defined specifically over non-synonymous somatic
  SNVs
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3308
tools:
- name: WMDS.net
  role: Supplies the predicted driver gene lists whose somatic SNV mutation frequencies
    are computed; predictions come from applying WMDS.net to TCGA RNA-seq differential
    co-expression networks
  repo: https://github.com/chaofen123/WMDS.net
- name: Matlab R2020b
  role: Implementation environment for the WMDS.net pipeline whose outputs feed this
    frequency analysis (binary integer programming via intlinprog in the Optimization
    ToolBox)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad071
    title: 'WMDS.net: a network control framework for identifying key players in transcriptome
      programs'
  source_content_hash: sha256:9136af2073bea2d44b55950c48fb5bc97f02dfeae08d17a17b9f9f3c64430ccc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chaofen123/WMDS.net
  tools:
  - Matlab R2020b
  - WMDS.net
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# somatic-snv-mutation-frequency-calculation

## Summary

Computes the fraction of TCGA tumor samples carrying non-synonymous somatic SNVs in each WMDS.net-predicted driver gene per cancer type, then splits CGC-overlapping predicted drivers into low-frequency (<0.05) versus high-frequency (>0.05) classes to demonstrate that WMDS.net recovers rarely-mutated driver genes.

## When to use

Use when you have a list of predicted driver genes plus TCGA somatic SNV calls and need to classify genes as rarely- vs frequently-mutated per cancer type, e.g. to validate a driver-discovery method against CGC.

## When NOT to use

- Input is raw TCGA RNA-seq expression matrices (tumor/normal) — that is the input to WMDS.net driver-node discovery, not to this mutation-frequency characterization step
- Goal is to discover driver genes de novo; this skill only characterizes the mutation frequency of an existing predicted-driver list and does not rank or predict drivers
- Only germline variants or variant calls without non-synonymous consequence annotation are available, since the frequency is defined specifically over non-synonymous somatic SNVs

## Inputs

- WMDS.net-predicted driver gene lists for 14 TCGA cancer types (BLCA, BRCA, COAD, HNSC, KICH, KIRC, KIRP, LIHC, LUAD, LUSC, PRAD, STAD, THCA, UCEC)
- CGC (Cancer Gene Census) gene list of 616 genes from COSMIC (https://cancer.sanger.ac.uk/cosmic)
- TCGA somatic SNV data (non-synonymous mutations with sample and gene identifiers) for the same 14 cancer types from the Genomic Data Commons (https://portal.gdc.cancer.gov)

## Outputs

- Per-gene, per-cancer-type SNV mutation frequency (fraction of tumor samples with non-synonymous somatic SNVs in the gene)
- Per-cancer-type fractions of CGC-overlapping predicted drivers classified as low-frequency (<0.05) vs high-frequency (>0.05), as in the Figure 4 bar chart
- Case validation record confirming ERBB3 is a WMDS.net-predicted LUAD driver in the low-frequency (<0.05) class

## How to apply

Load the WMDS.net driver gene predictions for the 14 TCGA cancer types (BLCA, BRCA, COAD, HNSC, KICH, KIRC, KIRP, LIHC, LUAD, LUSC, PRAD, STAD, THCA, UCEC) and the CGC gene list of 616 genes from COSMIC. Download TCGA somatic SNV data for the same 14 cancer types from the Genomic Data Commons portal. For each predicted driver gene in each cancer type, compute mutation frequency as the fraction of tumor samples carrying non-synonymous somatic SNVs in that gene, using the per-cancer-type TCGA tumor sample count as denominator. Intersect predicted drivers with the CGC list per cancer type, then split CGC-overlapping genes at the 0.05 mutation-frequency cutoff into low-frequency (<0.05) and high-frequency (>0.05) categories and tabulate the fraction of each category, reproducing the per-cancer-type bar chart of Figure 4. As a case check, confirm ERBB3 appears among WMDS.net-predicted LUAD drivers with LUAD mutation frequency below 0.05, consistent with its reported rarity in COSMIC.

## Related tools

- **WMDS.net** (Supplies the predicted driver gene lists whose somatic SNV mutation frequencies are computed; predictions come from applying WMDS.net to TCGA RNA-seq differential co-expression networks) — https://github.com/chaofen123/WMDS.net
- **Matlab R2020b** (Implementation environment for the WMDS.net pipeline whose outputs feed this frequency analysis (binary integer programming via intlinprog in the Optimization ToolBox))

## Examples

```
freq = {g: snv_df[(snv_df.gene==g) & (snv_df.variant_class=='missense')].Tumor_Sample_Barcode.nunique()/n_tumors for g in driver_genes}
```

## Evaluation signals

- Per-cancer-type low/high-frequency fractions reproduce the Figure 4 pattern: most CGC-overlapping predicted drivers fall in the rarely-mutated (<0.05) class across the 14 TCGA cancer types
- Every gene's frequency is computed as (number of tumor samples with non-synonymous somatic SNVs in the gene) / (number of TCGA tumor samples for that cancer type), with non-synonymous-only filtering verified
- Frequency split is binary at the 0.05 cutoff: low-frequency < 0.05, high-frequency > 0.05, applied only to predicted drivers that overlap the 616-gene CGC list
- ERBB3 is present in the WMDS.net LUAD driver list and its LUAD mutation frequency is < 0.05, matching its reported rarity in COSMIC
- Sample denominators and gene-symbol keys match the TCGA cancer-type sample counts (the same 14 cancer types as the WMDS.net driver predictions)

## Limitations

- The low/high-frequency classification uses a single hard 0.05 mutation-frequency threshold, so genes near the cutoff are sensitive to sample-size and calling differences across TCGA cancer types
- Frequencies are based only on non-synonymous somatic SNVs from TCGA/GDC; indels, CNVs, and other variant classes are not counted, which may understate a gene's total alteration burden
- Validation depends on the 616-gene CGC list as the reference standard, and the article's discussion section (limitations, changelog) was not available in the source context

## Evidence

- [abstract] WMDS.net driver predictions applied to TCGA: "we applied WMDS.net to the discovery of cancer driver genes in RNA-seq datasets from The Cancer Genome Atlas"
- [abstract] Precision/recall balance justifies using these predictions: "WMDS.net is powerful among various cancer datasets and outperformed the other top-tier tools with a better balance between precision and recall"
- [intro] Matlab implementation of the upstream pipeline: "solve the binary integer-programming problem using function “intlinprog” which is available in the Optimization ToolBox of MatLab version R2020b"
- [readme] Input matrix conventions for the driver-discovery outputs: "The rows of the expression matrices represent gene expression, and the columns represent samples. The row and column names of the expression matrices are gene symbols and sample ids, respectively."
