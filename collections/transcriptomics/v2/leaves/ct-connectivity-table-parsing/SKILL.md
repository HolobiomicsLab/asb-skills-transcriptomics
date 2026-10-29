---
name: ct-connectivity-table-parsing
description: Use when the available RNA secondary structure file is a CT/connectivity-table
  file (six columns after one header line) and you need to route it, paired with a
  Shannon entropy text file, into RSCanner for BPC and entropy scanning.
when_to_use_negative:
- The secondary structure input is a dot-bracket (FASTA) file — RSCanner routes that
  through RSCanner_dotbracket_shannon.R instead of the CT script.
- You lack a per-nucleotide Shannon entropy list computed from base pairing probabilities;
  both RSCanner scripts require the structure file to be paired with one.
- You want a single-command multiple-input pipeline; RSCanner's command-line usage
  requires one structure file plus one entropy file per run of the appropriate script.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0097
tools:
- name: RSCanner
  role: R package whose CT-specific script (RSCanner_CT_shannon.R) parses the connectivity-table
    structure and scans it with Shannon entropy values in sliding windows to produce
    BPC and structure-count outputs
  repo: https://github.com/pylelab/RSCanner
- name: R
  role: Computing language in which the CT parsing and scanning scripts are implemented
    and executed via Rscript
- name: tidyverse
  role: One of three R package dependencies the CT script checks for and installs
    automatically before parsing and plotting
- name: dplyr
  role: R package dependency used for data manipulation in the CT-parsing workflow
- name: ggplot2
  role: R package dependency used to render the BPC, smoothed Shannon, histogram,
    and heatmap TIFF outputs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad111
    title: 'RSCanner: rapid assessment and visualization of RNA structure content'
  source_content_hash: sha256:30157a6c9a64c050293e1722171faa2dba60a53efc5b89ad8d5f251503598bcb
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
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0097
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# ct-connectivity-table-parsing

## Summary

Parses an RNA secondary structure in CT (connectivity table) format together with a per-nucleotide Shannon entropy list as input to RSCanner's CT-specific command-line script, which computes base pair content (BPC) and smoothed entropy via a centered sliding window.

## When to use

Use when the available RNA secondary structure file is a CT/connectivity-table file (six columns after one header line) and you need to route it, paired with a Shannon entropy text file, into RSCanner for BPC and entropy scanning.

## When NOT to use

- The secondary structure input is a dot-bracket (FASTA) file — RSCanner routes that through RSCanner_dotbracket_shannon.R instead of the CT script.
- You lack a per-nucleotide Shannon entropy list computed from base pairing probabilities; both RSCanner scripts require the structure file to be paired with one.
- You want a single-command multiple-input pipeline; RSCanner's command-line usage requires one structure file plus one entropy file per run of the appropriate script.

## Inputs

- RNA secondary structure in CT (connectivity table) format — six columns after one header line (e.g. HCV_Jc1.ct)
- Per-nucleotide Shannon entropy file — two-column tab-delimited text, col1 = index, col2 = entropy values, no header (e.g. HCV_shannon.txt)

## Outputs

- bpcplot.tiff — BPC line plot with dashed line at the global median
- smoothed_Shannonplot.tiff — smoothed Shannon entropy line plot with global median
- structure_counts_histogram.tiff
- structure_counts_heatmap.tiff — line plot/heatmap of structure-count frequency across the RNA
- bpc_data.csv — index, nucleotide number, BPC
- smoothed_Shannon_data.csv — index, nucleotide number, smoothed Shannon entropy
- structure_counts.csv — index, structure counts, BPC, smoothed Shannon entropy
- ordered_structure_table.csv — index, bin number, % structure content, bin start/end (nt)

## How to apply

