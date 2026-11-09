---
name: spatial-clone-territory-mapping
description: Use when an SAW-processed spatial dataset (e.g., Stereo-seq with outUnMappedFq=1)
  carries expressed synthetic lineage barcodes and per-spot clone territories must
  be annotated and overlapped with matched bulk (GEO) and single-cell barcode annotations.
when_to_use_negative:
- The dataset is population-level (bulk) barcode sequencing only — use the BARtab
  bulk/paired-bulk workflow instead of spot-level annotation.
- The spatial platform is not SAW-based (e.g., 10x Visium) or the barcode construct
  lacks known flanking constant regions and a reference FASTA, so cutadapt constants
  and bowtie1 reference alignment cannot be specified.
- Barcode reads need PCR-chimera removal or UMI error correction (e.g., starcode-umi)
  as part of the count step — the spatial SAM-counting module used here intentionally
  omits these steps.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: BARtab
  role: 'Nextflow pipeline running the single-cell --pipeline saw workflow: cutadapt
    trimming, bowtie1 alignment, COUNT_BARCODES_SAM and PARSE_BARCODES_SC for per-spot
    barcode annotation'
  repo: https://github.com/DaneVass/BARtab
- name: bartools
  role: R package used with R to overlap the top-10 spatial barcodes against matched
    bulk and single-cell barcode datasets
  repo: https://github.com/DaneVass/bartools
- name: Nextflow
  role: Execution engine for the BARtab pipeline with docker/singularity/conda profiles
    and slurm/lsf executors
  repo: https://github.com/DaneVass/BARtab
- name: SAW
  role: STOmics pipeline (>=v6.1.0) that produced the Stereo-seq unmapped FASTQ consumed
    as input
  repo: https://github.com/STOmics/SAW
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100763
    title: Analysis of synthetic cellular barcodes in the genome and transcriptome
      with BARtab and bartools
  source_content_hash: sha256:9da287028e1354019e91534d451bcc4a43a8014130f1af504bd5a342bb1a1276
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DaneVass/BARtab
  tools:
  - BARtab
  - Nextflow
  - R
  - SAW
  - bartools
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# spatial-clone-territory-mapping

## Summary

Extract SPLINTR lineage barcodes from Stereo-seq spatial transcriptomics reads (SAW unmapped FASTQ) with BARtab's single-cell workflow, assign a dominant clone to each spatial spot, and map the spatial territories of the top-10 lineage barcodes for cross-validation against matched population-level and single-cell-level datasets.

## When to use

Use when an SAW-processed spatial dataset (e.g., Stereo-seq with outUnMappedFq=1) carries expressed synthetic lineage barcodes and per-spot clone territories must be annotated and overlapped with matched bulk (GEO) and single-cell barcode annotations.

## When NOT to use

- The dataset is population-level (bulk) barcode sequencing only — use the BARtab bulk/paired-bulk workflow instead of spot-level annotation.
- The spatial platform is not SAW-based (e.g., 10x Visium) or the barcode construct lacks known flanking constant regions and a reference FASTA, so cutadapt constants and bowtie1 reference alignment cannot be specified.
- Barcode reads need PCR-chimera removal or UMI error correction (e.g., starcode-umi) as part of the count step — the spatial SAM-counting module used here intentionally omits these steps.

## Inputs

- SAW-generated unmapped FASTQ file for the Stereo-seq spleen sample (Zenodo 10685805, one FASTQ per sample produced with outUnMappedFq=1)
- SPLINTR_mCHERRY_V2_barcode_reference_library.fasta barcode reference
- Matched population-level bulk barcode counts (GSM4912416)
- Matched single-cell-level barcode annotation table (GSM4912482)
- Stereo-seq spot/binning coordinates for spatial territory mapping

## Outputs

- counts/<sample_id>.counts.tsv (barcodes counted per spatial spot from SAM alignments)
- counts/<sample_id>_cell_barcode_annotation.tsv (per-spot dominant barcode annotation after filtering)
- QC plots of barcodes per spot
- Spatial territory maps of the top-10 lineage barcodes
- Overlap table of top-10 spatial clones versus bulk and single-cell datasets

## How to apply

