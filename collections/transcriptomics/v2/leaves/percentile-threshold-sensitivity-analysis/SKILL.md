---
name: percentile-threshold-sensitivity-analysis
description: Use when you need to test how RSCanner's user-defined BPC percentile
  cutoff (raised from 0.25 to 0.75) and Shannon Entropy percentile cutoff (lowered
  from 0.75 to 0.25) change which nucleotide positions are called as structure counts
  in a CT or dot-bracket structure analysis.
when_to_use_negative:
- You only need a single default run (50th/50th percentile cutoffs) without comparing
  stringency levels — just run the standard RSCanner pipeline once.
- The RNA has no precomputed Shannon entropy values and none can be calculated from
  base pairing probabilities — RSCanner requires both a structure file and a shannon.txt.
- You want de novo RNA structure prediction rather than assessment of structural content
  in an already-computed secondary structure.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: RSCanner
  role: 'Primary tool: computes per-window BPC and median Shannon entropy, applies
    the user-defined BPC/SE percentile cutoffs to derive structure counts, and outputs
    CSV tables and heatmap/histogram figures per stringency level'
  repo: https://github.com/pylelab/RSCanner
- name: R
  role: Execution environment; RSCanner is distributed as R scripts (RSCanner_CT_shannon.R,
    RSCanner_dotbracket_shannon.R) and an R Markdown notebook
  repo: https://github.com/pylelab/RSCanner
- name: tidyverse
  role: Required R package dependency of the RSCanner scripts used during threshold-variation
    re-runs
  repo: https://github.com/pylelab/RSCanner
- name: dplyr
  role: Required R package dependency for data manipulation in the RSCanner filtering/tabulation
    steps
  repo: https://github.com/pylelab/RSCanner
- name: ggplot2
  role: Required R package dependency used to render the heatmap and histogram outputs
    across cutoff settings
  repo: https://github.com/pylelab/RSCanner
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad111
    title: 'RSCanner: rapid assessment and visualization of RNA structure content'
  source_content_hash: sha256:902c494f2f6ee491665fe667c0dfcd6faf7f8847ca7142ef5e96ebf520bdf088
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
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# percentile-threshold-sensitivity-analysis

## Summary

A sensitivity-analysis workflow that re-runs RSCanner's structure-count filtering under varying BPC (base pair content) and Shannon Entropy percentile cutoffs to quantify how the set and count of called 'structure counts' shrink as filtering stringency increases.

## When to use

Use when you need to test how RSCanner's user-defined BPC percentile cutoff (raised from 0.25 to 0.75) and Shannon Entropy percentile cutoff (lowered from 0.75 to 0.25) change which nucleotide positions are called as structure counts in a CT or dot-bracket structure analysis.

## When NOT to use

- You only need a single default run (50th/50th percentile cutoffs) without comparing stringency levels — just run the standard RSCanner pipeline once.
- The RNA has no precomputed Shannon entropy values and none can be calculated from base pairing probabilities — RSCanner requires both a structure file and a shannon.txt.
- You want de novo RNA structure prediction rather than assessment of structural content in an already-computed secondary structure.

## Inputs

- RNA secondary structure in CT (connectivity table) format, e.g. HCV_Jc1.ct (six columns after one header line)
- RNA secondary structure in dot-bracket (FASTA) format, e.g. HCV_Jc1.dot (alternative input)
- Positional Shannon entropy as two-column tab-delimited text (col1 = index, col2 = Shannon entropy values, no header), e.g. HCV_shannon.txt
- User-defined percentiles: BPC percentile cutoff (0.25/0.50/0.75) and Shannon Entropy percentile cutoff (0.75/0.50/0.25)

## Outputs

- structure_counts.csv (col1=index, col2=structure counts, col3=BPC, col4=smoothed Shannon Entropy) per cutoff pair
- ordered_structure_table.csv (col1=index, col2=bin number, col3=% structure content, col4=bin start (nt), col5=bin end (nt)) aggregated into 100-nt bins
- bpc_data.csv and smoothed_Shannon_data.csv (per-nucleotide index, nucleotide number, value)
- structure_counts_histogram.tiff and structure_counts_heatmap.tiff showing the frequency distribution of structure counts
- bpcplot.tiff and smoothed_Shannonplot.tiff with dashed lines marking global medians

## How to apply

