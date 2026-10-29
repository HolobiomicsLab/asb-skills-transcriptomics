---
name: r-command-line-script-execution
description: Use when reproducing a tool whose authors ship two separate R scripts
  — one for connectivity-table (.ct) and one for dot-bracket (FASTA) structure input
  — and the analysis goal is terminal-based batch execution.
when_to_use_negative:
- The structure input is neither CT format nor dot-bracket FASTA (e.g. a Stockholm
  or WUSS alignment) — neither script will accept it without prior conversion.
- The user wants the interactive RStudio Rmd notebook workflow instead of terminal
  execution, or lacks Rscript on the system.
- Only raw sequence or base-pairing probability data is available with no pre-computed
  per-nucleotide Shannon entropy list — the scripts require that entropy file as a
  mandatory second input.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: RSCanner
  role: the tool being executed; provided as two command-line R scripts (RSCanner_CT_shannon.R
    and RSCanner_dotbracket_shannon.R) dispatched by input format
  repo: https://github.com/pylelab/RSCanner
- name: R
  role: the computing language runtime invoked via Rscript; auto-installs missing
    packages
- name: tidyverse
  role: R package dependency auto-checked and installed by the scripts
- name: dplyr
  role: R package dependency auto-checked and installed by the scripts
- name: ggplot2
  role: R package dependency auto-checked and installed by the scripts; used for the
    plot outputs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad111
    title: 'RSCanner: rapid assessment and visualization of RNA structure content'
  source_content_hash: sha256:459d9fd9dbba71710efba7e2cc9890f48e30ca46c0ebbd65cdaf677d7408c933
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# r-command-line-script-execution

## Summary

Execute paper-provided R analysis pipelines (e.g. RSCanner) via Rscript on a Linux terminal, selecting the script variant that matches the input file format and accepting interactive parameter prompts.

## When to use

Use when reproducing a tool whose authors ship two separate R scripts — one for connectivity-table (.ct) and one for dot-bracket (FASTA) structure input — and the analysis goal is terminal-based batch execution.

## When NOT to use

- The structure input is neither CT format nor dot-bracket FASTA (e.g. a Stockholm or WUSS alignment) — neither script will accept it without prior conversion.
- The user wants the interactive RStudio Rmd notebook workflow instead of terminal execution, or lacks Rscript on the system.
- Only raw sequence or base-pairing probability data is available with no pre-computed per-nucleotide Shannon entropy list — the scripts require that entropy file as a mandatory second input.

## Inputs

- RNA secondary structure in CT (connectivity table) format: six columns after one header line
- RNA secondary structure in dot-bracket notation (FASTA file)
- Shannon entropy file (shannon.txt): two-column tab-delimited file, col1 = index, col2 = Shannon entropy values, no header

## Outputs

- bpcplot.tiff — BPC line plot with dashed global-median line
- smoothed_Shannonplot.tiff — smoothed Shannon entropy line plot with dashed global-median line
- structure_counts_histogram.tiff — histogram of structure counts
- structure_counts_heatmap.tiff — line plot/heatmap of structure counts
- bpc_data.csv — index, nucleotide number, BPC
- smoothed_Shannon_data.csv — index, nucleotide number, smoothed Shannon entropy
- structure_counts.csv — index, structure counts, BPC, smoothed Shannon entropy
- ordered_structure_table.csv — index, bin number, % structure content, bin start/end (nt)

## How to apply

First install R from a CRAN mirror (https://www.r-project.org/) and clone the tool's GitHub repository; the scripts auto-check and auto-install the tidyverse, dplyr, and ggplot2 dependencies if missing. Identify the input structure format: a CT file (six columns after one header line) routes to the CT script, while a dot-bracket FASTA file routes to the alternate script; both scripts pair the structure with a two-column, tab-delimited Shannon entropy file (index, entropy value, no header). Run the chosen script with Rscript from a freshly created output directory (e.g. mkdir sample_outputs; cd sample_outputs) passing the structure file and entropy file as the two positional arguments. At each run, the program prompts for user-defined parameters — accept the recommended defaults: 51-nt window for BPC, 51-nt window for Shannon smoothing, 0.5 percentile cutoffs for both BPC and Shannon entropy, transcript-length bounds (1 to transcript length), 7x3-inch images at 300 dpi, and 100-nt bins for the histogram/heatmap. Note that an even window size is silently bumped to the next odd number so the centered sliding window (25-nt flanks, 1-nt step) is always used, and windows at transcript ends are gradually truncated.

## Related tools

- **RSCanner** (the tool being executed; provided as two command-line R scripts (RSCanner_CT_shannon.R and RSCanner_dotbracket_shannon.R) dispatched by input format) — https://github.com/pylelab/RSCanner
- **R** (the computing language runtime invoked via Rscript; auto-installs missing packages)
- **tidyverse** (R package dependency auto-checked and installed by the scripts)
- **dplyr** (R package dependency auto-checked and installed by the scripts)
- **ggplot2** (R package dependency auto-checked and installed by the scripts; used for the plot outputs)

## Examples

```
Rscript ../RSCanner_CT_shannon.R ../sample_test_files/HCV_Jc1.ct ../sample_test_files/HCV_shannon.txt
```

## Evaluation signals

- Each run writes exactly eight declared files (bpcplot.tiff, smoothed_Shannonplot.tiff, structure_counts_histogram.tiff, structure_counts_heatmap.tiff, bpc_data.csv, smoothed_Shannon_data.csv, structure_counts.csv, ordered_structure_table.csv) into the working directory.
- The BPC and smoothed Shannon entropy cutoffs applied correspond to the prompted 0.5 percentile values (structure counts = BPC above cutoff AND entropy below cutoff).
- If an even window size was entered, the realized window size is odd (e.g. input=50 yields 51), reflecting the centered-window rule.
- The entropy input parses as two tab-delimited columns with no header; a parse error indicates the wrong file was paired with the structure.
- CT-branch and dot-bracket-branch runs on the paired HCV_Jc1 test files should produce structurally comparable output tables for the same entropy input.

## Limitations

- Shannon entropy must be pre-computed per nucleotide from base pairing probabilities and supplied externally; the scripts do not compute it.
- Version numbers for R and the packages are not specified in the documentation, so exact dependency versions are untested.
- The discussion section provides no changelog or known-bug list ('No changelog found.'), leaving undocumented failure modes.
- Windows are truncated at transcript 5'/3' ends, so early-window values use fewer than 51 nt and are not directly comparable to interior windows.

## Evidence

- [intro] Command-line dispatch by input format: "command line functionality has been provided via two separate R scripts, to be used depending on the type of input given"
- [abstract] Script-callable interface: "RSCanner is freely available as both R script and R Markdown files"
- [readme] Input formats and entropy pairing: "1 - RNA secondary structure in either dot bracket notation (FASTA file) or CT (connectivity table) format; 2 - positional Shannon entropy for each nucleotide in the RNA"
- [methods] CT script argument order: "Rscript RSCanner_CT_shannon.R <input_structure.ct>  <shannon.txt>"
- [readme] Package auto-install behavior: "The Rscripts automatically check for the required packages and install them if needed."
- [methods] Percentile cutoffs for structure counts: "it computes nucleotide positions with BPC values above a user-defined cutoff (default = 50th percentile) and Shannon entropy values below a user-defined cutoff"
