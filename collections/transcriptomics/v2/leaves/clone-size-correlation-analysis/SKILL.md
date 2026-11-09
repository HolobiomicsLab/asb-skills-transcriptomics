---
name: clone-size-correlation-analysis
description: Use when two lineage barcoding pre-processing pipelines must be cross-validated
  on identical single-cell amplicon FASTQ data and agreement should be quantified
  by per-cell annotation overlap and clone-size correlation.
when_to_use_negative:
- Input is already a barcode counts/annotation table and no cross-pipeline re-processing
  of raw FASTQ is required
- Data are population-level (bulk) barcoding reads rather than single-cell data —
  the population mode workflow does not involve cell barcode whitelists or PARSE_BARCODES_SC
  annotation
- No second pipeline's published annotations exist to compare against, so clone-size
  correlation has no reference
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: BARtab
  role: Nextflow pipeline run in single-cell mode to generate cell barcode annotation
    tables from the FM0-2 FASTQ data
  repo: https://github.com/DaneVass/BARtab
- name: bartools
  role: R package used for downstream analysis and visualization of the single-cell
    barcoding comparison
  repo: https://github.com/DaneVass/bartools
- name: FateMap
  role: Reference pipeline (Rewind/FateMap, Goyal et al.) re-run with UMI threshold
    15 to produce the comparator cell annotations
  repo: https://github.com/arjunrajlaboratory/FateMap_Goyal2023
- name: umi-tools
  role: whitelist/extract cell barcodes and UMIs from R1 and count UMIs per barcode
    per cell (count_tab, umi_dist 1, umi_count_filter 5)
  repo: https://github.com/CGATOxford/UMI-tools
- name: cutadapt
  role: Trims constant regions from barcode reads (--constants all for reference-based
    randomly fragmented reads)
- name: fastp
  role: Quality filters raw reads with minqual 20 and pctqual 80
  repo: https://github.com/OpenGene/fastp
- name: bowtie1
  role: Aligns trimmed reads to the SPLINTR barcode reference index with -a --best
    --strata -m1 --norc
- name: starcode
  role: starcode-umi used in the UMI processing step of the workflow
  repo: https://github.com/gui11aume/starcode
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100763
    title: Analysis of synthetic cellular barcodes in the genome and transcriptome
      with BARtab and bartools
  source_content_hash: sha256:7ad311d607a5290df0a91e3ac311ff562c861df54e1e8aa02b38961e40ed8bde
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DaneVass/BARtab
  tools:
  - BARtab
  - FastQC
  - FateMap
  - R
  - bartools
  - bowtie1
  - cutadapt
  - fastp
  - samtools
  - starcode
  - umi-tools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# clone-size-correlation-analysis

## Summary

Quantitative validation of a single-cell lineage barcode annotation pipeline (BARtab) against an existing pipeline (FateMap/Rewind) by comparing per-cell barcode annotations and per-clone sizes on the same FateMap FM0-2 paired-end amplicon FASTQ data, using overlap counts and Pearson clone-size correlations in R.

## When to use

Use when two lineage barcoding pre-processing pipelines must be cross-validated on identical single-cell amplicon FASTQ data and agreement should be quantified by per-cell annotation overlap and clone-size correlation.

## When NOT to use

- Input is already a barcode counts/annotation table and no cross-pipeline re-processing of raw FASTQ is required
- Data are population-level (bulk) barcoding reads rather than single-cell data — the population mode workflow does not involve cell barcode whitelists or PARSE_BARCODES_SC annotation
- No second pipeline's published annotations exist to compare against, so clone-size correlation has no reference

## Inputs

- FateMap (FM0-2) paired-end single-cell amplicon sequencing FASTQ files
- GEO-derived cell barcode whitelists formatted as <sample_id>_whitelist.tsv files with '-1' suffixes removed
- SPLINTR_mCHERRY_V2_barcode_reference_library.fasta lineage barcode reference library

## Outputs

- Per-cell barcode annotation table from BARtab (PARSE_BARCODES_SC, umi_fraction_filter 0.3)
- Per-cell barcode annotation table from the FateMap pipeline (UMI threshold 15)
- Per-cell annotation overlap contingency counts (BARtab-only, FateMap-only, both)
- Per-clone size table per pipeline and Pearson correlation coefficients between clone sizes

## How to apply