Run the BARtab single-cell workflow via Nextflow in mode single-cell with --pipeline saw, trimming the flanking constant regions with cutadapt using constants=all (maximum recovery from randomly fragmented spatial reads) and aligning trimmed reads to the SPLINTR barcode reference FASTA with bowtie1 (alnmismatches=2, flags -a --best --strata -m1 --norc). Count barcodes per spatial spot from the SAM file with the COUNT_BARCODES_SAM module, performing no PCR chimera removal or UMI error correction, to produce counts/<sample_id>.counts.tsv. Aggregate and filter the per-spot barcodes with PARSE_BARCODES_SC using umi_count_filter=1 and umi_fraction_filter=0.3 (a barcode is retained only if it has at least 1 supporting UMI and its UMI fraction relative to the dominant barcode in the spot passes 0.3), yielding counts/<sample_id>_cell_barcode_annotation.tsv plus QC plots of barcodes per spot. Bin spot-level counts the same way as the Stereo-seq data and map the spatial territories of the top-10 lineage barcodes. Finally, overlap the top-10 spatial barcodes against the matched bulk barcode counts (GSM4912416) and the single-cell barcode annotation (GSM4912482) using bartools/R.

## Related tools

- **BARtab** (Nextflow pipeline running the single-cell --pipeline saw workflow: cutadapt trimming, bowtie1 alignment, COUNT_BARCODES_SAM and PARSE_BARCODES_SC for per-spot barcode annotation) — https://github.com/DaneVass/BARtab
- **bartools** (R package used with R to overlap the top-10 spatial barcodes against matched bulk and single-cell barcode datasets) — https://github.com/DaneVass/bartools
- **Nextflow** (Execution engine for the BARtab pipeline with docker/singularity/conda profiles and slurm/lsf executors) — https://github.com/DaneVass/BARtab
- **SAW** (STOmics pipeline (>=v6.1.0) that produced the Stereo-seq unmapped FASTQ consumed as input) — https://github.com/STOmics/SAW

## Examples

```
nextflow run DaneVass/bartab --mode single-cell --pipeline saw -profile docker,slurm
```

## Evaluation signals

- Overlap analysis reports 10/10 of the top-10 most abundant spatial clones present in the matched population-level dataset (GSM4912416).
- Overlap analysis reports 9/10 of the top-10 spatial clones present in the matched single-cell-level dataset (GSM4912482), matching the published result.
- counts/<sample_id>.counts.tsv and counts/<sample_id>_cell_barcode_annotation.tsv are produced in the expected per-spot tabular format.
- QC plots of barcodes per spot are generated after PARSE_BARCODES_SC with umi_count_filter=1 and umi_fraction_filter=0.3.
- Spatial territories of the top-10 barcodes are binned identically to the Stereo-seq data so maps are comparable across clones.

## Limitations

- COUNT_BARCODES_SAM performs no PCR chimera removal or UMI error correction for spatial counts, so per-spot barcode calls may include chimeric or sequencing-error artifacts.
- Spatial data were randomly fragmented, so cutadapt must run with constants=all for maximum recovery; more restrictive constant-region trimming will lose reads.
- One of the top-10 spatial clones was absent from the matched single-cell dataset (9/10 recovery), showing spatial clone sets are not fully recapitulated at single-cell resolution.
- No gold-standard dataset with known barcode abundance exists, making quantitative benchmarking of the barcode extraction step difficult.

## Evidence

- [methods] Barcodes can be filtered based on the number of supporting UMIs (`--umi_count_filter`): "Barcodes can be filtered based on the number of supporting UMIs (`--umi_count_filter`)"
- [methods] UMI fraction filter: "and by the number of UMIs in comparison to the dominant barcode per cell (`--umi_fraction_filter`)."
- [methods] SAW processing of spatial data: "processed with the [SAW pipeline](https://github.com/STOmics/SAW) (>=v6.1.0)"
- [readme] BARtab Nextflow pipeline description: "A Nextflow pipeline to tabulate synthetic barcode counts from NGS data"
- [abstract] Spatial scope of BARtab/bartools: "lineage barcodes from population-level, single-cell, and spatial transcriptomics"
