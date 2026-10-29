---
name: mhc-region-exclusion
description: Use when preprocessing GWAS summary statistics (e.g. via scPagwas::GWAS_summary_input)
  before SNP-to-pathway linking, to drop SNPs in the MHC region that exhibit extensive
  LD and would bias pathway association estimates.
when_to_use_negative:
- When the trait of interest is explicitly MHC-mediated (e.g. autoimmune H loci) and
  the MHC signal is biologically central rather than a confounder
- When the input is not raw GWAS summary statistics but an already preprocessed Pagwas
  object in which MHC filtering has already been applied
- When using LDSC-SEG, RolyPoly, or EPIC-style pipelines whose own annotation/filtering
  scheme differs from scPagwas's 20 kb proximal window and KEGG pathway framework
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3517
tools:
- name: scPagwas
  role: Provides GWAS_summary_input (maf_filter=0.01) which removes MHC-region, sex-chromosome,
    and low-MAF SNPs during preprocessing
  repo: https://github.com/dengchunyu/scPagwas
- name: bigreadr
  role: Reads the GWAS summary statistics file (fread2) before scPagwas filtering
    is applied
- name: plink
  role: Computes pairwise LD (--autosome --r2 --ld-window-kb 1000 --ld-window-r2 0.2)
    on the filtered data, so MHC exclusion prevents LD-file contamination from this
    region
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.xgen.2023.100383
    title: Polygenic regression uncovers trait-relevant cellular contexts through
      pathway activation transformation of single-cell RNA sequencing data
  source_content_hash: sha256:fea70147eca790880ef285cf42ddf9f6f13f6a9f2de8b5a0c4707971276af95b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/dengchunyu/scPagwas
  repo_url: https://github.com/dengchunyu/scPagwas
  tools:
  - bigreadr
  - data.table
  - plink
  - scPagwas
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# mhc-region-exclusion

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

During scPagwas GWAS preprocessing, SNPs located in the major histocompatibility complex (MHC) region on chromosome 6 (25–35 Mbp) are removed because extensive linkage disequilibrium in this region confounds pathway-based polygenic regression.

## When to use

Use when preprocessing GWAS summary statistics (e.g. via scPagwas::GWAS_summary_input) before SNP-to-pathway linking, to drop SNPs in the MHC region that exhibit extensive LD and would bias pathway association estimates.

## When NOT to use

- When the trait of interest is explicitly MHC-mediated (e.g. autoimmune H loci) and the MHC signal is biologically central rather than a confounder
- When the input is not raw GWAS summary statistics but an already preprocessed Pagwas object in which MHC filtering has already been applied
- When using LDSC-SEG, RolyPoly, or EPIC-style pipelines whose own annotation/filtering scheme differs from scPagwas's 20 kb proximal window and KEGG pathway framework

## Inputs

- GWAS summary statistics file (e.g. GWAS_summ_example.txt) read with bigreadr::fread2
- maf_filter threshold (0.01) and MHC exclusion built into scPagwas::GWAS_summary_input

## Outputs

- Filtered GWAS data frame with MHC-region (Chr6: 25-35 Mbp), sex-chromosome (ChrX-Y), and MAF<0.01 SNPs removed
- Preprocessed Pagwas object (snp_gene_df, Pathway_list, pathway_blocks, chrom_ld) free of MHC-region SNPs

## How to apply

As part of the GWAS input step, read the summary statistics file with bigreadr::fread2 and run scPagwas::GWAS_summary_input with maf_filter = 0.01, which simultaneously drops SNPs with MAF below 0.01, SNPs on sex chromosomes (ChrX-Y), and the MHC region (Chr6: 25–35 Mbp). The rationale is that the MHC region carries unusually extensive LD, so variants there violate the roughly independent SNP-gene pairing assumptions made downstream by scPagwas::SnpToGene (marg = 10000) and by the LD matrix computation (plink --r2 --ld-window-r2 0.2). Validation then checks that no retained SNP maps to the excluded MHC interval or to a sex chromosome.

## Related tools

- **scPagwas** (Provides GWAS_summary_input (maf_filter=0.01) which removes MHC-region, sex-chromosome, and low-MAF SNPs during preprocessing) — https://github.com/dengchunyu/scPagwas
- **bigreadr** (Reads the GWAS summary statistics file (fread2) before scPagwas filtering is applied)
- **plink** (Computes pairwise LD (--autosome --r2 --ld-window-kb 1000 --ld-window-r2 0.2) on the filtered data, so MHC exclusion prevents LD-file contamination from this region)

## Examples

```
gwas_data <- bigreadr::fread2("GWAS_summ_example.txt"); gwas_filtered <- scPagwas::GWAS_summary_input(gwas_data, maf_filter = 0.01)
```

## Evaluation signals

- After running GWAS_summary_input, no retained SNP has coordinates within Chr6:25-35 Mbp (the MHC region)
- No retained SNP maps to sex chromosomes (ChrX-Y)
- No retained SNP has MAF below 0.01
- The preprocessed Pagwas object contains non-empty snp_gene_df, pathway_blocks, and chrom_ld elements consistent with the filtered SNP set
- Downstream SnpToGene output (snp_gene_df) contains no SNP-gene pairs derived from MHC-region SNPs

## Limitations

- MHC exclusion removes a genuinely polymorphic and disease-relevant interval, so traits whose causal variation resides in the MHC will lose that signal by design
- The filter is bundled with maf_filter=0.01 and sex-chromosome removal in a single function call; finer-grained control requires editing the scPagwas source or re-implementing the filter
- scPagwas annotates SNPs to genes solely by proximal distance (20 kb window); even after MHC removal, functionally informed SNP-to-gene linking is deferred to future work

## Evidence

- [other] workflow card describing GWAS_summary_input as the site of MHC exclusion: [evidence span withheld — anchor status: not_found]
- [readme] README grounding the wrapper that invokes the filtered GWAS input: [evidence span withheld — anchor status: external_doc]
