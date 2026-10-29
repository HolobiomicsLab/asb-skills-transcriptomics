---
name: structure-counts-classification
description: Use when per-nucleotide BPC and smoothed Shannon entropy values are already
  computed for an RNA (e.g. HCV_Jc1 from a .ct or dot-bracket file plus a shannon.txt),
  and positions must be labeled as structure counts under RSCanner's filtering rules.
when_to_use_negative:
- Only the raw .ct/dot-bracket structure file and raw shannon.txt are available —
  the sliding-window BPC and Shannon-entropy smoothing step must run first to produce
  bpc_data.csv and smoothed_Shannon_data.csv.
- The goal is the final heatmap/histogram visualization rather than the per-nucleotide
  classification itself — use the downstream frequency-distribution plotting step.
- The RNA is short or the user wants absolute (e.g. fixed BPC) thresholds rather than
  relative global percentile cutoffs — the default 50th-percentile rule is data-relative
  and may not match other filtering schemes.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_0092
tools:
- name: RSCanner
  role: Performs the sliding-window BPC/Shannon entropy calculation and applies the
    dual percentile-cutoff filter to designate structure counts
  repo: https://github.com/pylelab/RSCanner
- name: R
  role: Computing environment in which the classification and cutoff logic run
  repo: https://www.r-project.org/
- name: dplyr
  role: R package dependency used for the data filtering/manipulation behind the cutoffs
- name: ggplot2
  role: R package dependency used for plotting the downstream frequency distribution
    of structure counts
- name: tidyverse
  role: R package dependency of the RSCanner script that wraps the classification
    logic
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad111
    title: 'RSCanner: rapid assessment and visualization of RNA structure content'
  source_content_hash: sha256:8ae29488e4d6d905aa566fd54481e23352bb497533b2b716ac1a78c3ca92fca4
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
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# structure-counts-classification

## Summary

Classifies individual nucleotides of an RNA transcript as 'structure counts' by applying dual percentile cutoffs to sliding-window base pair content (BPC) and smoothed Shannon entropy values, then tabulating the passing positions in structure_counts.csv.

## When to use

Use when per-nucleotide BPC and smoothed Shannon entropy values are already computed for an RNA (e.g. HCV_Jc1 from a .ct or dot-bracket file plus a shannon.txt), and positions must be labeled as structure counts under RSCanner's filtering rules.

## When NOT to use

- Only the raw .ct/dot-bracket structure file and raw shannon.txt are available — the sliding-window BPC and Shannon-entropy smoothing step must run first to produce bpc_data.csv and smoothed_Shannon_data.csv.
- The goal is the final heatmap/histogram visualization rather than the per-nucleotide classification itself — use the downstream frequency-distribution plotting step.
- The RNA is short or the user wants absolute (e.g. fixed BPC) thresholds rather than relative global percentile cutoffs — the default 50th-percentile rule is data-relative and may not match other filtering schemes.

## Inputs

- bpc_data.csv — per-nucleotide base pair content (col1=index, col2=nucleotide number, col3=BPC)
- smoothed_Shannon_data.csv — per-nucleotide smoothed Shannon entropy (col1=index, col2=nucleotide number, col3=smoothed Shannon Entropy)
- Shannon entropy percentile cutoff (default 0.5)
- BPC percentile cutoff (default 0.5)

## Outputs

- structure_counts.csv — col1=index, col2=structure counts, col3=BPC, col4=smoothed Shannon Entropy
- Input to structure_counts_histogram.tiff and structure_counts_heatmap.tiff (frequency distribution of structure counts, default 100-nt window)

## How to apply

Load the per-nucleotide BPC values (bpc_data.csv) and smoothed Shannon entropy values (smoothed_Shannon_data.csv) produced by RSCanner's centered 51-nt sliding window (default 25-nt flanks, step 1 nt, truncated at RNA ends). Compute the global 50th-percentile cutoff for BPC and the global 50th-percentile cutoff for smoothed Shannon entropy — these are RSCanner's user-defined defaults (0.5). Classify a nucleotide position as a structure count when its BPC value is above the BPC percentile cutoff AND its smoothed Shannon entropy is below the Shannon entropy percentile cutoff, since structured positions show high base pairing paired with low positional entropy. Write the passing (and per-position) results to structure_counts.csv with columns: index, structure counts, BPC, and smoothed Shannon Entropy. These structure counts then feed the downstream frequency-distribution histogram (default 100-nt window, 7x3 in, 300 dpi).

## Related tools

- **RSCanner** (Performs the sliding-window BPC/Shannon entropy calculation and applies the dual percentile-cutoff filter to designate structure counts) — https://github.com/pylelab/RSCanner
- **R** (Computing environment in which the classification and cutoff logic run) — https://www.r-project.org/
- **dplyr** (R package dependency used for the data filtering/manipulation behind the cutoffs)
- **ggplot2** (R package dependency used for plotting the downstream frequency distribution of structure counts)
- **tidyverse** (R package dependency of the RSCanner script that wraps the classification logic)

## Examples

```
Rscript ../RSCanner_CT_shannon.R ../sample_test_files/HCV_Jc1.ct ../sample_test_files/HCV_shannon.txt
```

## Evaluation signals

- structure_counts.csv exists with exactly four columns: index, structure counts, BPC, smoothed Shannon Entropy.
- Every row flagged as a structure count has BPC > global 50th-percentile BPC cutoff AND smoothed Shannon entropy < global 50th-percentile entropy cutoff; rows failing either condition are not counts.
- Number of flagged structure counts equals the size of the intersection of positions above the BPC cutoff and positions below the entropy cutoff.
- structure_counts_histogram.tiff is produced using the default 100-nt window at 7x3 inches and 300 dpi, and its per-window counts aggregate the per-position structure counts.
- Even-numbered window inputs must be bumped to the next odd size (input=50 → 51) before any cutoffs are recomputed on the smoothed values.

## Limitations

- Cutoffs are user-defined but default to the 50th percentile of the transcript's own BPC and Shannon entropy distributions, so classification is relative to the analyzed RNA rather than absolute.
- If an even window size is supplied, RSCanner silently uses the next odd number (input=50 → 51), which shifts window geometry.
- Windows at RNA ends are gradually truncated (e.g. 5' end windows window #26=[1nt,51nt], #25=[1nt,50nt], …), so end positions have window statistics computed over fewer nucleotides.
- No version numbers are stated for R or the required packages (tidyverse, dplyr, ggplot2).

## Evidence

- [methods] Dual percentile filter definition: "it computes nucleotide positions with BPC values above a user-defined cutoff (default = 50th percentile) and Shannon entropy values below a user-defined cutoff (default = 50th percentile). These"
- [readme] Structure counts naming: "These positions are termed structure counts. RSCanner then outputs the frequency distribution of structure counts across the RNA as both heatmap and histogram plots."
- [methods] Sliding window parameters: "RSCanner uses a centered sliding window (default=51nt, 25nt flanks to each side of the center nucleotide, moved in steps of 1 nt across the RNA) to calculate BPC and median Shannon Entropy."
- [readme] Output file schema: "structure_counts.csv - output tab delimited file, col1=index, col2=structure counts, col3=BPC, col4=smoothed Shannon Entropy"
- [readme] Percentile cutoff parameters: "Shannon Entropy percentile cutoff, decimal between 0 and 1: 0.5<br/>BPC percentile cutoff, decimal between 0 and 1: 0.5"
