---
name: umi-error-correction-and-collapsing
description: Use when processing single-cell amplicon sequencing data for expressed
  lineage barcodes and you must remove PCR chimeras and collapse UMI sequencing errors
  before per-cell barcode annotation.
when_to_use_negative:
- Population-level (bulk) barcode data with no cell barcodes or UMIs — REMOVE_PCR_CHIMERISM
  and per-cell UMI collapsing are single-cell-specific steps
- Input is already a deduplicated barcode-by-cell counts table — re-collapsing UMIs
  would double-correct and distort counts
- Barcode protocols without UMI sequences, where there is no molecular identifier
  to collapse
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: BARtab
  role: Nextflow pipeline whose single-cell mode implements REMOVE_PCR_CHIMERISM and
    PARSE_BARCODES_SC around the umi-tools collapsing step
  repo: https://github.com/DaneVass/BARtab
- name: umi-tools
  role: Performs UMI error correction and collapsing via count_tab with umi_dist (Hamming
    distance) set to 1
  repo: https://github.com/CGATOxford/UMI-tools
- name: Nextflow
  role: Executes the BARtab single-cell workflow end-to-end
  repo: https://github.com/DaneVass/BARtab
- name: R
  role: Used with bartools for downstream per-cell barcode annotation and clone-size
    comparison
  repo: https://github.com/DaneVass/bartools
provenance:
  source_task_ids:
  - task_007
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100763
    title: Analysis of synthetic cellular barcodes in the genome and transcriptome
      with BARtab and bartools
  source_content_hash: sha256:45eab70b6b9b7bd9c552abf1ebb09dbb8496374b46bdbd09be70edae66490cac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DaneVass/BARtab
  tools:
  - BARtab
  - FastQC
  - Nextflow
  - R
  - bowtie1
  - cutadapt
  - fastp
  - samtools
  - starcode
  - umi-tools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# umi-error-correction-and-collapsing

## Summary

Denoising steps in the BARtab single-cell workflow that remove PCR chimeras (REMOVE_PCR_CHIMERISM) and error-correct/collapse UMIs with umi-tools count_tab at Hamming distance 1, improving lineage barcode annotation efficiency so that BARtab annotates an equivalent number of cells at a UMI threshold of 5 where FateMap/Rewind required 15.

## When to use

Use when processing single-cell amplicon sequencing data for expressed lineage barcodes and you must remove PCR chimeras and collapse UMI sequencing errors before per-cell barcode annotation.

## When NOT to use

- Population-level (bulk) barcode data with no cell barcodes or UMIs — REMOVE_PCR_CHIMERISM and per-cell UMI collapsing are single-cell-specific steps
- Input is already a deduplicated barcode-by-cell counts table — re-collapsing UMIs would double-correct and distort counts
- Barcode protocols without UMI sequences, where there is no molecular identifier to collapse

## Inputs

- Aligned, filtered lineage barcode alignments per cell (e.g. SAM/TSV from bowtie1 with cell barcode and UMI extracted from R1)
- Cell barcode whitelists (<sample_id>_whitelist.tsv) derived from GEO-supplied whitelists
- Lineage barcode reference library (e.g. SPLINTR_mCHERRY_V2_barcode_reference_library.fasta)

## Outputs

- counts/pcr_chimerism/<sample>_mapped_barcodes.tsv (most-supported cell barcode-UMI-lineage barcode combinations, formatted for umi-tools count_tab)
- Deduplicated counts table (e.g. test_sc.counts.tsv) with columns: barcode, cell barcode, deduplicated UMI count
- Cell barcode annotation table (e.g. test_sc_cell_barcode_annotation.tsv) plus QC plots of barcodes per cell

## How to apply

After alignment and FILTER_ALIGNMENTS, run REMOVE_PCR_CHIMERISM: for each cell barcode-UMI pair, retain only the lineage barcode combination supported by the most reads, writing counts/pcr_chimerism/<sample>_mapped_barcodes.tsv formatted as input to umi-tools count_tab. Then run umi-tools count_tab on that file with the Hamming distance parameter umi_dist set to 1 (the default and recommended setting) to error-correct and deduplicate UMIs within cells, producing a table with barcode, cell barcode, and deduplicated UMI count columns. Follow with PARSE_BARCODES_SC, aggregating barcodes per cell using umi_count_filter 1 and umi_fraction_filter 0.3. The rationale is that PCR chimerism inflates the number of distinct barcode combinations per cell, and UMI errors inflate apparent barcode diversity; removing both lets equivalent cells be called at a much lower UMI threshold than pipelines lacking these steps.

## Related tools

- **BARtab** (Nextflow pipeline whose single-cell mode implements REMOVE_PCR_CHIMERISM and PARSE_BARCODES_SC around the umi-tools collapsing step) — https://github.com/DaneVass/BARtab
- **umi-tools** (Performs UMI error correction and collapsing via count_tab with umi_dist (Hamming distance) set to 1) — https://github.com/CGATOxford/UMI-tools
- **Nextflow** (Executes the BARtab single-cell workflow end-to-end) — https://github.com/DaneVass/BARtab
- **R** (Used with bartools for downstream per-cell barcode annotation and clone-size comparison) — https://github.com/DaneVass/bartools

## Examples

```
umi-tools count_tab counts/pcr_chimerism/test_sc_mapped_barcodes.tsv --umi_dist=1 > test_sc.counts.tsv
```

## Evaluation signals

- The deduplicated counts file (e.g. test_sc.counts.tsv) exists and contains exactly three columns: barcode, cell barcode, and deduplicated UMI count
- The number of detected barcodes per cell substantially decreases after PCR chimera removal, as assessed in the counts/qc/<sample>_barcodes_per_cell.pdf QC plot
- On FateMap FM0-2 data, BARtab with these denoising steps annotates an equivalent number of cells at a UMI threshold of 5 where FateMap required a 15 UMI threshold
- Per-cell barcode annotations reproduce the published FateMap results with 99.6% overlap (12 BARtab-only, 145 FateMap-only, 39,884 shared cells)
- Pearson clone-size correlations against FateMap annotations fall between 0.965 and 0.998

## Limitations

- umi_dist of 1 is only the default and recommended Hamming distance; changing it may collapse genuinely distinct UMIs or fail to correct sequencing errors
- The denoising steps assume the cell barcode-UMI-lineage barcode combination with the most reads is the true one, which may mislabel when chimeric products dominate a cell-UMI pair
- These single-cell denoising steps are absent from Rewind/FateMap, so direct threshold comparisons between pipelines are confounded unless denoising is accounted for

## Evidence

- [methods] PCR chimera removal retains the most-supported combination: "Reads from PCR chimerism are removed by only retaining the cell barcode-umi-lineage barcode combination supported by the most reads."
- [methods] UMI-based barcode filtering parameters: "Barcodes can be filtered based on the number of supporting UMIs (`--umi_count_filter`)"
- [methods] UMI collapsing tool from starcode: "the python script `starcode-umi`, from [starcode/starcode-umi](https://github.com/gui11aume/starcode/blob/master/starcode-umi)"
- [readme] Pipeline purpose from README: "A Nextflow pipeline to tabulate synthetic barcode counts from NGS data"
