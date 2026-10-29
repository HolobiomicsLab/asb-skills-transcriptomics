---
name: shannon-entropy-filtering
description: Use when per-nucleotide structure calls must be derived from RSCanner
  sliding-window BPC and median Shannon entropy values, e.g. to select low-entropy/high-BPC
  positions from a CT or dot-bracket structure scan of HCV_Jc1 or any long RNA.
when_to_use_negative:
- No positional Shannon entropy file (shannon.txt) is available — the filter requires
  per-nucleotide entropy values in addition to the structure file, so RSCanner cannot
  compute smoothed entropy from the CT/dot-bracket input alone.
- The goal is de novo RNA structure prediction rather than filtering an existing structure
  — RSCanner takes a pre-computed secondary structure (CT or dot-bracket) and entropy,
  and does not fold or infer structure.
- An absolute (non-percentile) entropy threshold is required — the cutoffs are percentiles
  of the transcript's own distributions, not fixed entropy values, so the filter is
  not calibrated across independent transcripts.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0097
tools:
- name: RSCanner
  role: Implements the centered sliding-window BPC/median-Shannon-entropy calculation
    and applies the dual BPC-above / SE-below percentile filter to emit structure
    counts
  repo: github.com/pylelab/RSCanner
- name: R
  role: Runtime environment; RSCanner ships as R scripts (RSCanner_CT_shannon.R, RSCanner_dotbracket_shannon.R)
    and an Rmd notebook
- name: tidyverse
  role: One of the three required R package dependencies for the RSCanner scripts
- name: dplyr
  role: One of the three required R package dependencies used for filtering/tabulating
    structure counts
- name: ggplot2
  role: One of the three required R package dependencies used for output plots (SE
    line plot, heatmap, histogram)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad111
    title: 'RSCanner: rapid assessment and visualization of RNA structure content'
  source_content_hash: sha256:d0fa1c879b10c36193a85bd29035746a059bb3dd482386f296292ec984233a37
schema_version: 0.2.0
metadata:
  license_tier: open
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
yes_rate: 0.6
n_claims: 10
---

# shannon-entropy-filtering

## Summary

Filters nucleotide positions by keeping only those whose smoothed Shannon entropy falls below a user-defined percentile cutoff while base pair content (BPC) exceeds its percentile cutoff — the structure-count computation step of RSCanner used to isolate ordered, stable regions of an RNA secondary structure.

## When to use

Use when per-nucleotide structure calls must be derived from RSCanner sliding-window BPC and median Shannon entropy values, e.g. to select low-entropy/high-BPC positions from a CT or dot-bracket structure scan of HCV_Jc1 or any long RNA.

## When NOT to use

- No positional Shannon entropy file (shannon.txt) is available — the filter requires per-nucleotide entropy values in addition to the structure file, so RSCanner cannot compute smoothed entropy from the CT/dot-bracket input alone.
- The goal is de novo RNA structure prediction rather than filtering an existing structure — RSCanner takes a pre-computed secondary structure (CT or dot-bracket) and entropy, and does not fold or infer structure.
- An absolute (non-percentile) entropy threshold is required — the cutoffs are percentiles of the transcript's own distributions, not fixed entropy values, so the filter is not calibrated across independent transcripts.

## Inputs

- CT format secondary structure file (six columns after one header line), e.g. HCV_Jc1.ct
- Dot-bracket FASTA structure file (for RSCanner_dotbracket_shannon.R), e.g. HCV_Jc1.dot
- Tab-delimited shannon.txt: col1 = index, col2 = per-nucleotide Shannon entropy values, no header (e.g. HCV_shannon.txt)
- User-defined parameters: Shannon Entropy percentile cutoff (decimal 0–1, default 0.5), BPC percentile cutoff (default 0.5), integer window size (default 51), 100-nt histogram/heatmap bin width

## Outputs

- structure_counts.csv (col1=index, col2=structure counts, col3=BPC, col4=smoothed Shannon Entropy)
- smoothed_Shannon_data.csv (col1=index, col2=nucleotide number, col3=smoothed Shannon Entropy)
- smoothed_Shannonplot.tiff (smoothed SE line plot with global-median dashed line)
- ordered_structure_table.csv (index, bin number, % structure content, bin start/end nt)

## How to apply

