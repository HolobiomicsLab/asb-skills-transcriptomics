---
name: rna-structural-content-figure-generation
description: Use when you have an RNA secondary structure in CT (connectivity table)
  or dot-bracket FASTA format plus a tab-delimited per-nucleotide Shannon entropy
  file, and you need BPC/entropy plots and structure-count heatmap/histogram outputs.
when_to_use_negative:
- The secondary structure file is neither CT format nor dot-bracket FASTA (the two
  scripts dispatch only on these two input types).
- No per-nucleotide Shannon entropy list is available — both scripts require the structure
  file to be paired with a shannon.txt entropy file.
- The goal is interactive exploration rather than command-line figure generation —
  the README suggests the Rmd notebook via RStudio as the alternate access mode.
edam_operation: http://edamontology.org/operation_2481
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: RSCanner
  role: 'The skill''s tool: two dispatch scripts (RSCanner_CT_shannon.R and RSCanner_dotbracket_shannon.R)
    compute BPC and median Shannon entropy over a centered 51-nt sliding window and
    emit the four figure and four CSV outputs'
  repo: https://github.com/pylelab/RSCanner
- name: R
  role: Runtime for the dispatch scripts, invoked via Rscript from a Linux terminal
  repo: https://www.r-project.org/
- name: ggplot2
  role: One of three R package dependencies (with tidyverse and dplyr) used to render
    the BPC, Shannon, histogram, and heatmap figures
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad111
    title: 'RSCanner: rapid assessment and visualization of RNA structure content'
  source_content_hash: sha256:0743050165780f01244cfad986eb2a3cd611599c799a58f4809f4e13f85f49bb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pylelab/RSCanner
  tools:
  - R
  - RSCanner
  - dplyr
  - ggplot2
  - tidyverse
  edam_operation: http://edamontology.org/operation_2481
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# rna-structural-content-figure-generation

## Summary

Generate RSCanner's RNA structural-content figures (BPC line plot, smoothed Shannon entropy plot, structure-counts histogram, and heatmap) by routing a secondary-structure file in CT or dot-bracket format, paired with a per-nucleotide Shannon entropy list, to the correct R script.

## When to use

Use when you have an RNA secondary structure in CT (connectivity table) or dot-bracket FASTA format plus a tab-delimited per-nucleotide Shannon entropy file, and you need BPC/entropy plots and structure-count heatmap/histogram outputs.

## When NOT to use

- The secondary structure file is neither CT format nor dot-bracket FASTA (the two scripts dispatch only on these two input types).
- No per-nucleotide Shannon entropy list is available — both scripts require the structure file to be paired with a shannon.txt entropy file.
- The goal is interactive exploration rather than command-line figure generation — the README suggests the Rmd notebook via RStudio as the alternate access mode.

## Inputs

- RNA secondary structure in CT (connectivity table) format, e.g. sample_test_files/HCV_Jc1.ct
- RNA secondary structure in dot-bracket notation FASTA format, e.g. sample_test_files/HCV_Jc1.dot
- Positional Shannon entropy file (shannon.txt), two tab-delimited columns: index and entropy value, no header, e.g. sample_test_files/HCV_shannon.txt
- User-defined parameters: 51-nt BPC window, 51-nt Shannon smoothing window, 0.5 BPC and Shannon percentile cutoffs, transcript-length bounds, 7x3-inch images at 300 dpi, 100-nt histogram/heatmap bins

## Outputs

- bpcplot.tiff — BPC line plot with dashed line at the global median
- smoothed_Shannonplot.tiff — smoothed Shannon entropy line plot with dashed global-median line
- structure_counts_histogram.tiff — histogram of structure counts
- structure_counts_heatmap.tiff — line plot/heatmap of structure-count frequency distribution
- bpc_data.csv — index, nucleotide number, BPC
- smoothed_Shannon_data.csv — index, nucleotide number, smoothed Shannon entropy
- structure_counts.csv — index, structure counts, BPC, smoothed Shannon entropy
- ordered_structure_table.csv — index, bin number, % structure content, bin start/end (nt)

## How to apply

