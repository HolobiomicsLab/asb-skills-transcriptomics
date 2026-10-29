---
name: stereoseq-spatial-barcode-extraction
description: Use when processing a SAW-generated unmapped FASTQ from a Stereo-seq
  sample to recover expressed lineage barcodes per spatial spot and map the spatial
  territories of abundant clones.
when_to_use_negative:
- Input is a standard population-level barcode amplicon FASTQ rather than spatial
  data — use the BARtab bulk workflow instead
- Input FASTQ is not the SAW unmapped read set (e.g. genome-mapped BAM with no barcode
  reads) and lacks the flanking constant regions needed for cutadapt constants=all
  trimming
- Barcodes are evolving/naturally variable barcodes that need lineage reconstruction
  (e.g. CRISPR barcodes handled by Cassiopeia or LinRace) rather than fixed synthetic
  barcodes
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: BARtab
  role: Nextflow pipeline running mode single-cell with --pipeline saw to trim, align,
    and count SPLINTR barcodes per spatial spot
  repo: https://github.com/DaneVass/BARtab
- name: bartools
  role: R package for downstream per-spot barcode filtering, spatial territory mapping,
    and clone-overlap analysis with matched datasets
  repo: https://github.com/DaneVass/bartools
- name: Nextflow
  role: Execution engine for the BARtab single-cell workflow across computational
    environments
  repo: https://github.com/DaneVass/BARtab
- name: R
  role: Language used with bartools for overlap of top spatial barcodes against bulk
    (GSM4912416) and single-cell (GSM4912482) annotations
  repo: https://github.com/DaneVass/bartools
- name: SAW
  role: Upstream Stereo-seq pipeline (>=v6.1.0) that produced the unmapped FASTQ consumed
    by BARtab
  repo: https://github.com/STOmics/SAW
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100763
    title: Analysis of synthetic cellular barcodes in the genome and transcriptome
      with BARtab and bartools
  source_content_hash: sha256:4ffff4c3e5256c9a6cf41ce9d1dd70bb93b2b88b30c0949bb9676205317829ac
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# stereoseq-spatial-barcode-extraction

## Summary

Extracts and annotates SPLINTR synthetic lineage barcodes from Stereo-seq spatial transcriptomics data produced by the SAW pipeline, using the BARtab Nextflow single-cell workflow to generate per-spot barcode counts and spatial clone-territory annotations.

## When to use

Use when processing a SAW-generated unmapped FASTQ from a Stereo-seq sample to recover expressed lineage barcodes per spatial spot and map the spatial territories of abundant clones.

## When NOT to use

- Input is a standard population-level barcode amplicon FASTQ rather than spatial data — use the BARtab bulk workflow instead
- Input FASTQ is not the SAW unmapped read set (e.g. genome-mapped BAM with no barcode reads) and lacks the flanking constant regions needed for cutadapt constants=all trimming
- Barcodes are evolving/naturally variable barcodes that need lineage reconstruction (e.g. CRISPR barcodes handled by Cassiopeia or LinRace) rather than fixed synthetic barcodes

## Inputs

- SAW-generated unmapped FASTQ file (one per Stereo-seq sample, produced with outUnMappedFq=1)
- SPLINTR barcode reference FASTA (e.g. SPLINTR_mCHERRY_V2_barcode_reference_library.fasta)
- Stereo-seq spatial binning/coordinate information matching the sample
- Matched bulk barcode counts table and single-cell barcode annotation for clone-overlap validation

## Outputs

- counts/<sample_id>.counts.tsv — lineage barcode counts per spatial spot (from COUNT_BARCODES_SAM)
- counts/<sample_id>_cell_barcode_annotation.tsv — per-spot dominant-barcode annotation after PARSE_BARCODES_SC filtering
- QC plots of barcodes per spot
- Binned spatial territory maps of top lineage barcodes and clone-overlap statistics against bulk/single-cell datasets

## How to apply

