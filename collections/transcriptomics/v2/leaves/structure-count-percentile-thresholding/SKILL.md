---
name: structure-count-percentile-thresholding
description: Use when you have per-window BPC and smoothed Shannon entropy values
  (e.g., from RSCanner's 51-nt centered sliding windows on a CT or dot-bracket structure
  plus a Shannon file) and need to flag highly-structured positions and quantify structure
  density per genomic bin.
when_to_use_negative:
- Input is raw probing data (e.g., SHAPE or DMS reactivities) rather than a precomputed
  secondary structure (CT or dot bracket) plus a Shannon entropy file — RSCanner requires
  these precomputed inputs.
- You need overlapping/windowed per-nucleotide structure values rather than the non-overlapping
  100-nt binned % structure content summarized by ordered_structure_table.csv.
- The RNA is so short that 51-nt centered windows are mostly truncated, making 50th-percentile
  cutoffs unstable across the transcript.
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_0097
tools:
- name: RSCanner
  role: Executes the sliding-window BPC/Shannon scan and applies the percentile cutoffs
    to produce structure counts and downstream heatmap/histogram/table outputs
  repo: https://github.com/pylelab/RSCanner
- name: R
  role: Runtime environment; RSCanner is distributed as R scripts and an R Markdown
    notebook requiring R pre-installation
  repo: https://www.r-project.org/
- name: tidyverse
  role: One of the three R package dependencies required by the RSCanner script
- name: dplyr
  role: One of the three R package dependencies required by the RSCanner script
- name: ggplot2
  role: Renders the BPC, Shannon, histogram, and heatmap TIFF plots
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad111
    title: 'RSCanner: rapid assessment and visualization of RNA structure content'
  source_content_hash: sha256:04aa041ff6490149600d83dc6109915600b8d7fe369378d4efe430e516977824
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
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_0097
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# structure-count-percentile-thresholding

## Summary

Computes per-nucleotide 'structure counts' by thresholding sliding-window base pair content (BPC) above a percentile cutoff and smoothed Shannon entropy below a percentile cutoff, then aggregates counts into non-overlapping bins of default 100 nt to yield % structure content along an RNA.

## When to use