Run BARtab in single-cell mode ('nextflow run DaneVass/bartab --mode single-cell --input_type fastq') on the FM0-2 paired-end amplicon FASTQ files, providing GEO-derived cell barcode whitelists (<sample_id>_whitelist.tsv, '-1' suffixes removed) via --whitelist_indir. Pre-process reads with FastQC, fastp quality filtering (minqual 20, pctqual 80), umi-tools whitelist/extract with pattern CCCCCCCCCCCCCCCCNNNNNNNNNNNN on R1, and cutadapt constant-region trimming (--constants all for randomly fragmented reads). Align trimmed reads to a bowtie1 index of the SPLINTR barcode reference library with '-a --best --strata -m1 --norc' (alnmismatches 2), retaining only alignments starting at the first or ending at the last reference position (FILTER_ALIGNMENTS), remove PCR chimerism by keeping the most-supported cell barcode-UMI-lineage barcode combination, then count UMIs per barcode per cell with umi-tools count_tab (umi_dist 1, umi_count_filter 5) and aggregate per-cell barcodes with PARSE_BARCODES_SC (umi_fraction_filter 0.3). Re-run the FateMap pipeline on the same data with its UMI threshold of 15, then compare per-cell barcode annotations and clone sizes between pipelines in R (bartools) via overlap contingency counts and Pearson correlation. Correct application is confirmed when the cell annotation overlap reproduces the published 99.6% (12 BARtab-only, 145 FateMap-only, 39,884 in both) and Pearson clone-size correlations fall between 0.965 and 0.998.

## Related tools

- **BARtab** (Nextflow pipeline run in single-cell mode to generate cell barcode annotation tables from the FM0-2 FASTQ data) — https://github.com/DaneVass/BARtab
- **bartools** (R package used for downstream analysis and visualization of the single-cell barcoding comparison) — https://github.com/DaneVass/bartools
- **FateMap** (Reference pipeline (Rewind/FateMap, Goyal et al.) re-run with UMI threshold 15 to produce the comparator cell annotations) — https://github.com/arjunrajlaboratory/FateMap_Goyal2023
- **umi-tools** (whitelist/extract cell barcodes and UMIs from R1 and count UMIs per barcode per cell (count_tab, umi_dist 1, umi_count_filter 5)) — https://github.com/CGATOxford/UMI-tools
- **cutadapt** (Trims constant regions from barcode reads (--constants all for reference-based randomly fragmented reads))
- **fastp** (Quality filters raw reads with minqual 20 and pctqual 80) — https://github.com/OpenGene/fastp
- **bowtie1** (Aligns trimmed reads to the SPLINTR barcode reference index with -a --best --strata -m1 --norc)
- **starcode** (starcode-umi used in the UMI processing step of the workflow) — https://github.com/gui11aume/starcode

## Examples

```
nextflow run DaneVass/bartab --mode single-cell --input_type fastq --whitelist_indir <dir> --ref SPLINTR_mCHERRY_V2_barcode_reference_library.fasta
```

## Evaluation signals

- Cell annotation overlap reproduces the published 99.6% agreement across the four conditions (12 BARtab-only, 145 FateMap-only, 39,884 cells annotated by both)
- Pearson clone-size correlations between BARtab and FateMap annotations fall between 0.965 and 0.998
- FASTQ QC confirms reads passed fastp filtering at minqual 20 / pctqual 80 before barcode extraction
- Only alignments starting at the first or ending at the last reference position are retained after FILTER_ALIGNMENTS
- Per-cell tables show each cell's barcode passing the umi_fraction_filter 0.3 dominance criterion and the FateMap arm's UMI threshold of 15

## Limitations

- Benchmarking was hampered by the lack of a gold-standard dataset with known barcode abundance and differing parameter flexibility between tools
- Barcode assignment is sensitive to filter choices: umi_count_filter 5 (BARtab arm) versus FateMap's UMI threshold of 15, and umi_fraction_filter 0.3
- The default Levenshtein distance of 3 for barcode clustering is conservative and avoids collapsing truly distinct barcodes, which affects clone-size definition
- BARtab and bartools do not officially support evolving barcode methods

## Evidence

- [intro] to facilitate the downstream analysis and visualization of cellular barcoding workflows at bulk and single-cell resolution, we developed bartools, a flexible open-source R package: "to facilitate the downstream analysis and visualization of cellular barcoding workflows at bulk and single-cell resolution, we developed bartools, a flexible open-source R package"
- [methods] Barcodes can be filtered based on the number of supporting UMIs (`--umi_count_filter`): "Barcodes can be filtered based on the number of supporting UMIs (`--umi_count_filter`)"
- [methods] and by the number of UMIs in comparison to the dominant barcode per cell (`--umi_fraction_filter`).: "and by the number of UMIs in comparison to the dominant barcode per cell (`--umi_fraction_filter`)."
- [methods] Alignments that start at the first position or end at the last are retained.: "Alignments that start at the first position or end at the last are retained."
- [discussion] We benchmarked BARtab by reproducing published results, showing improved runtime and sensitivity.: "We benchmarked BARtab by reproducing published results, showing improved runtime and sensitivity."
- [readme] Comparison of [BARtab](https://github.com/DaneVass/BARtab/) with pycashier and timemachine in `tools-comparison/`.: [evidence span withheld — anchor status: external_doc]