Load the RNA secondary structure (e.g. HCV_Jc1.ct in CT format) plus its positional Shannon entropy file (HCV_shannon.txt) and run RSCanner's centered sliding-window calculation (51-nt window, 25-nt flanks, 1-nt step) to obtain per-window BPC and median Shannon entropy. Then re-run the structure-count filtering step — positions with BPC above the percentile cutoff and entropy below the percentile cutoff — while varying the BPC percentile cutoff over 0.25, 0.50, and 0.75 paired with the Shannon Entropy percentile cutoff over 0.75, 0.50, and 0.25, keeping other parameters at defaults (51-nt windows, 100-nt histogram/heatmap bins). For each cutoff pair, tabulate nucleotide positions passing both filters into structure_counts.csv and aggregate per-cutoff structure content into 100-nt bins as % structure content in ordered_structure_table.csv. The rationale is that raising the BPC cut-off and lowering the SE cut-off increases filtering stringency, as demonstrated in Supplementary Fig. S3A-E, so the per-cutoff tables reveal exactly which high-BPC/low-entropy positions survive at each stringency. Validate by confirming the 0.50/0.50 run reproduces the default-parameter structure counts.

## Related tools

- **RSCanner** (Primary tool: computes per-window BPC and median Shannon entropy, applies the user-defined BPC/SE percentile cutoffs to derive structure counts, and outputs CSV tables and heatmap/histogram figures per stringency level) — https://github.com/pylelab/RSCanner
- **R** (Execution environment; RSCanner is distributed as R scripts (RSCanner_CT_shannon.R, RSCanner_dotbracket_shannon.R) and an R Markdown notebook) — https://github.com/pylelab/RSCanner
- **tidyverse** (Required R package dependency of the RSCanner scripts used during threshold-variation re-runs) — https://github.com/pylelab/RSCanner
- **dplyr** (Required R package dependency for data manipulation in the RSCanner filtering/tabulation steps) — https://github.com/pylelab/RSCanner
- **ggplot2** (Required R package dependency used to render the heatmap and histogram outputs across cutoff settings) — https://github.com/pylelab/RSCanner

## Examples

```
Rscript ../RSCanner_CT_shannon.R ../sample_test_files/HCV_Jc1.ct ../sample_test_files/HCV_shannon.txt
```

## Evaluation signals

- The 0.50 BPC / 0.50 SE cutoff run must reproduce the default-parameter structure counts for the HCV_Jc1 sample.
- All output CSVs must contain the documented columns: structure_counts.csv = (index, structure counts, BPC, smoothed Shannon Entropy); ordered_structure_table.csv = (index, bin number, % structure content, bin start (nt), bin end (nt)).
- The structure-count set and total count must shrink monotonically as the BPC percentile cutoff rises (0.25→0.75) and/or the Shannon Entropy percentile cutoff falls (0.75→0.25).
- Per-bin % structure content in ordered_structure_table.csv must be computed over the specified 100-nt histogram/heatmap bins for each cutoff pair.
- Window parameters must match defaults (51-nt centered sliding window, 1-nt step, gradual truncation at 5'/3' ends) for all re-runs so stringency is the only varied factor.

## Limitations

- Window size is silently forced to the next odd number if an even window is supplied (e.g. input 50 becomes 51), so comparisons must keep window size consistent.
- At RNA ends RSCanner gradually truncates the window size (e.g. window #26=[1nt,51nt], window #25=[1nt,50nt]), which can affect BPC/entropy values near transcript boundaries.
- No version numbers were stated for R or the R packages, and no explicit limitations were extracted from the article's discussion section.

## Evidence

- [methods] RSCanner structure-count filter definition: "computes nucleotide positions with BPC values above a user-defined cutoff (default = 50th percentile) and Shannon entropy values below a user-defined cutoff (default = 50th percentile)"
- [readme] RSCanner sliding-window BPC/entropy calculation: "RSCanner uses a centered sliding window (default=51nt, 25nt flanks to each side of the center nucleotide, moved in steps of 1 nt across the RNA) to calculate BPC and median Shannon Entropy."
- [readme] User-defined percentile parameters: "Shannon Entropy percentile cutoff, decimal between 0 and 1: 0.5"
- [readme] Histogram/heatmap binning parameter: "Integer window length for histogram and heatmap: 100"
