---
name: lineage-barcode-counting-per-cell
description: Use when processing single-cell RNA-seq data with expressed synthetic
  lineage barcodes (e.g., 10X scRNA-seq FASTQs) to produce per-cell barcode annotation
  tables after removing PCR chimeras and UMI sequencing errors.
when_to_use_negative:
- Data are population-level (bulk) barcode sequencing, where BARtab's bulk workflow
  rather than the single-cell denoising steps applies
- Barcodes are evolving/lineage-reconstruction-style targets requiring tools like
  Cassiopeia or LinRace, which BARtab does not officially support
- Input is already a deduplicated per-cell barcode count table (e.g., a counts.tsv
  from another pipeline); skip extraction and go directly to bartools analysis
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3794
- http://edamontology.org/topic_0203
tools:
- name: BARtab
  role: 'Nextflow pipeline executing the single-cell workflow: cutadapt trimming,
    bowtie1 alignment, REMOVE_PCR_CHIMERISM, and PARSE_BARCODES_SC per-cell annotation'
  repo: https://github.com/DaneVass/BARtab
- name: Nextflow
  role: Workflow engine running DaneVass/bartab in single-cell (test_sc) mode
  repo: https://github.com/DaneVass/BARtab
- name: umi-tools
  role: count_tab error-corrects and collapses UMIs within cells at Hamming distance
    umi_dist=1
  repo: https://github.com/CGATOxford/UMI-tools
- name: bartools
  role: R package for downstream analysis and visualization of the per-cell barcode
    annotation
  repo: https://github.com/DaneVass/bartools
- name: cutadapt
  role: Trims constant regions and filters reads for length and N bases prior to alignment
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100763
    title: Analysis of synthetic cellular barcodes in the genome and transcriptome
      with BARtab and bartools
  source_content_hash: sha256:9034f3f30ab2400e19481c09d402749369d982fc448b715d4a1ee53d83298616
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DaneVass/BARtab
  tools:
  - BARtab
  - Nextflow
  - R
  - bartools
  - cutadapt
  - umi-tools
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3794
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# lineage-barcode-counting-per-cell

## Summary

BARtab's single-cell workflow for generating per-cell lineage barcode counts: PCR chimera removal, UMI error-correction/collapse with umi-tools, and per-cell barcode annotation with filters. These denoising steps let BARtab annotate an equivalent number of cells at a UMI threshold of 5 where FateMap needed a 15 UMI threshold.

## When to use

Use when processing single-cell RNA-seq data with expressed synthetic lineage barcodes (e.g., 10X scRNA-seq FASTQs) to produce per-cell barcode annotation tables after removing PCR chimeras and UMI sequencing errors.

## When NOT to use

- Data are population-level (bulk) barcode sequencing, where BARtab's bulk workflow rather than the single-cell denoising steps applies
- Barcodes are evolving/lineage-reconstruction-style targets requiring tools like Cassiopeia or LinRace, which BARtab does not officially support
- Input is already a deduplicated per-cell barcode count table (e.g., a counts.tsv from another pipeline); skip extraction and go directly to bartools analysis

## Inputs

- Single-cell FASTQ files containing expressed synthetic lineage barcode reads (e.g., 10X scRNA-seq data; test_sc test dataset)
- Barcode reference library FASTA (e.g., SPLINTR_mCHERRY_V2_barcode_reference_library.fasta)
- Optionally a BAM file with cell barcodes/UMIs (test_sc_bam mode)

## Outputs

- counts/pcr_chimerism/test_sc_mapped_barcodes.tsv (per cell barcode-UMI retained lineage barcode combination, formatted for umi-tools count_tab)
- test_sc.counts.tsv with columns: barcode, cell barcode, deduplicated UMI count
- test_sc_cell_barcode_annotation.tsv (per-cell barcode annotation after umi_count_filter and umi_fraction_filter)
- counts/qc/test_sc_barcodes_per_cell.pdf QC plot of barcodes per cell

