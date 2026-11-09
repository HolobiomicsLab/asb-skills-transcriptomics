---
name: saw-pipeline-spatial-data-processing
description: Use when the input is a SAW-processed Stereo-seq spatial dataset (unmapped
  FASTQ with outUnMappedFq=1, e.g. Zenodo 10685805) and the goal is per-spot lineage
  barcode annotation and spatial clone territory mapping.
when_to_use_negative:
- Input is population-level bulk barcode sequencing data — use the BARtab bulk workflow
  instead of the single-cell/saw mode
- Data are standard 10x-style single-cell (BAM) rather than SAW/Stereo-seq spatial
  output — use the regular test_sc or test_sc_bam pipeline mode
- No barcode reference FASTA is available — the reference-based bowtie1 alignment
  step cannot run, so the reference-free mode is required instead
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3308
tools:
- name: BARtab
  role: Nextflow pipeline run in single-cell mode with --pipeline saw to trim, align,
    count, and annotate barcodes per spatial spot
  repo: github.com/DaneVass/BARtab
- name: bartools
  role: R package used with R for downstream overlap of spatial top-10 barcodes against
    population-level and single-cell datasets
  repo: github.com/DaneVass/bartools
- name: SAW
  role: STOmics pipeline (>=v6.1.0) that generated the unmapped FASTQ input consumed
    by BARtab's saw mode
  repo: github.com/STOmics/SAW
- name: Nextflow
  role: Execution framework for the portable BARtab pipeline
  repo: github.com/DaneVass/BARtab
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100763
    title: Analysis of synthetic cellular barcodes in the genome and transcriptome
      with BARtab and bartools
  source_content_hash: sha256:aae7dd173fea6628e7d77f6c28e7a4fe8a89510e10d3bb90b18e008bad901b19
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BARtab
  - Nextflow
  - R
  - SAW
  - bartools
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# saw-pipeline-spatial-data-processing

## Summary

Extract and annotate lineage barcodes from Stereo-seq spatial transcriptomics data by running the BARtab single-cell workflow in --pipeline saw mode on SAW-generated unmapped FASTQ, then per-spot counting and filtering barcodes for spatial clone mapping.

## When to use

Use when the input is a SAW-processed Stereo-seq spatial dataset (unmapped FASTQ with outUnMappedFq=1, e.g. Zenodo 10685805) and the goal is per-spot lineage barcode annotation and spatial clone territory mapping.

## When NOT to use

- Input is population-level bulk barcode sequencing data — use the BARtab bulk workflow instead of the single-cell/saw mode
- Data are standard 10x-style single-cell (BAM) rather than SAW/Stereo-seq spatial output — use the regular test_sc or test_sc_bam pipeline mode
- No barcode reference FASTA is available — the reference-based bowtie1 alignment step cannot run, so the reference-free mode is required instead

## Inputs

- SAW-generated unmapped FASTQ for a Stereo-seq spatial sample (outUnMappedFq=1)
- SPLINTR barcode reference FASTA (e.g. SPLINTR_mCHERRY_V2_barcode_reference_library.fasta)
- Matched population-level barcode counts dataset (e.g. GSM4912416)
- Matched single-cell-level barcode annotation dataset (e.g. GSM4912482)

## Outputs

- counts/<sample_id>.counts.tsv (per-spot barcode counts from SAM)
- counts/<sample_id>_cell_barcode_annotation.tsv (per-spot barcode annotation table)
- QC plots of barcodes per spot
- Spatial territory maps of top-10 lineage barcodes and overlap statistics against matched datasets

## How to apply

Download the SAW-generated unmapped FASTQ for the Stereo-seq sample (produced by SAW >=v6.1.0 mapping with outUnMappedFq=1) and run the BARtab single-cell workflow via Nextflow with mode single-cell and --pipeline saw. Trim flanking constant regions with cutadapt using constants=all, which maximizes barcode recovery from randomly fragmented spatial reads, then align trimmed reads to the SPLINTR barcode reference FASTA with bowtie1 using alnmismatches=2 and flags -a --best --strata -m1 --norc. Count barcodes per spatial spot from the SAM file with the COUNT_BARCODES_SAM module (no PCR chimera removal or UMI error correction) to produce counts/<sample_id>.counts.tsv. Aggregate and filter per-spot barcodes with PARSE_BARCODES_SC using umi_count_filter=1 and umi_fraction_filter=0.3 to generate counts/<sample_id>_cell_barcode_annotation.tsv plus QC plots of barcodes per spot. Bin spot-level counts the same way as the stereo-seq data, map the spatial territories of the top-10 lineage barcodes, and overlap them against matched population-level and single-cell barcode datasets using bartools/R.

## Related tools

- **BARtab** (Nextflow pipeline run in single-cell mode with --pipeline saw to trim, align, count, and annotate barcodes per spatial spot) — github.com/DaneVass/BARtab
- **bartools** (R package used with R for downstream overlap of spatial top-10 barcodes against population-level and single-cell datasets) — github.com/DaneVass/bartools
- **SAW** (STOmics pipeline (>=v6.1.0) that generated the unmapped FASTQ input consumed by BARtab's saw mode) — github.com/STOmics/SAW
- **Nextflow** (Execution framework for the portable BARtab pipeline) — github.com/DaneVass/BARtab

## Examples

```
nextflow run DaneVass/bartab -profile test_sc_saw_fastq,docker
```

## Evaluation signals

- Overlap analysis reports 10/10 top spatial clones present in the matched population-level data (GSM4912416) and 9/10 in the matched single-cell data (GSM4912482), matching the published result
- counts/<sample_id>.counts.tsv exists with barcode counts per spatial spot derived from the SAM file
- counts/<sample_id>_cell_barcode_annotation.tsv is produced after PARSE_BARCODES_SC filtering with umi_count_filter=1 and umi_fraction_filter=0.3
- QC plots of barcodes per spot are generated and show plausible per-spot barcode recovery after constants=all trimming
- Top-10 spatial barcode territories map to coherent binned spatial regions of the spleen section

## Limitations

- Barcode counting from the SAM file omits PCR chimera removal and UMI error correction, which may inflate per-spot barcode diversity
- The spatial FASTQ reads are randomly fragmented, so trimming strategy and alignment sensitivity (alnmismatches=2) directly affect recovery; the article notes benchmarking was difficult due to the lack of a gold-standard dataset with known barcode abundance
- BARtab and bartools currently do not officially support evolving barcode methods
- Only a subset of top abundant clones overlap between spatial and matched single-cell data (9/10), so cross-platform clone recovery is not complete

## Evidence

- [methods] Run any of the test datasets using `nextflow run DaneVass/bartab -profile <test_SE,test_PE,test_SE_ref_free,test_sc,test_sc_bam,test_sc_saw_fastq>,<conda,docker,singularity>,<slurm,lsf>`: "Run any of the test datasets using `nextflow run DaneVass/bartab -profile <test_SE,test_PE,test_SE_ref_free,test_sc,test_sc_bam,test_sc_saw_fastq>,<conda,docker,singularity>,<slurm,lsf>`"
- [methods] SAW pipeline version note: "processed with the [SAW pipeline](https://github.com/STOmics/SAW) (>=v6.1.0)"
- [methods] UMI-based barcode filtering: "Barcodes can be filtered based on the number of supporting UMIs (`--umi_count_filter`)"
- [methods] Alignment filtering rule: "Alignments that start at the first position or end at the last are retained."
- [readme] Pipeline scope statement: "A Nextflow pipeline to tabulate synthetic barcode counts from NGS data"
