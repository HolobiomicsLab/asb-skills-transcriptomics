---
name: sliding-window-base-pair-content-calculation
description: Use when you have an RNA secondary structure in CT or dot-bracket format
  plus a positional Shannon entropy file, and need per-nucleotide BPC and smoothed
  SE profiles to locate structured vs. dynamic regions of a long transcript.
when_to_use_negative:
- You do not have a positional Shannon entropy file — RSCanner requires the structure
  file AND the shannon.txt as two mandatory inputs and cannot compute entropy itself
- You need to predict secondary structure from sequence — RSCanner scans a pre-specified
  secondary structure; it does not fold RNA
- The input is a single pre-computed structure-counts or feature table — this skill
  computes the upstream BPC/SE tracks, not the downstream percentile filtering (default
  BPC > 50th percentile, SE < 50th percentile)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0097
tools:
- name: RSCanner
  role: Performs the centered sliding-window calculation of BPC and median Shannon
    entropy over the provided secondary structure and entropy file; provided as RSCanner_CT_shannon.R
    and RSCanner_dotbracket_shannon.R
  repo: https://github.com/pylelab/RSCanner
- name: R
  role: Computing language in which RSCanner scripts are executed; scripts auto-install
    the required packages
- name: ggplot2
  role: R package dependency used by RSCanner to render the BPC and smoothed-SE line
    plots (TIFF at 300 dpi)
- name: dplyr
  role: R package dependency used by RSCanner for the window-wise data manipulation
    and tabulation of BPC/SE values
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad111
    title: 'RSCanner: rapid assessment and visualization of RNA structure content'
  source_content_hash: sha256:955a193410d20f3b71757c59dec1f27800ea1c0ae5f98ad5eba94ec542ee4484
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
  - http://edamontology.org/topic_0097
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# sliding-window-base-pair-content-calculation

## Summary

Compute per-nucleotide base pair content (BPC) and median Shannon entropy profiles across an RNA secondary structure using RSCanner's centered 51-nt sliding window (25-nt flanks, 1-nt steps). This converts a static structure/entropy input into positional structure-content tracks for downstream 'structure counts' filtering in long RNAs such as the HCV_Jc1 genome.

## When to use

Use when you have an RNA secondary structure in CT or dot-bracket format plus a positional Shannon entropy file, and need per-nucleotide BPC and smoothed SE profiles to locate structured vs. dynamic regions of a long transcript.

## When NOT to use

- You do not have a positional Shannon entropy file — RSCanner requires the structure file AND the shannon.txt as two mandatory inputs and cannot compute entropy itself
- You need to predict secondary structure from sequence — RSCanner scans a pre-specified secondary structure; it does not fold RNA
- The input is a single pre-computed structure-counts or feature table — this skill computes the upstream BPC/SE tracks, not the downstream percentile filtering (default BPC > 50th percentile, SE < 50th percentile)

## Inputs

- RNA secondary structure in CT (connectivity table) format, e.g. HCV_Jc1.ct (six columns after one header line)
- RNA secondary structure in dot-bracket (FASTA) format, e.g. HCV_Jc1.dot (alternative to CT)
- Positional Shannon entropy file, tab-delimited, two columns (col1=index, col2=entropy values, no header), e.g. HCV_shannon.txt, calculated from base pairing probabilities
- User-defined parameters: BPC window size (default 51), SE smoothing window size (default 51), percentile cutoffs (default 0.5), image width/height/resolution (7/3/300 dpi)

## Outputs

- bpcplot.tiff — per-nucleotide BPC line plot with dashed global-median reference line
- smoothed_Shannonplot.tiff — smoothed SE line plot with dashed global-median reference line
- bpc_data.csv — index, nucleotide number, BPC value per nucleotide
- smoothed_Shannon_data.csv — index, nucleotide number, smoothed Shannon Entropy value per nucleotide

## How to apply

