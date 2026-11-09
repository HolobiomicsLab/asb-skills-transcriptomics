---
name: shannon-entropy-median-smoothing
description: Use when you have per-nucleotide Shannon entropy values (e.g. from base
  pairing probabilities) alongside a CT or dot-bracket RNA secondary structure and
  need a smoothed entropy profile to visualize local structural dynamics.
when_to_use_negative:
- You need raw (unsmoothed) per-nucleotide Shannon entropy values rather than a windowed
  median-smoothed profile — feed the entropy file directly to downstream tools instead.
- Your input lacks a positional Shannon entropy file; RSCanner's CT and dot-bracket
  scripts both require entropy values calculated from base pairing probabilities as
  a second mandatory input.
- You want to detect structure from experimental probing data (e.g. SHAPE reactivities)
  rather than from a predicted secondary structure plus entropy — RSCanner scans a
  provided secondary structure only.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_0097
tools:
- name: RSCanner
  role: R package whose RSCanner_CT_shannon.R / RSCanner_dotbracket_shannon.R scripts
    perform the centered 51 nt sliding-window median smoothing of Shannon entropy
    and write the smoothed plot and CSV outputs
  repo: https://github.com/pylelab/RSCanner
- name: R
  role: Runtime language required to execute the RSCanner scripts and Rmd notebook
- name: ggplot2
  role: R package dependency used to render the smoothed Shannon entropy line plot
    with dashed global-median reference line
- name: dplyr
  role: R package dependency for data manipulation of per-nucleotide entropy values
    in the RSCanner scripts
- name: tidyverse
  role: R package dependency of the RSCanner scripts
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad111
    title: 'RSCanner: rapid assessment and visualization of RNA structure content'
  source_content_hash: sha256:1616f4b1686525b952375977d54b8ecab86f6aaa6adc8aa84846b12fbe643388
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_0097
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# shannon-entropy-median-smoothing

## Summary

Smooths per-nucleotide Shannon entropy values from an RNA secondary structure analysis using RSCanner's centered 51 nt sliding window median approach, producing a per-nucleotide smoothed entropy profile that parallels base-pairing content (BPC) for identifying structured or dynamic regions in long RNAs.

## When to use

Use when you have per-nucleotide Shannon entropy values (e.g. from base pairing probabilities) alongside a CT or dot-bracket RNA secondary structure and need a smoothed entropy profile to visualize local structural dynamics.

## When NOT to use

- You need raw (unsmoothed) per-nucleotide Shannon entropy values rather than a windowed median-smoothed profile — feed the entropy file directly to downstream tools instead.
- Your input lacks a positional Shannon entropy file; RSCanner's CT and dot-bracket scripts both require entropy values calculated from base pairing probabilities as a second mandatory input.
- You want to detect structure from experimental probing data (e.g. SHAPE reactivities) rather than from a predicted secondary structure plus entropy — RSCanner scans a provided secondary structure only.

## Inputs

- RNA secondary structure in CT (connectivity table) format (six columns after one header line), e.g. HCV_Jc1.ct
- RNA secondary structure in dot-bracket notation (FASTA format), e.g. HCV_Jc1.dot
- Positional Shannon entropy file: two-column tab-delimited text, col1 = index, col2 = Shannon entropy values, no header (e.g. HCV_shannon.txt)

## Outputs

- smoothed_Shannonplot.tiff — smoothed Shannon entropy line plot, dashed line marks the global median (300 dpi, 7x3 inch)
- smoothed_Shannon_data.csv — tab-delimited table, col1=index, col2=nucleotide number, col3=smoothed Shannon Entropy
- structure_counts.csv — tab-delimited table with index, structure counts, BPC, and smoothed Shannon Entropy per position
- structure_counts_heatmap.tiff and structure_counts_histogram.tiff — frequency distribution of structure counts using a 100 nt window

## How to apply

