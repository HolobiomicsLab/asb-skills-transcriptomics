---
name: per-nucleotide-shannon-entropy-mapping
description: Use when you have a per-nucleotide Shannon entropy list (two-column,
  tab-delimited, no header) and a pre-computed RNA secondary structure in CT or dot-bracket
  format, and need sliding-window structure-count maps.
when_to_use_negative:
- You have only a sequence and need folding prediction — RSCanner requires a pre-computed
  secondary structure (CT or dot-bracket) plus pre-calculated per-nucleotide Shannon
  entropy as inputs.
- The structure file is neither CT (six columns after one header line) nor dot-bracket
  FASTA — neither dispatch script parses other structure formats.
- No positional Shannon entropy list exists for the transcript — the entropy file
  is a mandatory second argument for both scripts.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: RSCanner
  role: Two dispatchable R scripts (RSCanner_CT_shannon.R and RSCanner_dotbracket_shannon.R)
    that pair the structure input with the Shannon entropy list, run the 51-nt centered
    sliding-window scan, and emit the heatmap/histogram TIFFs and CSV tables
  repo: https://github.com/pylelab/RSCanner
- name: R
  role: Computing-language runtime required to execute the R scripts from the terminal
    via Rscript
  repo: https://www.r-project.org/
- name: tidyverse
  role: R package dependency auto-checked and installed by the RSCanner scripts at
    runtime
- name: dplyr
  role: R package dependency auto-checked and installed by the RSCanner scripts at
    runtime
- name: ggplot2
  role: R package dependency used for the BPC, smoothed-entropy, histogram, and heatmap
    figure outputs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad111
    title: 'RSCanner: rapid assessment and visualization of RNA structure content'
  source_content_hash: sha256:69a22d46d7ae845496fbba44c43fee25191fae49f57866643e65b575485f0aae
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# per-nucleotide-shannon-entropy-mapping

## Summary

RSCanner consumes a per-nucleotide Shannon entropy list paired with an RNA secondary structure in either CT (connectivity table) or dot-bracket FASTA format, dispatching to one of two R scripts based on the structure format. It computes 51-nt centered sliding-window base pair content (BPC) and median entropy to map regions of stable vs. dynamic RNA structure.

## When to use

Use when you have a per-nucleotide Shannon entropy list (two-column, tab-delimited, no header) and a pre-computed RNA secondary structure in CT or dot-bracket format, and need sliding-window structure-count maps.

## When NOT to use

- You have only a sequence and need folding prediction — RSCanner requires a pre-computed secondary structure (CT or dot-bracket) plus pre-calculated per-nucleotide Shannon entropy as inputs.
- The structure file is neither CT (six columns after one header line) nor dot-bracket FASTA — neither dispatch script parses other structure formats.
- No positional Shannon entropy list exists for the transcript — the entropy file is a mandatory second argument for both scripts.

## Inputs

- RNA secondary structure in CT (connectivity table) format: six columns after one header line
- RNA secondary structure in dot-bracket notation (FASTA file)
- Per-nucleotide Shannon entropy file (shannon.txt): two-column tab-delimited file, col1 = index, col2 = entropy values, no header
- Optional user-defined parameters: window sizes (default 51), percentile cutoffs (default 0.5), transcript bounds, image dimensions 7x3 in at 300 dpi, 100-nt histogram/heatmap bins

## Outputs

- bpcplot.tiff — BPC line plot with dashed line at the global median
- smoothed_Shannonplot.tiff — smoothed Shannon entropy line plot with global median line
- structure_counts_histogram.tiff — frequency distribution histogram of structure counts
- structure_counts_heatmap.tiff — line plot/heatmap of structure counts across the RNA
- bpc_data.csv — index, nucleotide number, BPC
- smoothed_Shannon_data.csv — index, nucleotide number, smoothed Shannon Entropy
- structure_counts.csv — index, structure counts, BPC, smoothed Shannon Entropy
- ordered_structure_table.csv — index, bin number, % structure content, bin start (nt), bin end (nt)

## How to apply

Prepare the Shannon entropy file as two tab-delimited columns (col1 = nucleotide index, col2 = entropy value, no header line). Choose the dispatch script by structure format: run RSCanner_CT_shannon.R when the structure is a CT file (six columns after one header line), or RSCanner_dotbracket_shannon.R when it is dot-bracket FASTA — both scripts are selected depending on the type of input given. Execute with Rscript from a working directory such as sample_outputs/, passing the structure file and the entropy file as the two positional arguments; the scripts check for and auto-install the tidyverse, dplyr, and ggplot2 dependencies if missing. Accept the prompted parameters (or supply equivalents): 51-nt window for BPC, 51-nt window for Shannon smoothing, 0.5 percentile cutoffs for both BPC and Shannon entropy, transcript-length bounds, 7x3-inch images at 300 dpi, and 100-nt bins for the histogram/heatmap. RSCanner scans the structure along with the entropy values in centered sliding windows (25 nt flanks each side, 1-nt step, truncated at transcript ends) and flags structure counts as positions with BPC above the cutoff and entropy below the cutoff.