Run RSCanner_CT_shannon.R (for CT input) or RSCanner_dotbracket_shannon.R (for dot-bracket/FASTA input), passing the structure file and the tab-delimited Shannon entropy file (col1=index, col2=entropy, no header). Accept the default centered sliding window of 51 nt (25 nt flanks each side of the center nucleotide, moved in 1-nt steps) for both BPC calculation and Shannon entropy smoothing; if an even window size is inputted, RSCanner substitutes the next odd number (input=50 becomes 51) so a centered window is always maintained. At the 5' and 3' ends, allow RSCanner to gradually truncate the window (e.g., window #26=[1nt,51nt], window #25=[1nt,50nt], etc.) rather than padding. For each window, calculate BPC (fraction of base-paired nucleotides) and the median Shannon entropy, then emit line plots of BPC and smoothed SE with dashed lines at the global medians (TIFF, 7x3 inch, 300 dpi) and the per-nucleotide value tables bpc_data.csv and smoothed_Shannon_data.csv (index, nucleotide number, value). These smoothed tracks are then fed into the structure-counts filtering step (BPC above a percentile cutoff, SE below a percentile cutoff, defaults 0.50/0.50).

## Related tools

- **RSCanner** (Performs the centered sliding-window calculation of BPC and median Shannon entropy over the provided secondary structure and entropy file; provided as RSCanner_CT_shannon.R and RSCanner_dotbracket_shannon.R) — https://github.com/pylelab/RSCanner
- **R** (Computing language in which RSCanner scripts are executed; scripts auto-install the required packages)
- **ggplot2** (R package dependency used by RSCanner to render the BPC and smoothed-SE line plots (TIFF at 300 dpi))
- **dplyr** (R package dependency used by RSCanner for the window-wise data manipulation and tabulation of BPC/SE values)

## Examples

```
Rscript ../RSCanner_CT_shannon.R ../sample_test_files/HCV_Jc1.ct ../sample_test_files/HCV_shannon.txt
```

## Evaluation signals

- Even-window invariant: supplying window size 50 must result in an actual window size of 51, so all windows remain centered
- End-truncation invariant: near the 5' end the windows shrink progressively (window #26=[1nt,51nt], #25=[1nt,50nt], #24=[1nt,49nt], etc.), so per-nucleotide values exist for every position including position 1
- Schema check: bpc_data.csv and smoothed_Shannon_data.csv each contain exactly three columns (index, nucleotide number, value) with one row per nucleotide spanning the full transcript length
- Plot check: bpcplot.tiff and smoothed_Shannonplot.tiff are 7x3 inch TIFFs at 300 dpi and each displays a dashed line at the global median, matching Supplementary Fig. S2A and B
- Downstream consistency: with default 0.50 BPC / 0.50 SE percentile cutoffs, the resulting structure_counts.csv must reproduce the default-parameter structure counts for the HCV_Jc1 sample

## Limitations

- Even-numbered window sizes are silently converted to the next odd number (input=50 → window=51), so the user's exact window size is not honored verbatim
- Windows are gradually truncated at the 5' and 3' ends, so edge nucleotides have BPC/SE values computed from progressively smaller windows with less local context
- Two separate scripts must be chosen based on input format (CT vs. dot-bracket); no version numbers for R or the tidyverse/dplyr/ggplot2 dependencies are specified
- Shannon entropy must be pre-computed externally from base pairing probabilities; RSCanner does not derive it from the structure file

## Evidence

- [methods] Centered sliding window calculation of BPC and median Shannon entropy: "RSCanner uses a centered sliding window (default=51nt, 25nt flanks to each side of the center nucleotide, moved in steps of 1 nt across the RNA) to calculate BPC and median Shannon Entropy."
- [readme] Window truncation at RNA ends: "When dealing with the ends, RSCanner will gradually truncate the window size (i.e., for 5' end windows - window #26=[1nt,51nt], window#25=[1nt,50nt], window#24 =[1nt,49nt], etc;"
- [readme] Even-window handling: "If an even number window size is inputted, RSCanner will use the next odd number as the window size (i.e., if input=50, the actual window size=51)."
- [readme] Output specification for BPC and smoothed SE: "bpcplot.tiff - output BPC line plot figure, dashed line represents the global median"
