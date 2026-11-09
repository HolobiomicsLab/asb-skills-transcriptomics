---
name: allele-umi-count-analysis
description: Use when allele-level UMI counts from single-cell 3' RNA-seq (Cell Ranger
  output, SNVs called with Freebayes) must be converted into per-cell genotype calls
  or two-genotype mixture inference, e.g. donor/host chimerism in transplant or mixing
  datasets.
when_to_use_negative:
- The goal is gene-expression analysis (use Seurat on the gene-barcode matrix with
  the top 1,000 most variable genes instead)
- Only cell-multiplexing/demultiplexing is needed and prior species/genotype identities
  are known via the simple SNV-fraction multiplet rule (cells with SNV fraction 0.2-0.8
  from either species counted as multiplets)
- SNVs do not meet the genotype-assignment filters (support from at least two cell
  barcodes, SNV Qual>=30, base Qual>=1) — downstream mixture fitting will be unreliable
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Cell Ranger Single-Cell Software Suite
  role: Sample demultiplexing, barcode processing and single-cell 3' gene counting
    producing the genome BAM and gene-barcode matrix from which allele-level UMI counts
    are derived
  repo: https://github.com/10XGenomics/cellranger
- name: STAR
  role: Aligns Read1 (cDNA insert) to the hg19 reference genome prior to SNV calling
- name: Freebayes 1.0.2
  role: Calls SNVs on the Cell Ranger genome BAM that are then filtered and used for
    genotype assignment
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/ncomms14049
    title: Massively parallel digital transcriptional profiling of single cells
  source_content_hash: sha256:56f2a4cdcdabf26ba06b5f2e11d9e3c87b887c9e70af490568a0b72b83201d17
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/10XGenomics/cellranger
  tools:
  - Cell Ranger Single-Cell Software Suite
  - Freebayes 1.0.2
  - STAR
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# allele-umi-count-analysis

## Summary

An SNV-based single-cell genotyping method that fits Gibbs-sampled K=1 vs K=2 mixture models to allele-level UMI counts per cell barcodes, enabling de novo donor/host chimerism assignment in single-cell transcriptome data without prior genotype knowledge.

## When to use

Use when allele-level UMI counts from single-cell 3' RNA-seq (Cell Ranger output, SNVs called with Freebayes) must be converted into per-cell genotype calls or two-genotype mixture inference, e.g. donor/host chimerism in transplant or mixing datasets.

## When NOT to use

- The goal is gene-expression analysis (use Seurat on the gene-barcode matrix with the top 1,000 most variable genes instead)
- Only cell-multiplexing/demultiplexing is needed and prior species/genotype identities are known via the simple SNV-fraction multiplet rule (cells with SNV fraction 0.2-0.8 from either species counted as multiplets)
- SNVs do not meet the genotype-assignment filters (support from at least two cell barcodes, SNV Qual>=30, base Qual>=1) — downstream mixture fitting will be unreliable

## Inputs

- Allele-level UMI counts per cell barcode for filtered donor SNVs (from Cell Ranger genome BAM, SNVs called by Freebayes 1.0.2)
- Single-cell 3' gene-barcode matrices from Cell Ranger (hg19-aligned via STAR) used to identify cell barcodes
- In silico or wet-lab mixing ratio specifications (e.g. Donor B/C PBMC mixes at 0-50% minor fraction; 50:50, 90:10, 99:1 wet mixes)

## Outputs

- Per-cell genotype or mixture (K=1/K=2) posterior assignments
- Sensitivity and positive predictive value per minor-fraction ratio (mixing_sensitivity_ppv.csv)
- Selected model per sample (K=1 vs K=2) indicating chimerism level vs below-detection

## How to apply

Start from allele-level UMI counts per cell for SNVs that passed genotype-assignment filters: support from at least two cell barcodes, SNV Qual >=30, SNV base Qual>=1 (indels ignored). For each SNV, fit both K=1 (single genotype) and K=2 (two-genome mixture) models to the allele UMI counts with a Gibbs sampler. Select the K=2 model over K=1 only when 90% of the cells have a posterior probability >75%; otherwise report the sample as below the detection level (in silico mixing determined this to be 4% of 6,000 cells). To assess sensitivity, construct in silico mixtures by pooling single-cell allele-count profiles across minor-fraction ratios (e.g. 0-50% for Donor B/C PBMC mixes) plus wet-lab mixes (e.g. 50:50, 90:10, 99:1), then compute sensitivity and positive predictive value of minor-genotype detection per ratio. Tabulate sensitivity, PPV, and selected model (K=1/K=2) per ratio (e.g. to mixing_sensitivity_ppv.csv).

## Related tools

- **Cell Ranger Single-Cell Software Suite** (Sample demultiplexing, barcode processing and single-cell 3' gene counting producing the genome BAM and gene-barcode matrix from which allele-level UMI counts are derived) — https://github.com/10XGenomics/cellranger
- **STAR** (Aligns Read1 (cDNA insert) to the hg19 reference genome prior to SNV calling)
- **Freebayes 1.0.2** (Calls SNVs on the Cell Ranger genome BAM that are then filtered and used for genotype assignment)

## Evaluation signals

- Minor-genotype detection sensitivity and PPV must exceed 95% for minor fractions down to 3% in Donor B/C in silico mixes
- The 99:1 wet mix should yield K=1 selection, consistent with the reported detection limit of 4% of 6,000 cells
- K=2 model selected only when 90% of cells have posterior probability >75%; otherwise output reports the mixture as below detection level
- All SNVs used must have support from at least two cell barcodes, SNV Qual>=30, and SNV base Qual>=1
- Output table (mixing_sensitivity_ppv.csv) contains sensitivity, PPV, and selected K model for each in silico ratio and wet mix

## Limitations

- Minor populations could not be detected below a 3% mixing ratio; selecting K=1 indicates the mixture fraction is below the detection level (4% of 6,000 cells in in silico mixing)
- Proof-of-principle chimerism study included BMMCs from only two transplant patients; larger patient cohorts and clinical validation not yet performed
- Barcodes are corrected only if posterior probability >=0.975 and UMIs are valid only if quality score >10 and not homopolymers — upstream errors propagate if these preprocessing filters are skipped

## Evidence

- [discussion] SNV-based genotype assignment filters: "Only SNVs with support from at least two cell barcodes, with a minimal SNV Qual score >=30, minimal SNV base Qual>=1 were included."
- [discussion] K=2 model selection threshold: "We applied a requirement that 90% of the cells have a posterior probability >75% to select the K=2 model over the K=1 model."
- [discussion] Detection limit: "Selecting K=1 indicates that the mixture fraction is below the level of detection of the method, which in in silico mixing experiments was determined to be 4% of 6,000 cells."
- [discussion] De novo chimerism method novelty: "To our knowledge, this is the first report of a high-throughput method to determine chimerism of immune cell populations."
- [readme] Pipeline provenance: "Cell Ranger is a set of analysis pipelines that perform sample demultiplexing, barcode processing, single cell 3' and 5' gene counting"
