---
name: per-cell-genotype-matrix-quantification
description: Use when 10x Chromium single-cell BAMs aligned with cellranger must be
  genotyped at a pre-called somatic variant set (e.g. from eWGS tumor/normal) to assign
  variant evidence to individual cell barcodes.
when_to_use_negative:
- VarTrix does not perform variant calling — do not use it to discover novel variants
  from scRNA-seq; input must be a pre-called variant set (e.g. from eWGS)
- Multi-allelic sites are ignored by VarTrix (rows retained but values empty), so
  this skill is wrong for multi-allelic loci
- Not appropriate for detecting CNAs — use copyCat on paired tumor/normal coverage
  instead
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0199
tools:
- name: cb_sniffer
  role: Mutation barcode caller that parses CB barcodes from 10x single-cell BAMs
    and calls mutant vs reference barcodes per cell
  repo: https://github.com/sridnona/cb_sniffer
- name: vartrix
  role: Extracts single-cell variant information via Smith-Waterman alignment and
    outputs a Market Matrix genome matrix of variant-by-cell assignments
  repo: https://github.com/10XGenomics/vartrix
- name: python3 (sridnona/python3:180925.v1 Docker image)
  role: Environment for running cb_sniffer and computing scVAF/MCF/MCDR metrics and
    Pearson correlations
- name: copyCat
  role: R package for somatic CNA detection from paired tumor/normal coverage; complementary,
    not used for per-cell SNV genotyping
  repo: https://github.com/chrisamiller/copyCat
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-11591-1
    title: A general approach for detecting expressed mutations in AML cells using
      single cell RNA-sequencing
  source_content_hash: sha256:6ec59eb4efec9174b0fb6c3315b70d1efb6f261187af089268dbf93a851da9d5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sridnona/cb_sniffer
  tools:
  - Docker
  - cb_sniffer
  - copyCat
  - python3
  - python3 (sridnona/python3:180925.v1 Docker image)
  - sridnona/python3:180925.v1
  - vartrix
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# per-cell-genotype-matrix-quantification

## Summary

Quantify somatic SNV evidence per cell in 10x Genomics Chromium scRNA-seq data by parsing CB-tagged barcodes and UMI/read counts, producing a per-cell genotype matrix from which scVAF, Mutant Cell Fraction (MCF), and Mutant Cell Detection Rate can be computed against eWGS and bulk RNA-seq VAFs.

## When to use

Use when 10x Chromium single-cell BAMs aligned with cellranger must be genotyped at a pre-called somatic variant set (e.g. from eWGS tumor/normal) to assign variant evidence to individual cell barcodes.

## When NOT to use

- VarTrix does not perform variant calling — do not use it to discover novel variants from scRNA-seq; input must be a pre-called variant set (e.g. from eWGS)
- Multi-allelic sites are ignored by VarTrix (rows retained but values empty), so this skill is wrong for multi-allelic loci
- Not appropriate for detecting CNAs — use copyCat on paired tumor/normal coverage instead

## Inputs

- 10x cellranger-aligned BAM with CB tags and index
- Variant file with header (chrm, start, stop, ref, var, gene_name, trv_type) of pre-called somatic SNVs from eWGS tumor/normal
- Cell barcodes file (list of good barcodes from Cell Ranger)
- Sample/upn name used as output file prefix

## Outputs

- Per-cell genotype matrix assigning variants to individual cell barcodes
- UB_tag read counts table (_counts_UB.tsv)
- Per-variant metrics (scVAF, MCF, MCDR) and Pearson correlation coefficients, e.g. scvaf_correlations.tsv

## How to apply

Start from a cellranger-produced 10x BAM (with CB tags), a variant file with header (chrm, start, stop, ref, var, gene_name, trv_type), and a list of good barcodes. Run cb_sniffer to parse CB barcodes and call mutant vs reference barcodes, applying read-per-barcode (-f), mapping quality (-mq, default 0), and base quality (-bq, default 1) filters; alternatively use VarTrix, which uses Smith-Waterman alignment to assign cells to input variants and outputs a Cell Ranger-style sparse Market Matrix genome matrix. From the resulting per-cell genotype matrix, compute per-variant scVAF (mutant reads over total informative reads), MCF (fraction of cells carrying the variant), and Mutant Cell Detection Rate (fraction of informative cells detecting the variant). Merge these with eWGS and bulk RNA-seq VAFs by variant identifier and validate with Pearson correlations (MCF vs eWGS r ≈ 0.69, scVAF vs eWGS r ≈ 0.68, bulk RNA vs eWGS r = 0.52, scVAF vs bulk RNA r = 0.34).

## Related tools

- **cb_sniffer** (Mutation barcode caller that parses CB barcodes from 10x single-cell BAMs and calls mutant vs reference barcodes per cell) — https://github.com/sridnona/cb_sniffer
- **vartrix** (Extracts single-cell variant information via Smith-Waterman alignment and outputs a Market Matrix genome matrix of variant-by-cell assignments) — https://github.com/10XGenomics/vartrix
- **python3 (sridnona/python3:180925.v1 Docker image)** (Environment for running cb_sniffer and computing scVAF/MCF/MCDR metrics and Pearson correlations)
- **copyCat** (R package for somatic CNA detection from paired tumor/normal coverage; complementary, not used for per-cell SNV genotyping) — https://github.com/chrisamiller/copyCat

## Examples

```
python3 cb_sniffer.py bam_file variant_file barcodes upn -f 0 -mq 0 -bq 1
```

## Evaluation signals

- Pearson r ≈ 0.69 for MCF vs eWGS VAF and r ≈ 0.68 for scVAF vs eWGS VAF, within tolerance
- Pearson r = 0.52 for bulk RNA-seq VAF vs eWGS VAF and r = 0.34 for scVAF vs bulk RNA-seq VAF
- scVAF is attenuated relative to eWGS VAF, consistent with dropout reducing mutation-detection sensitivity by a factor of two
- Every variant in the input variant file appears as a row in the output matrix (multi-allelic sites retained with empty values per VarTrix behavior)
- Output matrix column labels match the Cell Ranger cell barcodes input file

## Limitations

- Low transcript abundance, allelic dropout, and incomplete transcript coverage make variant detection in scRNA-seq reads difficult; dropout reduces the sensitivity of mutation detection by a factor of two
- Partial transcript coverage is specific to end-biased platforms such as Chromium and limits the sensitivity of variant detection; coverage drops non-linearly across transcript length, so some variants are much more easily detectable than others
- As a result of dropout, it is impossible to determine whether a cell is truly wild-type for a given mutation
- Noise inherent in reverse transcription leads to a high false-positive rate when calling variants directly from RNA-seq
- Utility depends on the mutational composition of the sample; the approach will likely perform better for tumor types with higher mutation burdens than AML

## Evidence

- [readme] Mutation barcode caller, calls mutant and ref barcodes from 10x single cell data. Bams aligned using 10x cellranger pipeline: "Mutation barcode caller,  calls mutant and ref barcodes from 10x single cell data"
- [readme] VarTrix overview: [evidence span withheld — anchor status: external_doc]
- [readme] VarTrix does not perform variant calling: [evidence span withheld — anchor status: external_doc]
- [discussion] dropout sensitivity: "In addition, dropout reduces the sensitivity of mutation detection by a factor of two."
- [intro] variant detection difficulty: "Detecting genetic variants in scRNA-seq reads is difficult due to the low transcript abundance, allelic dropout, and incomplete transcript coverage inherent to this platform"
- [results] correlation finding: [evidence span withheld — anchor status: not_found]