Confirm the structure input is a CT file — six columns following one header line, each row encoding a nucleotide's connectivity — rather than a dot-bracket FASTA, because RSCanner dispatches to two separate scripts depending on input type. Prepare the companion Shannon entropy file as a two-column, tab-delimited table (col1 = index, col2 = entropy values, no header). From a sample_outputs directory inside the cloned RSCanner-main repository, run Rscript ../RSCanner_CT_shannon.R with the .ct file and shannon.txt as positional arguments. The script auto-checks and installs tidyverse, dplyr, and ggplot2 if missing, then prompts for parameters (accept defaults: 51-nt windows for BPC and Shannon smoothing, 0.5 percentile cutoffs for BPC and Shannon entropy, transcript-length bounds, 7x3-inch 300 dpi images, 100-nt histogram/heatmap bins). RSCanner uses a centered sliding window (51 nt default, 25 nt flanks, 1-nt step, truncated at RNA ends) to calculate BPC and median Shannon entropy, then flags structure counts where BPC is above the percentile cutoff and Shannon entropy below it.

## Related tools

- **RSCanner** (R package whose CT-specific script (RSCanner_CT_shannon.R) parses the connectivity-table structure and scans it with Shannon entropy values in sliding windows to produce BPC and structure-count outputs) — https://github.com/pylelab/RSCanner
- **R** (Computing language in which the CT parsing and scanning scripts are implemented and executed via Rscript)
- **tidyverse** (One of three R package dependencies the CT script checks for and installs automatically before parsing and plotting)
- **dplyr** (R package dependency used for data manipulation in the CT-parsing workflow)
- **ggplot2** (R package dependency used to render the BPC, smoothed Shannon, histogram, and heatmap TIFF outputs)

## Examples

```
Rscript ../RSCanner_CT_shannon.R ../sample_test_files/HCV_Jc1.ct ../sample_test_files/HCV_shannon.txt
```

## Evaluation signals

- All eight declared output files (bpcplot.tiff, smoothed_Shannonplot.tiff, structure_counts_histogram.tiff, structure_counts_heatmap.tiff, bpc_data.csv, smoothed_Shannon_data.csv, structure_counts.csv, ordered_structure_table.csv) are written into the working directory (sample_outputs/).
- The CT input parses as six columns after one header line; the Shannon file parses as two tab-delimited columns with no header (index, entropy).
- Smoothed Shannon entropy and BPC values are computed with a centered odd-sized window (even inputs are bumped to the next odd number, e.g. 50 → 51) with window truncation at the 5'/3' ends.
- Structure counts in structure_counts.csv correspond to positions with BPC above the 0.5 percentile cutoff AND Shannon entropy below the 0.5 percentile cutoff.
- Rerunning the CT branch on HCV_Jc1.ct + HCV_shannon.txt with default prompts reproduces the same eight outputs as the dot-bracket branch structure-count logic.

## Limitations

- Only two input types are supported (CT or dot-bracket FASTA); any other structure format must be converted before use.
- Even-numbered window sizes are silently adjusted to the next odd number, so the effective window differs from the user input.
- At RNA ends the sliding window is gradually truncated, so terminal windows cover fewer nucleotides (e.g. window #26 = [1nt, 51nt]).
- No version numbers for R or its packages are specified in the documentation.

## Evidence

- [readme] Input files: 1 - RNA secondary structure in either dot bracket notation (FASTA file) or CT (connectivity table) format; 2 - positional Shannon entropy for each nucleotide: "Input files: 1 - RNA secondary structure in either dot bracket notation (FASTA file) or CT (connectivity table) format; 2 - positional Shannon entropy for each nucleotide in the RNA"
- [readme] CT format file; six columns after one header line: "input_structure.ct        - CT format file; six columns after one header line"
- [intro] command line functionality has been provided via two separate R scripts, to be used depending on the type of input given: "command line functionality has been provided via two separate R scripts, to be used depending on the type of input given"
- [methods] Rscript RSCanner_CT_shannon.R <input_structure.ct>  <shannon.txt>: "Rscript RSCanner_CT_shannon.R <input_structure.ct>  <shannon.txt>"
- [methods] centered sliding window (default=51nt, 25nt flanks to each side of the center nucleotide, moved in steps of 1 nt across the RNA): "RSCanner uses a centered sliding window (default=51nt, 25nt flanks to each side of the center nucleotide, moved in steps of 1 nt across the RNA) to calculate BPC and median Shannon Entropy."
