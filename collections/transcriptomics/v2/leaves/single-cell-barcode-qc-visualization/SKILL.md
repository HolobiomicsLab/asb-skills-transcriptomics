---
name: single-cell-barcode-qc-visualization
description: Use when processing single-cell (10x-style) FASTQs containing expressed
  synthetic lineage barcodes and you need chimeras removed, UMIs error-corrected,
  cells annotated with barcodes, and QC plots of barcodes per cell.
when_to_use_negative:
- Input is population-level (bulk) barcode sequencing data — BARtab provides a separate
  bulk workflow for that
- Input is spatial transcriptomics data, which is handled by the dedicated BARtab
  spatial mode (e.g. SAW-processed data)
- Barcodes are sequenced from the genome (genomic barcode capture) rather than expressed
  barcodes in scRNA-seq libraries
edam_operation: http://edamontology.org/operation_0284
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: BARtab
  role: 'Nextflow pipeline providing the single-cell workflow: chimera removal (REMOVE_PCR_CHIMERISM)
    and per-cell barcode aggregation (PARSE_BARCODES_SC) with QC plots'
  repo: https://github.com/DaneVass/BARtab
- name: umi-tools
  role: count_tab is used to error-correct and collapse UMIs within cells (Hamming
    distance umi_dist=1) to produce the counts table
  repo: https://github.com/CGATOxford/UMI-tools
- name: cutadapt
  role: Trims constant regions and filters reads for length and N bases before alignment
  repo: https://cutadapt.readthedocs.io/en/stable/
- name: bowtie1
  role: Aligns trimmed reads against the indexed barcode reference library
  repo: http://bowtie-bio.sourceforge.net/index.shtml
- name: bartools
  role: R package for downstream per-cell barcode annotation, filtering, and visualization
    of single-cell barcoding results
  repo: https://github.com/DaneVass/bartools
- name: Nextflow
  role: Workflow engine that runs the DaneVass/bartab pipeline with test/conda/docker/singularity
    profiles
  repo: https://github.com/DaneVass/BARtab
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100763
    title: Analysis of synthetic cellular barcodes in the genome and transcriptome
      with BARtab and bartools
  source_content_hash: sha256:4baa2f74b2415c01ae26653e40c524bb08143ef02314119b05aeb0ddfeee75fa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DaneVass/BARtab
  tools:
  - BARtab
  - Nextflow
  - R
  - bartools
  - bowtie1
  - cutadapt
  - umi-tools
  edam_operation: http://edamontology.org/operation_0284
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# single-cell-barcode-qc-visualization

## Summary

BARtab single-cell workflow for cleaning and annotating expressed lineage barcodes from single-cell RNA-seq data: PCR chimera removal, UMI error-correction/collapse, per-cell barcode aggregation with UMI-based filtering, and QC visualization of barcodes per cell.

## When to use

Use when processing single-cell (10x-style) FASTQs containing expressed synthetic lineage barcodes and you need chimeras removed, UMIs error-corrected, cells annotated with barcodes, and QC plots of barcodes per cell.

## When NOT to use

- Input is population-level (bulk) barcode sequencing data — BARtab provides a separate bulk workflow for that
- Input is spatial transcriptomics data, which is handled by the dedicated BARtab spatial mode (e.g. SAW-processed data)
- Barcodes are sequenced from the genome (genomic barcode capture) rather than expressed barcodes in scRNA-seq libraries

## Inputs

- Single-cell single-end FASTQ files containing expressed lineage barcode reads (e.g. test_sc test dataset)
- Barcode reference library FASTA (e.g. SPLINTR_mCHERRY_V2_barcode_reference_library.fasta)
- test_sc_mapped_barcodes.tsv (mapped barcode TSV formatted as input to umi-tools count_tab)

## Outputs

- counts/pcr_chimerism/test_sc_mapped_barcodes.tsv (chimera-filtered mapped barcodes for umi-tools count_tab)
- test_sc.counts.tsv with columns: barcode, cell barcode, deduplicated UMI count
- test_sc_cell_barcode_annotation.tsv (per-cell barcode annotations)
- counts/qc/test_sc_barcodes_per_cell.pdf (QC plot of barcodes per cell)

## How to apply

