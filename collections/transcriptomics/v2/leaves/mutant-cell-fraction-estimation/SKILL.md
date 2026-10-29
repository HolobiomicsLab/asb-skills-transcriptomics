---
name: mutant-cell-fraction-estimation
description: Use when a per-cell somatic SNV genotype matrix from 10x scRNA-seq data
  must be summarized into per-variant mutation frequencies and benchmarked against
  bulk VAFs (eWGS or RNA-seq) to assess single-cell detection sensitivity.
when_to_use_negative:
- When no eWGS or bulk RNA-seq VAF calls exist for the same variants, since the correlation-based
  validation cannot be performed
- When the goal is de novo variant calling from scRNA-seq data — VarTrix and cb_sniffer
  require pre-called variant sets and noise from reverse transcription makes RNA-seq
  variant calling error-prone
- When variants lie outside regions covered by end-biased 10x Chromium transcripts,
  since coverage drops non-linearly across transcript length and detection is unreliable
  there
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: python3
  role: Computes per-variant scVAF, MCF, and MCDR from the genotype matrix and calculates
    Pearson correlations against eWGS and bulk RNA VAFs
  repo: https://github.com/genome/scrna_mutations
- name: sridnona/python3:180925.v1
  role: Docker image providing the python3 execution environment for the correlation
    analysis
- name: cb_sniffer
  role: Upstream cell genotyping tool whose mutant/ref barcode calls supply the per-cell
    genotype matrix used to derive MCF and scVAF
  repo: https://github.com/sridnona/cb_sniffer
- name: VarTrix
  role: Alternative upstream tool that assigns single cells to previously defined
    variant loci, producing the genome matrix from which cell-based metrics are computed
  repo: https://github.com/10XGenomics/vartrix
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-11591-1
    title: A general approach for detecting expressed mutations in AML cells using
      single cell RNA-sequencing
  source_content_hash: sha256:e40efbd54d8598b3fd7e04cc87fdc6539eff7098e708cc7cf7bfbe7bf4da69f3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/genome/scrna_mutations
  tools:
  - Docker
  - VarTrix
  - cb_sniffer
  - python3
  - sridnona/python3:180925.v1
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# mutant-cell-fraction-estimation

## Summary

Compute read-based and cell-based mutation detection metrics — single-cell VAF (scVAF), Mutant Cell Fraction (MCF), and Mutant Cell Detection Rate (MCDR) — from a per-cell scRNA-seq genotype matrix and validate them by Pearson correlation against eWGS and bulk RNA-seq variant allele fractions.

## When to use

Use when a per-cell somatic SNV genotype matrix from 10x scRNA-seq data must be summarized into per-variant mutation frequencies and benchmarked against bulk VAFs (eWGS or RNA-seq) to assess single-cell detection sensitivity.

## When NOT to use

- When no eWGS or bulk RNA-seq VAF calls exist for the same variants, since the correlation-based validation cannot be performed
- When the goal is de novo variant calling from scRNA-seq data — VarTrix and cb_sniffer require pre-called variant sets and noise from reverse transcription makes RNA-seq variant calling error-prone
- When variants lie outside regions covered by end-biased 10x Chromium transcripts, since coverage drops non-linearly across transcript length and detection is unreliable there

## Inputs

- Per-cell genotype matrix for somatic SNVs (cell barcode x variant genotype calls from cb_sniffer or VarTrix)
- eWGS variant allele fractions per somatic SNV
- Bulk RNA-seq variant allele fractions per somatic SNV
- Variant list with identifiers (chrm, start, stop, ref, var, gene_name, trv_type)

## Outputs

- Per-variant table of scVAF, Mutant Cell Fraction, and Mutant Cell Detection Rate
- Pearson correlation coefficients (scvaf_correlations.tsv)

## How to apply

Load the per-cell genotype matrix (produced by cell genotyping with cb_sniffer or VarTrix) using python3 in the sridnona/python3:180925.v1 Docker image. For each somatic SNV, compute scVAF as mutant reads over total informative reads across cells, MCF as the fraction of cells carrying the variant, and MCDR as the fraction of cells with informative coverage in which the variant is detected. Merge these per-variant metrics with eWGS VAFs and bulk RNA-seq VAFs by variant identifier, then compute Pearson correlations for scVAF vs eWGS, MCF vs eWGS, scVAF vs bulk RNA VAF, and bulk RNA VAF vs eWGS. Expect scVAF and MCF to correlate better with eWGS VAFs (r = 0.68 and 0.69) than bulk RNA VAFs do (r = 0.52), and scVAF to be attenuated relative to eWGS VAF because dropout halves mutation-detection sensitivity. Write per-variant metrics and correlation coefficients to scvaf_correlations.tsv.

## Related tools

- **python3** (Computes per-variant scVAF, MCF, and MCDR from the genotype matrix and calculates Pearson correlations against eWGS and bulk RNA VAFs) — https://github.com/genome/scrna_mutations
- **sridnona/python3:180925.v1** (Docker image providing the python3 execution environment for the correlation analysis)
- **cb_sniffer** (Upstream cell genotyping tool whose mutant/ref barcode calls supply the per-cell genotype matrix used to derive MCF and scVAF) — https://github.com/sridnona/cb_sniffer
- **VarTrix** (Alternative upstream tool that assigns single cells to previously defined variant loci, producing the genome matrix from which cell-based metrics are computed) — https://github.com/10XGenomics/vartrix

## Examples

```
python3 compute_scvaf_correlations.py --genotype-matrix per_cell_genotypes.tsv --ewgs-vaf ewgs_vafs.tsv --bulk-rna-vaf bulk_rna_vafs.tsv --out scvaf_correlations.tsv
```

## Evaluation signals

- Pearson r for MCF vs eWGS VAF ≈ 0.69 and scVAF vs eWGS VAF ≈ 0.68, within tolerance of reported Table 1 values
- Pearson r for bulk RNA-seq VAF vs eWGS VAF = 0.52 and scVAF vs bulk RNA-seq VAF = 0.34
- scVAF values are attenuated (lower) relative to eWGS VAFs, consistent with dropout halving mutation-detection sensitivity
- Output file scvaf_correlations.tsv contains per-variant metrics plus all four correlation coefficients
- Every variant in the output carries informative-coverage denominators greater than zero for MCDR calculation

## Limitations

- Dropout (transcript and allelic) reduces the sensitivity of mutation detection by a factor of two, attenuating single-cell metrics relative to bulk VAFs
- Partial transcript coverage on end-biased platforms such as Chromium limits variant detection, and coverage drops non-linearly across transcript length so some variants are more easily detectable than others
- It is impossible to determine whether a cell is truly wild-type for a given mutation, so MCF is a lower bound on true mutant cell fraction

## Evidence

- [discussion] scVAF is attenuated relative to eWGS VAF given that dropout reduces the sensitivity of mutation detection by a factor of two: "In addition, dropout reduces the sensitivity of mutation detection by a factor of two."
- [results] r ≈ 0.69 (MCF vs eWGS), r ≈ 0.68 (scVAF vs eWGS), r = 0.52 (bulk RNA-seq vs eWGS), and r = 0.34 (scVAF vs bulk RNA-seq): [evidence span withheld — anchor status: not_found]
- [methods] python3 environment used for computing the metrics: "Docker image
----------
* sridnona/python3:180925.v1"
- [readme] cb_sniffer supplies per-cell mutant/ref barcode calls feeding the genotype matrix: "Mutation barcode caller,  calls mutant and ref barcodes from 10x single cell data"