Use when you have per-window BPC and smoothed Shannon entropy values (e.g., from RSCanner's 51-nt centered sliding windows on a CT or dot-bracket structure plus a Shannon file) and need to flag highly-structured positions and quantify structure density per genomic bin.

## When NOT to use

- Input is raw probing data (e.g., SHAPE or DMS reactivities) rather than a precomputed secondary structure (CT or dot bracket) plus a Shannon entropy file — RSCanner requires these precomputed inputs.
- You need overlapping/windowed per-nucleotide structure values rather than the non-overlapping 100-nt binned % structure content summarized by ordered_structure_table.csv.
- The RNA is so short that 51-nt centered windows are mostly truncated, making 50th-percentile cutoffs unstable across the transcript.

## Inputs

- RNA secondary structure file in CT format (e.g., HCV_Jc1.ct) or dot-bracket FASTA format (e.g., HCV_Jc1.dot)
- Positional Shannon entropy file, two-column tab-delimited (col1=index, col2=Shannon entropy, no header; e.g., HCV_shannon.txt)
- User-defined parameters: window size (default 51 nt), percentile cutoffs (default 0.5), lower/upper bounds, bin window length for histogram/heatmap (default 100 nt)

## Outputs

- structure_counts.csv — col1=index, col2=structure counts, col3=BPC, col4=smoothed Shannon Entropy
- bpc_data.csv and smoothed_Shannon_data.csv (per-nucleotide BPC and smoothed Shannon entropy)
- bpcplot.tiff and smoothed_Shannonplot.tiff with global-median dashed lines
- structure_counts_histogram.tiff and structure_counts_heatmap.tiff (default 7 x 3 inches, 300 dpi)
- ordered_structure_table.csv — col1=index, col2=bin number, col3=% structure content, col4=bin start (nt), col5=bin end (nt)

## How to apply

Run RSCanner (RSCanner_CT_shannon.R for CT input, RSCanner_dotbracket_shannon.R for dot-bracket) so it slides a centered 51-nt window (25-nt flanks, 1-nt step, gradually truncated at the RNA ends) computing BPC and median Shannon entropy. At each prompt, set the BPC percentile cutoff and Shannon entropy percentile cutoff to 0.5 (default 50th percentile); a position counts as structured when its BPC is ABOVE the cutoff and its Shannon entropy is BELOW the cutoff — the conjunction captures stable yet well-defined pairing. RSCanner writes structure_counts.csv (index, structure counts, BPC, smoothed Shannon Entropy). Optionally set upper/lower bounds (default 1 to transcript length) to restrict the region analyzed. Downstream, bin structure counts into non-overlapping windows of default 100 nt tiling the transcript and rank bins by % structure content (ordered_structure_table.csv) to locate high-structure-density regions such as known structural elements in HCV_Jc1.

## Related tools

- **RSCanner** (Executes the sliding-window BPC/Shannon scan and applies the percentile cutoffs to produce structure counts and downstream heatmap/histogram/table outputs) — https://github.com/pylelab/RSCanner
- **R** (Runtime environment; RSCanner is distributed as R scripts and an R Markdown notebook requiring R pre-installation) — https://www.r-project.org/
- **tidyverse** (One of the three R package dependencies required by the RSCanner script)
- **dplyr** (One of the three R package dependencies required by the RSCanner script)
- **ggplot2** (Renders the BPC, Shannon, histogram, and heatmap TIFF plots)

## Examples

```
cd sample_outputs && Rscript ../RSCanner_CT_shannon.R ../sample_test_files/HCV_Jc1.ct ../sample_test_files/HCV_shannon.txt
```

## Evaluation signals

- structure_counts.csv exists with exactly four columns (index, structure counts, BPC, smoothed Shannon Entropy).
- BPC and Shannon data files have one row per nucleotide from lower bound (1) to upper bound (transcript length).
- With default parameters, each structured position satisfies both conditions: BPC above the 50th-percentile cutoff AND Shannon entropy below the 50th-percentile cutoff.
- ordered_structure_table.csv has five columns (index, bin number, % structure content, bin start (nt), bin end (nt)) and bins are non-overlapping 100-nt windows tiling the transcript.
- The highest % structure content bins overlap RNA structural elements annotated in a reference figure (e.g., Fig. 1C for HCV_Jc1).

## Limitations

- If an even window size is supplied, RSCanner silently uses the next odd number (e.g., input 50 becomes 51), so centered windows are always used.
- Cutoffs are percentile-based (default 50th percentile) and relative to the transcript, not absolute structure scores; changing bounds or cutoffs changes which positions are flagged.
- Windows are gradually truncated at the RNA 5'/3' ends (e.g., window #26=[1nt,51nt], #25=[1nt,50nt], ...), so end-window statistics rest on fewer nucleotides.
- No version numbers for R or the R packages are specified in the documentation.

## Evidence

- [methods] it computes nucleotide positions with BPC values above a user-defined cutoff (default = 50th percentile) and Shannon entropy values below a user-defined cutoff (default = 50th percentile).: "it computes nucleotide positions with BPC values above a user-defined cutoff (default = 50th percentile) and Shannon entropy values below a user-defined cutoff (default = 50th percentile)."
- [methods] RSCanner uses a centered sliding window (default=51nt, 25nt flanks to each side of the center nucleotide, moved in steps of 1 nt across the RNA) to calculate BPC and median Shannon Entropy.: "RSCanner uses a centered sliding window (default=51nt, 25nt flanks to each side of the center nucleotide, moved in steps of 1 nt across the RNA) to calculate BPC and median Shannon Entropy."
- [readme] When dealing with the ends, RSCanner will gradually truncate the window size (i.e., for 5' end windows - window #26=[1nt,51nt], window#25=[1nt,50nt], window#24 =[1nt,49nt], etc): "When dealing with the ends, RSCanner will gradually truncate the window size (i.e., for 5' end windows - window #26=[1nt,51nt], window#25=[1nt,50nt], window#24 =[1nt,49nt], etc"
- [readme] If an even number window size is inputted, RSCanner will use the next odd number as the window size (i.e., if input=50, the actual window size=51).: "If an even number window size is inputted, RSCanner will use the next odd number as the window size (i.e., if input=50, the actual window size=51)."
- [readme] structure_counts.csv                     - output tab delimited file, col1=index, col2=structure counts, col3=BPC, col4=smoothed Shannon Entropy: "structure_counts.csv                     - output tab delimited file, col1=index, col2=structure counts, col3=BPC, col4=smoothed Shannon Entropy"
