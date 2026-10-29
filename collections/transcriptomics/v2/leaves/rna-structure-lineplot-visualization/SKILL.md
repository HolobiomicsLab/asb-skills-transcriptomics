---
name: rna-structure-lineplot-visualization
description: Use when you have an RNA secondary structure (CT or dot-bracket format)
  plus a per-nucleotide Shannon entropy file and need per-nucleotide BPC and smoothed
  SE line plots (e.g. reproducing Supplementary Fig. S2A/B for HCV_Jc1).
when_to_use_negative:
- No positional Shannon entropy file (calculated from base pairing probabilities)
  is available — RSCanner_CT_shannon.R / RSCanner_dotbracket_shannon.R require both
  a structure file and a shannon.txt input
- Input is not a secondary structure in CT or dot-bracket (FASTA) format (e.g. raw
  sequencing data, 3D coordinates, or alignment files)
- The goal is the structure-counts frequency distribution (heatmap/histogram) rather
  than per-nucleotide BPC and smoothed SE line plots
edam_operation: http://edamontology.org/operation_3200
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: RSCanner
  role: R package whose R scripts (RSCanner_CT_shannon.R, RSCanner_dotbracket_shannon.R)
    perform the centered sliding-window BPC and median Shannon entropy calculation
    and generate the line plots
  repo: github.com/pylelab/RSCanner
- name: R
  role: Computing language in which the RSCanner scripts and Rmd notebook are implemented;
    run from a Linux terminal or RStudio
- name: ggplot2
  role: R package dependency used to render the BPC and smoothed Shannon entropy line
    plots
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad111
    title: 'RSCanner: rapid assessment and visualization of RNA structure content'
  source_content_hash: sha256:dde64299e05516366a5bebd065dda494e642783c0ec460bd5f717fc7ddfdb6c2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - RSCanner
  - dplyr
  - ggplot2
  - tidyverse
  edam_operation: http://edamontology.org/operation_3200
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# rna-structure-lineplot-visualization

## Summary

Per-nucleotide line-plot visualization of RNA secondary-structure content: base-pairing content (BPC) and smoothed Shannon entropy computed over a centered 51 nt sliding window (as in RSCanner), rendered as ggplot2 line plots with dashed global-median reference lines and exported as CSV data tables.

## When to use

Use when you have an RNA secondary structure (CT or dot-bracket format) plus a per-nucleotide Shannon entropy file and need per-nucleotide BPC and smoothed SE line plots (e.g. reproducing Supplementary Fig. S2A/B for HCV_Jc1).

## When NOT to use

- No positional Shannon entropy file (calculated from base pairing probabilities) is available — RSCanner_CT_shannon.R / RSCanner_dotbracket_shannon.R require both a structure file and a shannon.txt input
- Input is not a secondary structure in CT or dot-bracket (FASTA) format (e.g. raw sequencing data, 3D coordinates, or alignment files)
- The goal is the structure-counts frequency distribution (heatmap/histogram) rather than per-nucleotide BPC and smoothed SE line plots

## Inputs

- RNA secondary structure in CT (connectivity table) format, e.g. HCV_Jc1.ct (six columns after one header line)
- RNA secondary structure in dot-bracket (FASTA) format, e.g. HCV_Jc1.dot (alternative to CT)
- Positional Shannon entropy file, two columns, tab-delimited, no header (col1 = index, col2 = shannon entropy values), e.g. HCV_shannon.txt

## Outputs

- bpcplot.tiff — BPC line plot figure with dashed line at the global median (300 dpi, 7x3 inch)
- smoothed_Shannonplot.tiff — smoothed Shannon entropy line plot with dashed global-median line (300 dpi, 7x3 inch)
- bpc_data.csv — col1=index, col2=nucleotide number, col3=BPC
- smoothed_Shannon_data.csv — col1=index, col2=nucleotide number, col3=smoothed Shannon entropy

## How to apply