Run RSCanner's centered sliding window (default 51 nt, 25-nt flanks, 1-nt step) over the input structure (CT or dot-bracket) plus the tab-delimited shannon.txt entropy file to obtain per-window BPC and smoothed Shannon entropy. A nucleotide position passes the filter when its BPC is above a user-defined percentile cutoff (default = 50th) AND its smoothed Shannon entropy is below a user-defined percentile cutoff (default = 50th, given as a decimal between 0 and 1 at the prompt); passing positions are termed 'structure counts' and are written to structure_counts.csv (index, structure counts, BPC, smoothed Shannon Entropy). To increase stringency, lower the SE percentile cutoff (e.g. 0.75 → 0.25) and/or raise the BPC cutoff, as demonstrated in Supplementary Fig. S3A-E; varying only the SE cutoff while holding BPC at 0.50 isolates the entropy filter's effect. If an even window size is supplied, RSCanner silently uses the next odd number so the window is always centered; windows at the 5'/3' ends are gradually truncated.

## Related tools

- **RSCanner** (Implements the centered sliding-window BPC/median-Shannon-entropy calculation and applies the dual BPC-above / SE-below percentile filter to emit structure counts) — github.com/pylelab/RSCanner
- **R** (Runtime environment; RSCanner ships as R scripts (RSCanner_CT_shannon.R, RSCanner_dotbracket_shannon.R) and an Rmd notebook)
- **tidyverse** (One of the three required R package dependencies for the RSCanner scripts)
- **dplyr** (One of the three required R package dependencies used for filtering/tabulating structure counts)
- **ggplot2** (One of the three required R package dependencies used for output plots (SE line plot, heatmap, histogram))

## Examples

```
Rscript ../RSCanner_CT_shannon.R ../sample_test_files/HCV_Jc1.ct ../sample_test_files/HCV_shannon.txt
```

## Evaluation signals

- The 0.50/0.50 BPC/SE cutoff run must reproduce the default-parameter structure counts for the HCV_Jc1 sample test files.
- Every retained position in structure_counts.csv must have smoothed Shannon entropy below the SE cutoff AND BPC above the BPC percentile cutoff.
- structure_counts.csv must contain exactly the documented columns (index, structure counts, BPC, smoothed Shannon Entropy); ordered_structure_table.csv must contain index, bin number, % structure content, bin start, bin end.
- As the SE cutoff decreases (0.75 → 0.50 → 0.25) with BPC held fixed, the set and count of structure counts and per-100-nt-bin % structure content must shrink monotonically (increasing stringency).
- If an even window size was supplied, verify the effective window is the next odd number so a centered sliding window was used in all calculations.

## Limitations

- Cutoffs are transcript-relative percentiles (default 50th), not absolute entropy thresholds, so stringency comparisons across RNAs are not directly calibrated.
- No version numbers are stated for R or the tidyverse/dplyr/ggplot2 packages, and no explicit GitHub URL for the repository is given in the methods text.
- The R scripts auto-install missing packages, which can pull unpinned CRAN package versions; there is no changelog documenting reproducibility gaps.
- Percentile cutoffs are decimals between 0 and 1 entered at the interactive prompt; non-decimal or out-of-range values are not covered by the documentation.

## Evidence

- [methods] Structure counts (BPC above percentile cutoff and Shannon entropy below percentile cutoff): "it computes nucleotide positions with BPC values above a user-defined cutoff (default = 50th percentile) and Shannon entropy values below a user-defined cutoff (default = 50th percentile). These"
- [readme] Centered sliding window parameters: "RSCanner uses a centered sliding window (default=51nt, 25nt flanks to each side of the center nucleotide, moved in steps of 1 nt across the RNA) to calculate BPC and median Shannon Entropy."
- [readme] SE cutoff prompt and default: "Shannon Entropy percentile cutoff, decimal between 0 and 1: 0.5"
- [readme] structure_counts.csv schema: "structure_counts.csv                     - output tab delimited file, col1=index, col2=structure counts, col3=BPC, col4=smoothed Shannon Entropy"
- [supplementary] Stringency effect of varying SE cutoff: [evidence span withheld — anchor status: not_found]
- [readme] Even window size handling: "If an even number window size is inputted, RSCanner will use the next odd number as the window size (i.e., if input=50, the actual window size=51)."
