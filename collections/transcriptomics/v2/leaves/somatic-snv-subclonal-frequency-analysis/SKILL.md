---
name: somatic-snv-subclonal-frequency-analysis
description: Use when per-cell genotype matrices for somatic SNVs (from cb_sniffer
  or VarTrix on 10x Chromium data) exist and you need to estimate subclonal frequencies
  and check sensitivity against bulk VAFs.
when_to_use_negative:
- When the goal is de novo variant discovery — cb_sniffer and VarTrix only genotype
  pre-called variants (VarTrix explicitly 'does not perform variant calling'); somatic
  SNVs must come from eWGS of paired tumor/normal
- When determining true wild-type genotype per cell is required — dropout makes it
  impossible to confirm a cell is truly wild-type for a given mutation
- When multi-allelic sites are of interest — VarTrix ignores all multi-allelic sites,
  leaving them empty in the output matrix
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: python3
  role: Computes scVAF, MCF, and MCDR per variant, merges with bulk VAFs, and calculates
    Pearson correlations; run inside Docker image sridnona/python3:180925.v1
- name: cb_sniffer
  role: Mutation barcode caller that parses CB barcodes from 10x Cell Ranger BAMs
    to call mutant and ref barcodes per cell, feeding the per-cell genotype matrix
  repo: https://github.com/sridnona/cb_sniffer
- name: VarTrix
  role: Alternative tool to assign previously defined variants to individual cells
    from 10x data, producing genome matrices used for subclonal frequency estimation
  repo: https://github.com/10XGenomics/vartrix
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-11591-1
    title: A general approach for detecting expressed mutations in AML cells using
      single cell RNA-sequencing
  source_content_hash: sha256:6941f6227b36d90754b5e078173ddc268c1e10fce50c0a6aa84f49c8c227dc43
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sridnona/cb_sniffer
  tools:
  - Docker
  - VarTrix
  - cb_sniffer
  - python3
  - sridnona/python3:180925.v1
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# somatic-snv-subclonal-frequency-analysis

## Summary

Quantifies subclonal somatic SNV burden from 10x scRNA-seq genotype matrices by computing read-based (scVAF) and cell-based (MCF, MCDR) metrics, then validating them against eWGS and bulk RNA-seq VAFs via Pearson correlation.

## When to use

Use when per-cell genotype matrices for somatic SNVs (from cb_sniffer or VarTrix on 10x Chromium data) exist and you need to estimate subclonal frequencies and check sensitivity against bulk VAFs.

## When NOT to use

- When the goal is de novo variant discovery — cb_sniffer and VarTrix only genotype pre-called variants (VarTrix explicitly 'does not perform variant calling'); somatic SNVs must come from eWGS of paired tumor/normal
- When determining true wild-type genotype per cell is required — dropout makes it impossible to confirm a cell is truly wild-type for a given mutation
- When multi-allelic sites are of interest — VarTrix ignores all multi-allelic sites, leaving them empty in the output matrix

## Inputs

- Per-cell genotype matrix of somatic SNVs (mutant/ref barcodes per cell) from cb_sniffer or VarTrix
- Somatic SNV variant list with chrm/start/stop/ref/var/gene_name/trv_type columns
- eWGS variant allele fractions per variant
- Bulk RNA-seq variant allele fractions per variant

## Outputs

- scvaf_correlations.tsv — per-variant scVAF, MCF, MCDR merged with eWGS and bulk RNA-seq VAFs, plus Pearson r values
- Pearson correlation coefficients (e.g., MCF vs eWGS r≈0.69, scVAF vs eWGS r≈0.68)

## How to apply

Load the per-cell genotype matrix (mutant and reference barcodes called per cell) produced by cb_sniffer/VarTrix in Python 3. For each somatic SNV, compute scVAF as the fraction of mutant reads over total informative reads across cells, Mutant Cell Fraction (MCF) as the fraction of cells carrying the variant, and Mutant Cell Detection Rate (MCDR) as the fraction of cells with informative coverage in which the variant is detected. Merge these per-variant metrics with eWGS VAFs and bulk RNA-seq VAFs by variant identifier and compute Pearson correlations (scVAF vs eWGS, MCF vs eWGS, scVAF vs bulk RNA, bulk RNA vs eWGS). Write the merged metrics and coefficients to scvaf_correlations.tsv. Rationale: dropout reduces mutation-detection sensitivity by roughly a factor of two, so single-cell scVAFs are expected to be attenuated relative to eWGS VAFs; the correlations quantify how faithfully the scRNA-seq genotyping recovers subclonal structure.

## Related tools

- **python3** (Computes scVAF, MCF, and MCDR per variant, merges with bulk VAFs, and calculates Pearson correlations; run inside Docker image sridnona/python3:180925.v1)
- **cb_sniffer** (Mutation barcode caller that parses CB barcodes from 10x Cell Ranger BAMs to call mutant and ref barcodes per cell, feeding the per-cell genotype matrix) — https://github.com/sridnona/cb_sniffer
- **VarTrix** (Alternative tool to assign previously defined variants to individual cells from 10x data, producing genome matrices used for subclonal frequency estimation) — https://github.com/10XGenomics/vartrix

## Examples

```
python3 cb_sniffer.py -f 0 -mq 0 -bq 1 cellranger.bam somatic_variants.tsv good_barcodes.tsv UPN1
```

## Evaluation signals

- Pearson r values match the reported results within tolerance: MCF vs eWGS r≈0.69, scVAF vs eWGS r≈0.68, bulk RNA-seq vs eWGS r=0.52, scVAF vs bulk RNA-seq r=0.34
- scVAF is attenuated relative to eWGS VAF, consistent with dropout halving mutation-detection sensitivity
- All three metrics (scVAF, MCF, MCDR) are bounded in [0,1] and defined only over variants/cells with informative coverage
- scvaf_correlations.tsv contains one row per somatic SNV with merged eWGS and bulk RNA-seq VAFs keyed by variant identifier

## Limitations

- Dropout (transcript and allelic) occurs with most scRNA-seq platforms and reduces the sensitivity of mutation detection by a factor of two
- Coverage drops non-linearly across transcript length on end-biased platforms like Chromium, so some variants are much more easily detectable than others
- Partial transcript coverage on the Chromium platform further limits sensitivity of variant detection
- Utility depends on the specific mutational composition of the sample and will likely perform better for tumor types with higher mutation burdens than AML

## Evidence

- [results] Single-cell MCFs and scVAFs were more highly correlated with eWGS VAFs (r = 0.69 and 0.68, respectively) than bulk RNA VAFs were (r = 0.52; Table 1).: [evidence span withheld — anchor status: not_found]
- [discussion] In addition, dropout reduces the sensitivity of mutation detection by a factor of two.: "In addition, dropout reduces the sensitivity of mutation detection by a factor of two."
- [intro] Detecting genetic variants in scRNA-seq reads is difficult due to the low transcript abundance, allelic dropout, and incomplete transcript coverage inherent to this platform: "Detecting genetic variants in scRNA-seq reads is difficult due to the low transcript abundance, allelic dropout, and incomplete transcript coverage inherent to this platform"
- [readme] Mutation barcode caller,  calls mutant and ref barcodes from 10x single cell data: "Mutation barcode caller,  calls mutant and ref barcodes from 10x single cell data"
- [readme] VarTrix is a software tool for extracting single cell variant information from 10x Genomics single cell data.: [evidence span withheld — anchor status: external_doc]