Download the SAW unmapped FASTQ (produced with outUnMappedFq=1, e.g. from Zenodo 10685805) and run the BARtab single-cell workflow via Nextflow with mode single-cell and --pipeline saw. Trim flanking constant regions with cutadapt using constants=all (maximum recovery from randomly fragmented reads), then align trimmed reads to the SPLINTR barcode reference FASTA (e.g. SPLINTR_mCHERRY_V2_barcode_reference_library.fasta) with bowtie1 using alnmismatches=2 and flags -a --best --strata -m1 --norc. Count barcodes per spatial spot from the SAM file with the COUNT_BARCODES_SAM module (no PCR chimera removal or UMI error correction) to produce counts/<sample_id>.counts.tsv, then aggregate and filter with PARSE_BARCODES_SC using umi_count_filter=1 and umi_fraction_filter=0.3 to generate the per-spot annotation table counts/<sample_id>_cell_barcode_annotation.tsv with QC plots of barcodes per spot. Finally bin spot-level counts the same way as the Stereo-seq data and map spatial territories of the top lineage barcodes; overlap them against matched bulk and single-cell barcode data with bartools/R.

## Related tools

- **BARtab** (Nextflow pipeline running mode single-cell with --pipeline saw to trim, align, and count SPLINTR barcodes per spatial spot) — https://github.com/DaneVass/BARtab
- **bartools** (R package for downstream per-spot barcode filtering, spatial territory mapping, and clone-overlap analysis with matched datasets) — https://github.com/DaneVass/bartools
- **Nextflow** (Execution engine for the BARtab single-cell workflow across computational environments) — https://github.com/DaneVass/BARtab
- **R** (Language used with bartools for overlap of top spatial barcodes against bulk (GSM4912416) and single-cell (GSM4912482) annotations) — https://github.com/DaneVass/bartools
- **SAW** (Upstream Stereo-seq pipeline (>=v6.1.0) that produced the unmapped FASTQ consumed by BARtab) — https://github.com/STOmics/SAW

## Examples

```
nextflow run DaneVass/bartab --mode single-cell --pipeline saw --input sample_unmapped.fastq.gz --ref test/ref/SPLINTR_mCHERRY_V2_barcode_reference_library.fasta
```

## Evaluation signals

- counts/<sample_id>.counts.tsv exists and contains per-spot barcode counts derived from the SAM alignments
- PARSE_BARCODES_SC filtering applied with umi_count_filter=1 and umi_fraction_filter=0.3; annotation table columns match barcode/spot schema
- QC plots show a plausible distribution of barcodes per spot after filtering (no empty annotation table)
- Validation threshold: overlap analysis reports 10/10 of the top-10 spatial clones present in the matched population-level dataset (GSM4912416) and 9/10 in the matched single-cell dataset (GSM4912482)
- Mapped spatial territories of top barcodes form coherent binned regions consistent with the Stereo-seq binning scheme

## Limitations

- COUNT_BARCODES_SAM applies no PCR chimera removal or UMI error correction, so per-spot counts may contain chimeric/error-derived barcodes that survive only via the 0.3 umi_fraction_filter
- No accepted gold-standard dataset with known barcode abundance exists, so benchmarking against other pre-processing tools was difficult and optimal parameter selection (e.g. no simulation-based benchmarking) remains untested
- BARtab and bartools currently do not officially support evolving barcode methods, limiting reuse beyond fixed synthetic barcodes

## Evidence

- [abstract] simplify the extraction, quality control, analysis, and visualization of lineage barcodes from population-level, single-cell, and spatial transcriptomics: "BARtab and bartools contain methods to simplify the extraction, quality control, analysis, and visualization of lineage barcodes from population-level, single-cell, and spatial transcriptomics"
- [methods] processed with the SAW pipeline (>=v6.1.0): "processed with the [SAW pipeline](https://github.com/STOmics/SAW) (>=v6.1.0)"
- [methods] Barcodes can be filtered based on the number of supporting UMIs (--umi_count_filter): "Barcodes can be filtered based on the number of supporting UMIs (`--umi_count_filter`)"
- [methods] Alignments that start at the first position or end at the last are retained.: "Alignments that start at the first position or end at the last are retained."
- [readme] Run any of the test datasets using nextflow run DaneVass/bartab -profile test_sc_saw_fastq: "Run any of the test datasets using `nextflow run DaneVass/bartab -profile <test_SE,test_PE,test_SE_ref_free,test_sc,test_sc_bam,test_sc_saw_fastq>,<conda,docker,singularity>,<slurm,lsf>`"
- [discussion] benchmarking BARtab against similar tools was difficult due to the lack of a gold-standard dataset with known barcode abundance: "benchmarking BARtab against similar tools was difficult due to the lack of a gold-standard dataset with known barcode abundance"
