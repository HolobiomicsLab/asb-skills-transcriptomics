---
name: ct-format-file-processing
description: Use when the input RNA secondary structure is provided in CT format (six
  columns after one header line) and per-nucleotide Shannon entropy is available as
  a two-column tab-delimited text file, and structural content visualization is needed.
when_to_use_negative:
- The secondary structure input is in dot bracket notation (FASTA file) — use RSCanner_dotbracket_shannon.R
  instead of the CT script
- No per-nucleotide Shannon entropy file is available; the CT script requires both
  a structure file and a shannon.txt input
- The structure is not a pre-computed secondary structure — RSCanner scans an existing
  structure and does not predict or fold the RNA
edam_topics:
- http://edamontology.org/topic_0099
tools:
- name: RSCanner
  role: R script (RSCanner_CT_shannon.R) that reads the CT structure file plus the
    Shannon entropy file and computes BPC, smoothed entropy, and structure counts
  repo: https://github.com/pylelab/RSCanner
- name: R
  role: Runtime language required to execute RSCanner; three package dependencies
    (tidyverse, dplyr, ggplot2) must be installed
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad111
    title: 'RSCanner: rapid assessment and visualization of RNA structure content'
  source_content_hash: sha256:414ae01ea81328b0192407cf8aa8b1d90c7b21c019aad52e0778f5582bc1b261
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# ct-format-file-processing

## Summary

Process a connectivity table (CT) RNA secondary structure file through RSCanner's RSCanner_CT_shannon.R script, which pairs the CT structure with a tab-delimited per-nucleotide Shannon entropy file to compute base pair content (BPC), smoothed entropy, and structure counts across the RNA.

## When to use

Use when the input RNA secondary structure is provided in CT format (six columns after one header line) and per-nucleotide Shannon entropy is available as a two-column tab-delimited text file, and structural content visualization is needed.

## When NOT to use

- The secondary structure input is in dot bracket notation (FASTA file) — use RSCanner_dotbracket_shannon.R instead of the CT script
- No per-nucleotide Shannon entropy file is available; the CT script requires both a structure file and a shannon.txt input
- The structure is not a pre-computed secondary structure — RSCanner scans an existing structure and does not predict or fold the RNA

## Inputs

- RNA secondary structure in CT (connectivity table) format; six columns after one header line
- shannon.txt - positional Shannon entropy file, two columns, tab-delimited, no header (col1 = index, col2 = Shannon entropy values)
- User-defined parameters: window sizes (default 51 nt), Shannon entropy and BPC percentile cutoffs (default 0.5), lower/upper transcript bounds, image width/height/resolution (default 7/3 inches, 300 dpi), histogram/heatmap bin width (default 100 nt)

## Outputs

- bpcplot.tiff - BPC line plot with dashed line at the global median
- smoothed_Shannonplot.tiff - smoothed SE line plot with dashed line at the global median
- structure_counts_histogram.tiff - structure count histogram
- structure_counts_heatmap.tiff - final line plot/heatmap of structure counts
- bpc_data.csv - col1=index, col2=nucleotide number, col3=BPC
- smoothed_Shannon_data.csv - col1=index, col2=nucleotide number, col3=smoothed Shannon Entropy
- structure_counts.csv - col1=index, col2=structure counts, col3=BPC, col4=smoothed Shannon Entropy
- ordered_structure_table.csv - col1=index, col2=bin number, col3=% structure content, col4=bin start (nt), col5=bin end (nt)

## How to apply

Select RSCanner_CT_shannon.R (rather than the dot-bracket script) when the structure input is a CT file. Supply the CT file and the shannon.txt file (two columns, no header: col1 = index, col2 = Shannon entropy values) as command-line arguments. The script scans a centered sliding window (default 51 nt, 25 nt flanks, 1 nt steps, gradually truncated at the 5' and 3' RNA ends) to compute BPC and median Shannon entropy per window, then flags structure counts as positions with BPC above the 50th percentile cutoff and Shannon entropy below the 50th percentile cutoff. Accept the prompted defaults (51 nt windows, 0.5 percentile cutoffs, 7x3 inch images at 300 dpi, 100 nt heatmap bins) or adjust them interactively. Rationale: RSCanner uses the CT connectivity table to read the secondary structure and the entropy file to weight each window, so both files must cover the same transcript.

## Related tools

- **RSCanner** (R script (RSCanner_CT_shannon.R) that reads the CT structure file plus the Shannon entropy file and computes BPC, smoothed entropy, and structure counts) — https://github.com/pylelab/RSCanner
- **R** (Runtime language required to execute RSCanner; three package dependencies (tidyverse, dplyr, ggplot2) must be installed)

## Examples

```
Rscript ../RSCanner_CT_shannon.R ../sample_test_files/HCV_Jc1.ct ../sample_test_files/HCV_shannon.txt
```

## Evaluation signals

- All four named TIFF plots and four CSV tables (bpcplot.tiff, smoothed_Shannonplot.tiff, structure_counts_histogram.tiff, structure_counts_heatmap.tiff, bpc_data.csv, smoothed_Shannon_data.csv, structure_counts.csv, ordered_structure_table.csv) are generated
- BPC and smoothed Shannon entropy line plots show dashed reference lines at the global median
- bpc_data.csv and smoothed_Shannon_data.csv contain exactly three columns: index, nucleotide number, value
- If an even window size is given, the effective window is the next odd number (input=50 → 51), ensuring a centered sliding window
- structure counts reflect the dual filter: BPC above the 50th percentile AND Shannon entropy below the 50th percentile

## Limitations

- No version numbers are stated for R or the R packages (tidyverse, dplyr, ggplot2)
- Even-numbered window sizes are silently converted to the next odd number, so the user's input window size is not used verbatim
- Windows at the RNA ends are gradually truncated, so 5' and 3' terminal BPC/entropy values are computed over smaller windows
- The CT script path is only valid for CT inputs; using dot bracket FASTA input with RSCanner_CT_shannon.R is out of scope

## Evidence

- [readme] input_structure.ct        - CT format file; six columns after one header line: "input_structure.ct        - CT format file; six columns after one header line"
- [methods] Rscript RSCanner_CT_shannon.R <input_structure.ct>  <shannon.txt>: "Rscript RSCanner_CT_shannon.R <input_structure.ct>  <shannon.txt>"
- [readme] RSCanner scans the secondary structure provided along with Shannon entropy values in sliding windows covering the entire RNA: "RSCanner scans the secondary structure provided along with Shannon entropy values in sliding windows covering the entire RNA and calculates the base pair content (BPC) and median entropy values"
- [readme] shannon.txt               - two columns in a tab delimited file, col1 = index, col2 = shannon entropy values with no header: "shannon.txt               - two columns in a tab delimited file, col1 = index, col2 = shannon entropy values with no header"