Select the dispatch script by input format: RSCanner_CT_shannon.R for CT files (six columns after one header line) or RSCanner_dotbracket_shannon.R for dot-bracket FASTA files, passing the structure file and shannon.txt (two tab-delimited columns, no header) as arguments. Run from a sample_outputs working directory inside the cloned pylelab/RSCanner repo; the scripts auto-install the tidyverse, dplyr, and ggplot2 dependencies if missing. Accept the prompted defaults: 51-nt centered sliding window for BPC and Shannon smoothing (even inputs are bumped to the next odd number, and windows are truncated at transcript ends), 0.5 percentile cutoffs for both BPC (above) and Shannon entropy (below), transcript-length bounds, 7x3-inch images at 300 dpi, and 100-nt bins for the histogram/heatmap. Each run writes eight files (four .tiff figures and four .csv tables) into sample_outputs/. Correctness is judged by presence of all eight outputs and by plots matching the declared content, e.g. dashed global-median lines on BPC and smoothed Shannon plots.

## Related tools

- **RSCanner** (The skill's tool: two dispatch scripts (RSCanner_CT_shannon.R and RSCanner_dotbracket_shannon.R) compute BPC and median Shannon entropy over a centered 51-nt sliding window and emit the four figure and four CSV outputs) — https://github.com/pylelab/RSCanner
- **R** (Runtime for the dispatch scripts, invoked via Rscript from a Linux terminal) — https://www.r-project.org/
- **ggplot2** (One of three R package dependencies (with tidyverse and dplyr) used to render the BPC, Shannon, histogram, and heatmap figures)

## Examples

```
mkdir sample_outputs; cd sample_outputs; Rscript ../RSCanner_CT_shannon.R ../sample_test_files/HCV_Jc1.ct ../sample_test_files/HCV_shannon.txt
```

## Evaluation signals

- All eight declared output files (bpcplot.tiff, smoothed_Shannonplot.tiff, structure_counts_histogram.tiff, structure_counts_heatmap.tiff, bpc_data.csv, smoothed_Shannon_data.csv, structure_counts.csv, ordered_structure_table.csv) are written into the working directory.
- BPC and smoothed Shannon line plots include a dashed line representing the global median, as declared in the script's usage text.
- CT and dot-bracket branches run on the same HCV_shannon.txt entropy file produce comparable structure-count outputs for the matched HCV_Jc1 transcripts.
- CSV schemas match the declared column layouts, e.g. bpc_data.csv has col1=index, col2=nucleotide number, col3=BPC; structure_counts.csv has index, structure counts, BPC, smoothed Shannon entropy.
- If an even window size (e.g. 50) is supplied, the actual window used is the next odd number (51), so a centered sliding window is maintained.

## Limitations

- The Shannon entropy file must be tab-delimited with exactly two columns (index, entropy) and no header; deviations break parsing.
- No version numbers for R or the tidyverse/dplyr/ggplot2 packages are stated in the article or README, so environment reproducibility depends on current CRAN snapshots.
- Structure-count classification depends on user-defined percentile cutoffs (default 50th), so figures change with non-default cutoffs; the article's discussion/limitations section was not available.

## Evidence

- [intro] RSCanner is an R package for rapid assessment and visualization of RNA structural content that is particularly useful for long RNAs: "RSCanner is an R package for rapid assessment and visualization of RNA structural content that is particularly useful for long RNAs"
- [intro] Two-script dispatch on input type: "command line functionality has been provided via two separate R scripts, to be used depending on the type of input given"
- [methods] Cutoffs and structure counts: "it computes nucleotide positions with BPC values above a user-defined cutoff (default = 50th percentile) and Shannon entropy values below a user-defined cutoff (default = 50th percentile)"
- [methods] Sliding window geometry: "RSCanner uses a centered sliding window (default=51nt, 25nt flanks to each side of the center nucleotide, moved in steps of 1 nt across the RNA) to calculate BPC and median Shannon Entropy."
- [readme] CT input spec: "input_structure.ct        - CT format file; six columns after one header line"
- [readme] Shannon file spec: "shannon.txt               - two columns in a tab delimited file, col1 = index, col2 = shannon entropy values with no header"
- [readme] Even window handling: "If an even number window size is inputted, RSCanner will use the next odd number as the window size (i.e., if input=50, the actual window size=51)."
- [readme] Example invocation: "Rscript ../RSCanner_CT_shannon.R ../sample_test_files/HCV_Jc1.ct ../sample_test_files/HCV_shannon.txt"