Load the secondary structure (CT format via RSCanner_CT_shannon.R, or dot-bracket FASTA via RSCanner_dotbracket_shannon.R) and the two-column tab-delimited Shannon entropy file (col1 = index, col2 = entropy, no header) into R. Slide a centered 51 nt window (25 nt flanks on each side, 1 nt steps) across the RNA and compute the median Shannon entropy per window; if an even window size is given, RSCanner uses the next odd number so the window is always centered. At the 5' and 3' ends, gradually truncate the window (e.g. window #26=[1nt,51nt], window #25=[1nt,50nt], window #24=[1nt,49nt]) rather than padding. Plot the smoothed entropy as a line plot with a dashed line at the global median (TIFF, 300 dpi, 7x3 inch) and write the values to smoothed_Shannon_data.csv (columns: index, nucleotide number, value). The recommended parameter set also prompts for a Shannon Entropy percentile cutoff (default 0.5), bounds (1 to transcript length), and a 100 nt window for the histogram/heatmap.

## Related tools

- **RSCanner** (R package whose RSCanner_CT_shannon.R / RSCanner_dotbracket_shannon.R scripts perform the centered 51 nt sliding-window median smoothing of Shannon entropy and write the smoothed plot and CSV outputs) — https://github.com/pylelab/RSCanner
- **R** (Runtime language required to execute the RSCanner scripts and Rmd notebook)
- **ggplot2** (R package dependency used to render the smoothed Shannon entropy line plot with dashed global-median reference line)
- **dplyr** (R package dependency for data manipulation of per-nucleotide entropy values in the RSCanner scripts)
- **tidyverse** (R package dependency of the RSCanner scripts)

## Examples

```
Rscript ../RSCanner_CT_shannon.R ../sample_test_files/HCV_Jc1.ct ../sample_test_files/HCV_shannon.txt
```

## Evaluation signals

- All expected output files are generated, in particular smoothed_Shannonplot.tiff and smoothed_Shannon_data.csv.
- smoothed_Shannonplot.tiff shows a line plot with a dashed line at the global median, matching Supplementary Fig. S2B.
- smoothed_Shannon_data.csv has three columns (index, nucleotide number, smoothed Shannon Entropy) and one row per transcript position.
- The same 51 nt centered window (odd window enforced; even inputs bumped to the next odd number) is used for entropy smoothing as for BPC, with gradual truncation at both RNA ends.
- Running RSCanner_CT_shannon.R and RSCanner_dotbracket_shannon.R on the equivalent HCV_Jc1 CT and dot inputs produces consistent smoothed entropy profiles.

## Limitations

- No version numbers are stated for R or the required packages (tidyverse, dplyr, ggplot2), so reproducibility depends on the user's R environment.
- The window size (default 51 nt) and percentile cutoffs (default 0.5) are user-defined parameters; different choices change the smoothing and structure-count calls.
- Entropy must be supplied per nucleotide from base pairing probabilities — RSCanner does not compute the Shannon entropy values itself.
- No limitations or changelog were extracted from the article's discussion section ('No changelog found.').

## Evidence

- [readme] RSCanner uses a centered sliding window (default=51nt, 25nt flanks to each side of the center nucleotide, moved in steps of 1 nt across the RNA) to calculate BPC and median Shannon Entropy.: "RSCanner uses a centered sliding window (default=51nt, 25nt flanks to each side of the center nucleotide, moved in steps of 1 nt across the RNA) to calculate BPC and median Shannon Entropy."
- [readme] If an even number window size is inputted, RSCanner will use the next odd number as the window size (i.e., if input=50, the actual window size=51).: "If an even number window size is inputted, RSCanner will use the next odd number as the window size (i.e., if input=50, the actual window size=51)."
- [methods] When dealing with the ends, RSCanner will gradually truncate the window size (i.e., for 5' end windows - window #26=[1nt,51nt], window#25=[1nt,50nt], window#24 =[1nt,49nt], etc: "When dealing with the ends, RSCanner will gradually truncate the window size (i.e., for 5' end windows - window #26=[1nt,51nt], window#25=[1nt,50nt], window#24 =[1nt,49nt], etc"
- [readme] smoothed_Shannonplot.tiff                - output smoothed SE line plot figure, dashed line represents the global median: "smoothed_Shannonplot.tiff                - output smoothed SE line plot figure, dashed line represents the global median"
- [readme] smoothed_Shannon_data.csv                - output tab delimited file, col1=index, col2=nucleotide number, col3=smoothed Shannon Entropy: "smoothed_Shannon_data.csv                - output tab delimited file, col1=index, col2=nucleotide number, col3=smoothed Shannon Entropy"