Run the BARtab single-cell mode of the DaneVass/bartab Nextflow pipeline (e.g. on the test_sc dataset with SPLINTR_mCHERRY_V2_barcode_reference_library.fasta as reference), trimming constant regions with cutadapt and aligning with bowtie1. Apply REMOVE_PCR_CHIMERISM: for each cell barcode-UMI pair retain only the lineage barcode combination supported by the most reads, writing counts/pcr_chimerism/test_sc_mapped_barcodes.tsv formatted for umi-tools count_tab. Error-correct and collapse UMIs within cells with umi-tools count_tab using Hamming distance umi_dist=1 (default and recommended) to yield test_sc.counts.tsv with barcode, cell barcode and deduplicated UMI count columns. Aggregate barcodes per cell with PARSE_BARCODES_SC using umi_count_filter 1 and umi_fraction_filter 0.3 to produce test_sc_cell_barcode_annotation.tsv and QC plots. The rationale is that these two denoising steps (absent from Rewind/FateMap) let BARtab annotate an equivalent number of cells at a UMI threshold of 5, whereas the FateMap analysis required a 15 UMI threshold.

## Related tools

- **BARtab** (Nextflow pipeline providing the single-cell workflow: chimera removal (REMOVE_PCR_CHIMERISM) and per-cell barcode aggregation (PARSE_BARCODES_SC) with QC plots) — https://github.com/DaneVass/BARtab
- **umi-tools** (count_tab is used to error-correct and collapse UMIs within cells (Hamming distance umi_dist=1) to produce the counts table) — https://github.com/CGATOxford/UMI-tools
- **cutadapt** (Trims constant regions and filters reads for length and N bases before alignment) — https://cutadapt.readthedocs.io/en/stable/
- **bowtie1** (Aligns trimmed reads against the indexed barcode reference library) — http://bowtie-bio.sourceforge.net/index.shtml
- **bartools** (R package for downstream per-cell barcode annotation, filtering, and visualization of single-cell barcoding results) — https://github.com/DaneVass/bartools
- **Nextflow** (Workflow engine that runs the DaneVass/bartab pipeline with test/conda/docker/singularity profiles) — https://github.com/DaneVass/BARtab

## Examples

```
nextflow run DaneVass/bartab -profile test_sc,docker
```

## Evaluation signals

- test_sc.counts.tsv exists and contains exactly three columns: barcode, cell barcode and deduplicated UMI count
- Number of detected barcodes per cell substantially decreases after PCR chimera removal, as assessed by counts/qc/test_sc_barcodes_per_cell.pdf
- PCR chimerism output file (counts/pcr_chimerism/test_sc_mapped_barcodes.tsv) is formatted as valid input to umi-tools count_tab
- Per-cell barcode aggregation applies umi_count_filter 1 and umi_fraction_filter 0.3, producing test_sc_cell_barcode_annotation.tsv
- With the denoising steps applied, the pipeline annotates an equivalent number of cells at a 5 UMI threshold that FateMap required a 15 UMI threshold to reach

## Limitations

- Benchmarking against similar tools was difficult due to lack of a gold-standard dataset with known barcode abundance and differing parameter flexibility between tools
- Benchmarking on simulated data (for accuracy and optimal parameter selection) was not performed
- Default Levenshtein clustering distance is 3, which is conservative to avoid collapsing truly distinct barcodes; parameter choices affect sensitivity

## Evidence

- [methods] REMOVE_PCR_CHIMERISM: "Reads from PCR chimerism are removed by only retaining the cell barcode-umi-lineage barcode combination supported by the most reads."
- [methods] umi_count_filter: "Barcodes can be filtered based on the number of supporting UMIs (`--umi_count_filter`)"
- [methods] umi_fraction_filter: "and by the number of UMIs in comparison to the dominant barcode per cell (`--umi_fraction_filter`)."
- [methods] CUTADAPT_READS: "Constant regions are trimmed and reads are filtered for length and N bases using [cutadapt](https://cutadapt.readthedocs.io/en/stable/)."
- [abstract] BARtab single-cell scope: "simplify the extraction, quality control, analysis, and visualization of lineage barcodes"
- [readme] BARtab README purpose: "A Nextflow pipeline to tabulate synthetic barcode counts from NGS data"