## Related tools

- **RSCanner** (Two dispatchable R scripts (RSCanner_CT_shannon.R and RSCanner_dotbracket_shannon.R) that pair the structure input with the Shannon entropy list, run the 51-nt centered sliding-window scan, and emit the heatmap/histogram TIFFs and CSV tables) — https://github.com/pylelab/RSCanner
- **R** (Computing-language runtime required to execute the R scripts from the terminal via Rscript) — https://www.r-project.org/
- **tidyverse** (R package dependency auto-checked and installed by the RSCanner scripts at runtime)
- **dplyr** (R package dependency auto-checked and installed by the RSCanner scripts at runtime)
- **ggplot2** (R package dependency used for the BPC, smoothed-entropy, histogram, and heatmap figure outputs)

## Examples

```
Rscript ../RSCanner_CT_shannon.R ../sample_test_files/HCV_Jc1.ct ../sample_test_files/HCV_shannon.txt
```

## Evaluation signals

- Each run writes all eight declared files (bpcplot.tiff, smoothed_Shannonplot.tiff, structure_counts_histogram.tiff, structure_counts_heatmap.tiff, bpc_data.csv, smoothed_Shannon_data.csv, structure_counts.csv, ordered_structure_table.csv) into the working directory (sample_outputs/).
- The entropy input is accepted only if it is exactly two tab-delimited columns (index, entropy) with no header line; format violations should cause a parse failure.
- At default settings, structure_counts.csv contains only positions where BPC is above the 50th percentile and Shannon entropy is below the 50th percentile.
- Running the CT branch (HCV_Jc1.ct) and the dot-bracket branch (HCV_Jc1.dot) with the same HCV_shannon.txt should produce comparable structure-count outputs on the sample data.
- The ordered_structure_table.csv bins should span the transcript with 100-nt bin start/end coordinates, matching the prompted histogram/heatmap bin size.

## Limitations

- If an even window size is input, RSCanner silently substitutes the next odd number (input 50 becomes window 51) to keep the window centered.
- Windows at the transcript ends are gradually truncated rather than wrapped (e.g., 5' window #26 = [1nt,51nt], #25 = [1nt,50nt]), so end windows rest on fewer flanking nucleotides.
- No version numbers are stated for R or the three package dependencies; the article's discussion text was absent, so reproducibility caveats beyond those documented could not be extracted.

## Evidence

- [intro] command line functionality has been provided via two separate R scripts, to be used depending on the type of input given: "command line functionality has been provided via two separate R scripts, to be used depending on the type of input given"
- [methods] RSCanner scans the secondary structure provided along with Shannon entropy values in sliding windows covering the entire RNA and calculates the base pair content (BPC) and median entropy values: "RSCanner scans the secondary structure provided along with Shannon entropy values in sliding windows covering the entire RNA and calculates the base pair content (BPC) and median entropy values"
- [methods] it computes nucleotide positions with BPC values above a user-defined cutoff (default = 50th percentile) and Shannon entropy values below a user-defined cutoff (default = 50th percentile): "it computes nucleotide positions with BPC values above a user-defined cutoff (default = 50th percentile) and Shannon entropy values below a user-defined cutoff (default = 50th percentile)"
- [readme] col1 = index, col2 = shannon entropy values with no header: "col1 = index, col2 = shannon entropy values with no header"
- [readme] Rscript ../RSCanner_CT_shannon.R ../sample_test_files/HCV_Jc1.ct ../sample_test_files/HCV_shannon.txt: "Rscript ../RSCanner_CT_shannon.R ../sample_test_files/HCV_Jc1.ct ../sample_test_files/HCV_shannon.txt"
- [readme] RSCanner uses a centered sliding window (default=51nt, 25nt flanks to each side of the center nucleotide, moved in steps of 1 nt across the RNA) to calculate BPC and median Shannon Entropy.: "RSCanner uses a centered sliding window (default=51nt, 25nt flanks to each side of the center nucleotide, moved in steps of 1 nt across the RNA) to calculate BPC and median Shannon Entropy."
