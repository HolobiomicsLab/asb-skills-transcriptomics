---
name: shannon-entropy-percentile-thresholding
description: Use when you have per-nucleotide BPC and smoothed Shannon entropy values
  (from a CT or dot-bracket structure plus a shannon.txt file) and need to binarize
  positions into 'structure counts' using percentile cutoffs before building heatmap/histogram
  summaries.
when_to_use_negative:
- The structure file is not in CT or dot-bracket (FASTA) format, or no positional
  shannon.txt is available — RSCanner's two scripts require exactly these input types.
- You need absolute (absolute-threshold) structural stability calls rather than relative
  percentile-based labels within one transcript; the default 50th-percentile cutoffs
  are transcript-relative.
- The goal is raw per-window BPC or entropy profiling (bpcplot.tiff / smoothed_Shannonplot.tiff)
  rather than binarized structure-count classification.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0097
tools:
- name: RSCanner
  role: R package implementing the sliding-window BPC/entropy computation and the
    percentile-cutoff structure-count filtering step
  repo: https://github.com/pylelab/RSCanner
- name: R
  role: Computing language in which the RSCanner filtering scripts (RSCanner_CT_shannon.R
    / RSCanner_dotbracket_shannon.R) are run
- name: dplyr
  role: R package dependency used inside RSCanner for data manipulation during filtering
    and summarization
- name: ggplot2
  role: R package dependency used to render the structure-counts histogram and heatmap
    outputs
- name: tidyverse
  role: R package dependency of the RSCanner scripts
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad111
    title: 'RSCanner: rapid assessment and visualization of RNA structure content'
  source_content_hash: sha256:3bd585552d1695840f8ef6cce0efb44715b90a37252e1a6e57a0d6a5c5095276
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0097
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# shannon-entropy-percentile-thresholding

## Summary

RSCanner's two-criterion filtering step that designates individual nucleotides as 'structure counts': positions are kept only when smoothed BPC exceeds a percentile cutoff (default 50th) AND smoothed Shannon entropy falls below a percentile cutoff (default 50th), flagging nucleotides that are both heavily base-paired and structurally stable (low entropy).

## When to use

Use when you have per-nucleotide BPC and smoothed Shannon entropy values (from a CT or dot-bracket structure plus a shannon.txt file) and need to binarize positions into 'structure counts' using percentile cutoffs before building heatmap/histogram summaries.

## When NOT to use

- The structure file is not in CT or dot-bracket (FASTA) format, or no positional shannon.txt is available — RSCanner's two scripts require exactly these input types.
- You need absolute (absolute-threshold) structural stability calls rather than relative percentile-based labels within one transcript; the default 50th-percentile cutoffs are transcript-relative.
- The goal is raw per-window BPC or entropy profiling (bpcplot.tiff / smoothed_Shannonplot.tiff) rather than binarized structure-count classification.

## Inputs

- bpc_data.csv — tab-delimited, col1=index, col2=nucleotide number, col3=BPC (from the centered sliding-window scan)
- smoothed_Shannon_data.csv — tab-delimited, col1=index, col2=nucleotide number, col3=smoothed Shannon Entropy
- shannon.txt — two-column tab-delimited positional Shannon entropy file (col1=index, col2=entropy, no header), upstream input to smoothing
- Percentile cutoffs for BPC and Shannon entropy as decimals between 0 and 1 (default 0.5 each)

## Outputs

- structure_counts.csv — tab-delimited, col1=index, col2=structure counts, col3=BPC, col4=smoothed Shannon Entropy
- structure_counts_histogram.tiff — frequency distribution of structure counts across the RNA
- structure_counts_heatmap.tiff — final line plot/heatmap of structure-count density

## How to apply

After RSCanner's centered 51-nt sliding window (25-nt flanks, 1-nt step, truncated at RNA ends) has produced bpc_data.csv and smoothed_Shannon_data.csv, compute global percentile cutoffs for each track — the defaults are the 50th percentile for both (input as decimals between 0 and 1: 0.5 for BPC and 0.5 for Shannon entropy). Label a nucleotide position as a structure count only when BPC is above its cutoff AND smoothed Shannon entropy is below its cutoff, i.e., high base-pair content combined with low entropy (stability), so the two tracks jointly define the call. Write the per-position calls to structure_counts.csv with columns index, structure counts, BPC, and smoothed Shannon Entropy. The resulting structure-count series then feeds the frequency-distribution heatmap and histogram, computed over a default 100-nt window (images 7x3 inches at 300 dpi). Decision point: the cutoffs are user-defined at the program prompt, so tighten or relax both percentiles symmetrically only if your downstream interpretation calls for it.

## Related tools

- **RSCanner** (R package implementing the sliding-window BPC/entropy computation and the percentile-cutoff structure-count filtering step) — https://github.com/pylelab/RSCanner
- **R** (Computing language in which the RSCanner filtering scripts (RSCanner_CT_shannon.R / RSCanner_dotbracket_shannon.R) are run)
- **dplyr** (R package dependency used inside RSCanner for data manipulation during filtering and summarization)
- **ggplot2** (R package dependency used to render the structure-counts histogram and heatmap outputs)
- **tidyverse** (R package dependency of the RSCanner scripts)

## Examples

```
Rscript ../RSCanner_CT_shannon.R ../sample_test_files/HCV_Jc1.ct ../sample_test_files/HCV_shannon.txt
```

## Evaluation signals

- structure_counts.csv exists with exactly the four documented columns: index, structure counts, BPC, smoothed Shannon Entropy
- Every row flagged as a structure count satisfies BPC above the chosen percentile cutoff (default 0.5) AND smoothed Shannon entropy below its cutoff (default 0.5); rows failing either condition are not structure counts
- structure_counts_histogram.tiff and structure_counts_heatmap.tiff are produced with the default image settings (7x3 inches, 300 dpi) using the default 100-nt histogram/heatmap window
- The number of structure counts changes predictably if the percentile cutoffs at the program prompt are shifted (e.g., a lower entropy cutoff keeps fewer positions)
- For an even-number window input (e.g., 50), RSCanner uses the next odd number (51) as the window size, so the smoothed values feeding the filter must come from an odd-sized centered window

## Limitations

- Cutoffs are user-defined at the program prompt; defaults are the 50th percentile for both BPC and Shannon entropy, and no version numbers for R or the packages are documented.
- With an even-number window size input, RSCanner silently uses the next odd number as the window size, which affects the smoothed entropy values feeding the filter.
- At RNA ends the sliding window is gradually truncated (e.g., 5' end windows #26=[1nt,51nt], #25=[1nt,50nt], ...), so edge positions are scored with fewer flanking nucleotides.
- The discussion section text was absent from the provided context, so further caveats or reproducibility gaps could not be extracted.

## Evidence

- [methods] criterion: "it computes nucleotide positions with BPC values above a user-defined cutoff (default = 50th percentile) and Shannon entropy values below a user-defined cutoff (default = 50th percentile). These"
- [readme] outputs: "RSCanner then outputs the frequency distribution of structure counts across the RNA as both heatmap and histogram plots."
- [readme] window: "RSCanner uses a centered sliding window (default=51nt, 25nt flanks to each side of the center nucleotide, moved in steps of 1 nt across the RNA)"
- [readme] parameters: "Shannon Entropy percentile cutoff, decimal between 0 and 1: 0.5"