## How to apply

Run the BARtab single-cell mode of the DaneVass/bartab Nextflow pipeline (e.g., on the test_sc test dataset) with a barcode reference such as SPLINTR_mCHERRY_V2_barcode_reference_library.fasta, trimming constant regions with cutadapt and aligning trimmed reads with bowtie1. Apply REMOVE_PCR_CHIMERISM: for each cell barcode-UMI pair retain only the lineage barcode combination supported by the most reads, writing counts/pcr_chimerism/test_sc_mapped_barcodes.tsv formatted as input to umi-tools count_tab. Error-correct and collapse UMIs within cells using umi-tools count_tab with Hamming distance umi_dist set to 1 (default and recommended), producing test_sc.counts.tsv with columns barcode, cell barcode, and deduplicated UMI count. Aggregate barcodes per cell with PARSE_BARCODES_SC using umi_count_filter 1 and umi_fraction_filter 0.3 to generate test_sc_cell_barcode_annotation.tsv and QC plots of barcodes per cell. The rationale is that these two denoising steps (absent from Rewind/FateMap) remove PCR chimeras and UMI sequencing errors, improving annotation efficiency so fewer UMIs are needed per cell for confident calling.

## Related tools

- **BARtab** (Nextflow pipeline executing the single-cell workflow: cutadapt trimming, bowtie1 alignment, REMOVE_PCR_CHIMERISM, and PARSE_BARCODES_SC per-cell annotation) — https://github.com/DaneVass/BARtab
- **Nextflow** (Workflow engine running DaneVass/bartab in single-cell (test_sc) mode) — https://github.com/DaneVass/BARtab
- **umi-tools** (count_tab error-corrects and collapses UMIs within cells at Hamming distance umi_dist=1) — https://github.com/CGATOxford/UMI-tools
- **bartools** (R package for downstream analysis and visualization of the per-cell barcode annotation) — https://github.com/DaneVass/bartools
- **cutadapt** (Trims constant regions and filters reads for length and N bases prior to alignment)

## Examples

```
nextflow run DaneVass/bartab -profile test_sc,docker --ref test/ref/SPLINTR_mCHERRY_V2_barcode_reference_library.fasta
```

## Evaluation signals

- test_sc.counts.tsv exists and contains exactly the three required columns: barcode, cell barcode, and deduplicated UMI count
- The number of detected barcodes per cell substantially decreases after PCR chimera removal, as assessed in counts/qc/test_sc_barcodes_per_cell.pdf
- Per-cell annotation is produced with umi_count_filter=1 and umi_fraction_filter=0.3 yielding test_sc_cell_barcode_annotation.tsv
- The UMI count_tab deduplication runs with Hamming distance umi_dist=1 (the default and recommended setting)
- Comparable cell annotation numbers to FateMap's 15-UMI-threshold results are achievable at a UMI threshold of 5 due to the denoising steps

## Limitations

- Benchmarking against similar tools was difficult due to the lack of a gold-standard dataset with known barcode abundance and differing parameter flexibility between tools
- Benchmarking on simulated data (for accuracy and optimal parameter selection) was not performed
- BARtab and bartools currently do not officially support evolving barcode methods

## Evidence

- [methods] PCR chimera removal retains dominant lineage barcode per cell barcode-UMI pair: "Reads from PCR chimerism are removed by only retaining the cell barcode-umi-lineage barcode combination supported by the most reads."
- [methods] UMI filtering parameters for per-cell annotation: "Barcodes can be filtered based on the number of supporting UMIs (`--umi_count_filter`)"
- [methods] Barcode clustering distance is conservative: "The default Levenshtein distance for clustering is 3 which is conservative to avoid collapsing truly distingt barcodes."
- [discussion] BARtab benchmarked by reproducing published results: "We benchmarked BARtab by reproducing published results, showing improved runtime and sensitivity."
- [readme] BARtab benchmark comparison code location: [evidence span withheld — anchor status: external_doc]