Load the secondary structure (CT via RSCanner_CT_shannon.R, or dot-bracket via RSCanner_dotbracket_shannon.R) and the tab-delimited two-column Shannon entropy file (col1=index, col2=entropy, no header) in R with tidyverse/dplyr/ggplot2. Slide a centered 51 nt window (25 nt flanks on each side of the center nucleotide, 1 nt steps) across the RNA; if an even window size is supplied, RSCanner uses the next odd number so the window is always centered, and at the 5'/3' ends the window is gradually truncated (e.g. 5' end window #26=[1nt,51nt], window #25=[1nt,50nt]). Compute base-pair content per window for BPC and take the median Shannon entropy per window for entropy smoothing. Plot both as line plots with dashed lines at the global medians, saved as TIFF images at 300 dpi with 7x3 inch dimensions, and write the values to bpc_data.csv and smoothed_Shannon_data.csv (columns: index, nucleotide number, value).

## Related tools

- **RSCanner** (R package whose R scripts (RSCanner_CT_shannon.R, RSCanner_dotbracket_shannon.R) perform the centered sliding-window BPC and median Shannon entropy calculation and generate the line plots) — github.com/pylelab/RSCanner
- **R** (Computing language in which the RSCanner scripts and Rmd notebook are implemented; run from a Linux terminal or RStudio)
- **ggplot2** (R package dependency used to render the BPC and smoothed Shannon entropy line plots)

## Examples

```
Rscript ../RSCanner_CT_shannon.R ../sample_test_files/HCV_Jc1.ct ../sample_test_files/HCV_shannon.txt
```

## Evaluation signals

- All four named output files are generated: bpcplot.tiff, smoothed_Shannonplot.tiff, bpc_data.csv, smoothed_Shannon_data.csv
- Both TIFF line plots display a dashed reference line at the global median, matching Supplementary Fig. S2A and B
- CSV tables contain the three expected columns (index, nucleotide number, value) and cover the transcript length with 1 nt steps
- An even user-supplied window size is converted to the next odd number (e.g. input=50 → window=51) so the window stays centered
- Values at the 5'/3' ends reflect gradual window truncation (window #26=[1nt,51nt], #25=[1nt,50nt], etc.) rather than dropped or padded positions
- Images are saved at 300 dpi with 7x3 inch dimensions

## Limitations

- Even-numbered window sizes are silently converted to the next odd number; a centered window is always enforced
- At the transcript ends the window is gradually truncated, so end values are computed over fewer nucleotides than interior windows
- The Shannon entropy input must be a two-column tab-delimited file with no header; other formats are not handled
- No version numbers are stated for R or the tidyverse/dplyr/ggplot2 dependencies (scripts auto-install missing packages)

## Evidence

- [readme] RSCanner uses a centered sliding window (default=51nt, 25nt flanks to each side of the center nucleotide, moved in steps of 1 nt across the RNA) to calculate BPC and median Shannon Entropy.: "RSCanner uses a centered sliding window (default=51nt, 25nt flanks to each side of the center nucleotide, moved in steps of 1 nt across the RNA) to calculate BPC and median Shannon Entropy."
- [readme] When dealing with the ends, RSCanner will gradually truncate the window size (i.e., for 5' end windows - window #26=[1nt,51nt], window#25=[1nt,50nt], window#24 =[1nt,49nt], etc): "When dealing with the ends, RSCanner will gradually truncate the window size (i.e., for 5' end windows - window #26=[1nt,51nt], window#25=[1nt,50nt], window#24 =[1nt,49nt], etc"
- [readme] bpcplot.tiff - output BPC line plot figure, dashed line represents the global median: "bpcplot.tiff                             - output BPC line plot figure, dashed line represents the global median"
- [methods] RSCanner scans the secondary structure provided along with Shannon entropy values in sliding windows covering the entire RNA and calculates the base pair content (BPC) and median entropy values: "RSCanner scans the secondary structure provided along with Shannon entropy values in sliding windows covering the entire RNA and calculates the base pair content (BPC) and median entropy values for"
- [methods] Rscript ../RSCanner_CT_shannon.R ../sample_test_files/HCV_Jc1.ct ../sample_test_files/HCV_shannon.txt: "Rscript ../RSCanner_CT_shannon.R ../sample_test_files/HCV_Jc1.ct ../sample_test_files/HCV_shannon.txt"
- [other] Running RSCanner on the HCV genome yields per-nucleotide BPC and smoothed SE values output as line plots (Supplementary Fig. S2A and B) together with accompanying text files.: [evidence span withheld — anchor status: not_found]
