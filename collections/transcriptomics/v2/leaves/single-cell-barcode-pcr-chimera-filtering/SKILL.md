---
name: single-cell-barcode-pcr-chimera-filtering
description: Use when processing single-cell lineage-barcode reads where PCR chimeras
  inflate barcodes-per-cell and UMI sequencing errors inflate UMI counts, and an accurate
  per-cell barcode annotation table is needed.
when_to_use_negative:
- Input is population-level (bulk) barcode data — REMOVE_PCR_CHIMERISM relies on cell
  barcode-UMI grouping and belongs to the single-cell workflow
- Reference-free or evolving-barcode designs where an exact-match reference library
  is unavailable and bowtie1 alignment to a fixed FASTA is not possible
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: BARtab
  role: Nextflow pipeline providing the single-cell REMOVE_PCR_CHIMERISM and PARSE_BARCODES_SC
    steps
  repo: https://github.com/DaneVass/BARtab
- name: umi-tools
  role: count_tab with umi_dist=1 error-corrects and collapses UMIs within cells after
    chimera removal
  repo: https://github.com/CGATOxford/UMI-tools
- name: Nextflow
  role: Workflow engine running the DaneVass/bartab pipeline in single-cell mode
  repo: https://github.com/DaneVass/BARtab
- name: R
  role: bartools R package used for downstream analysis/visualization of the annotated
    single-cell barcode counts
  repo: https://github.com/DaneVass/bartools
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100763
    title: Analysis of synthetic cellular barcodes in the genome and transcriptome
      with BARtab and bartools
  source_content_hash: sha256:6b62f3166c6d0a9857fec63acef9a29d05680253a7e6f9f32a790142974846fc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DaneVass/BARtab
  tools:
  - BARtab
  - Nextflow
  - R
  - umi-tools
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# single-cell-barcode-pcr-chimera-filtering

## Summary

Denoising steps in the BARtab single-cell workflow that remove PCR chimeras (retaining only the read-supported cell barcode-UMI-lineage barcode combination) and error-correct UMIs with umi-tools count_tab at Hamming distance 1, enabling cell annotation at a UMI threshold of 5 versus the 15 required by FateMap.

## When to use

Use when processing single-cell lineage-barcode reads where PCR chimeras inflate barcodes-per-cell and UMI sequencing errors inflate UMI counts, and an accurate per-cell barcode annotation table is needed.

## When NOT to use

- Input is population-level (bulk) barcode data — REMOVE_PCR_CHIMERISM relies on cell barcode-UMI grouping and belongs to the single-cell workflow
- Reference-free or evolving-barcode designs where an exact-match reference library is unavailable and bowtie1 alignment to a fixed FASTA is not possible

## Inputs

- Aligned barcode-UMI-cell barcode records (e.g. test_sc_mapped_barcodes.tsv) from cutadapt/bowtie1 processing of single-cell FASTQ data
- Reference barcode library FASTQ/FASTA (SPLINTR_mCHERRY_V2_barcode_reference_library.fasta)

## Outputs

- counts/pcr_chimerism/<sample>_mapped_barcodes.tsv formatted as input to umi-tools count_tab
- <sample>.counts.tsv with columns: barcode, cell barcode, deduplicated UMI count
- <sample>_cell_barcode_annotation.tsv and QC plots of barcodes per cell (counts/qc/<sample>_barcodes_per_cell.pdf)

## How to apply

After aligning cutadapt-trimmed reads to the barcode reference (e.g. SPLINTR_mCHERRY_V2_barcode_reference_library.fasta) with bowtie1, run the REMOVE_PCR_CHIMERISM step: for each cell barcode-UMI pair retain only the lineage barcode combination supported by the most reads, writing counts/pcr_chimerism/<sample>_mapped_barcodes.tsv formatted as input to umi-tools count_tab. Then error-correct and collapse UMIs within cells with umi-tools count_tab using umi_dist=1 (the default and recommended Hamming distance) to produce <sample>.counts.tsv with columns barcode, cell barcode, and deduplicated UMI count. Finally, aggregate barcodes per cell with PARSE_BARCODES_SC using umi_count_filter 1 and umi_fraction_filter 0.3 to generate the per-cell barcode annotation and QC plots. These steps are absent from Rewind/FateMap, which is why BARtab annotates an equivalent number of cells at a UMI threshold of 5 where FateMap required a threshold of 15.

## Related tools

- **BARtab** (Nextflow pipeline providing the single-cell REMOVE_PCR_CHIMERISM and PARSE_BARCODES_SC steps) — https://github.com/DaneVass/BARtab
- **umi-tools** (count_tab with umi_dist=1 error-corrects and collapses UMIs within cells after chimera removal) — https://github.com/CGATOxford/UMI-tools
- **Nextflow** (Workflow engine running the DaneVass/bartab pipeline in single-cell mode) — https://github.com/DaneVass/BARtab
- **R** (bartools R package used for downstream analysis/visualization of the annotated single-cell barcode counts) — https://github.com/DaneVass/bartools

## Examples

```
nextflow run DaneVass/bartab -profile test_sc,docker
```

## Evaluation signals

- <sample>.counts.tsv exists and contains exactly the three required columns: barcode, cell barcode, deduplicated UMI count
- The number of detected barcodes per cell substantially decreases after PCR chimera removal, as assessed by the counts/qc/<sample>_barcodes_per_cell.pdf plots
- The intermediate counts/pcr_chimerism/<sample>_mapped_barcodes.tsv is formatted correctly as input to umi-tools count_tab
- Per-cell annotation succeeds with umi_count_filter 1 and umi_fraction_filter 0.3, and equivalent cell counts are achieved at a UMI threshold of 5 (vs. FateMap's required threshold of 15)

## Limitations

- These denoising steps are absent from Rewind/FateMap, so direct comparisons of cell-annotation efficiency must account for the differing UMI thresholds (5 vs. 15)
- Benchmarking BARtab against similar tools was difficult due to the lack of a gold-standard dataset with known barcode abundance and differing parameter flexibility between tools
- Benchmarking on simulated data (for accuracy and optimal parameter selection) was not performed, so optimal thresholds may vary per dataset

## Evidence

- [methods] PCR chimera removal retains read-supported combination: "Reads from PCR chimerism are removed by only retaining the cell barcode-umi-lineage barcode combination supported by the most reads."
- [methods] UMI filter parameters for per-cell annotation: "Barcodes can be filtered based on the number of supporting UMIs (`--umi_count_filter`) and by the number of UMIs in comparison to the dominant barcode per cell (`--umi_fraction_filter`)."
- [discussion] Benchmarking caveat from discussion: "benchmarking BARtab against similar tools was difficult due to the lack of a gold-standard dataset with known barcode abundance and differing parameter flexibility between tools."
- [full_text] Benchmarking comparison scope from source card finding: [evidence span withheld — anchor status: not_found]
