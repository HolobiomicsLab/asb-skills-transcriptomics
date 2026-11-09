---
name: rna-structure-basepair-content-filtering
description: Use when you have per-window BPC values and smoothed Shannon entropy
  values for an RNA (e.g., from HCV_Jc1 CT/dot-bracket structure plus a shannon.txt)
  and must designate structure counts under RSCanner's dual-threshold filtering.
when_to_use_negative:
- You only have raw dot-bracket/CT structure or Shannon entropy input and have not
  yet run RSCanner's sliding-window BPC and entropy smoothing step — run RSCanner
  first to generate bpc_data.csv and smoothed_Shannon_data.csv.
- The goal is the heatmap of % structure content per bin (ordered_structure_table.csv),
  not the per-nucleotide structure counts filter and its histogram.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: RSCanner
  role: implements the sliding-window BPC/entropy calculation, the dual-percentile
    structure counts filter, and the histogram output
  repo: https://github.com/pylelab/RSCanner
- name: R
  role: computing environment in which the RSCanner filter and plots are executed
- name: dplyr
  role: R package dependency used by the RSCanner script for data manipulation of
    structure counts
- name: ggplot2
  role: R package dependency used by the RSCanner script to render the structure counts
    histogram
- name: tidyverse
  role: R package dependency of the RSCanner script
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad111
    title: 'RSCanner: rapid assessment and visualization of RNA structure content'
  source_content_hash: sha256:fc2678b29c941adf02263469f1a9402ab58a9cca56dad91eeae179a54f741607
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
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# rna-structure-basepair-content-filtering

## Summary

Applies RSCanner's structure counts filter, classifying each nucleotide of an RNA transcript as a structure count where its base pair content (BPC) exceeds a percentile cutoff and its smoothed Shannon entropy falls below a percentile cutoff (both default = 50th percentile), then saves the surviving positions and plots their frequency distribution as a histogram.

## When to use

Use when you have per-window BPC values and smoothed Shannon entropy values for an RNA (e.g., from HCV_Jc1 CT/dot-bracket structure plus a shannon.txt) and must designate structure counts under RSCanner's dual-threshold filtering.

## When NOT to use

- You only have raw dot-bracket/CT structure or Shannon entropy input and have not yet run RSCanner's sliding-window BPC and entropy smoothing step — run RSCanner first to generate bpc_data.csv and smoothed_Shannon_data.csv.
- The goal is the heatmap of % structure content per bin (ordered_structure_table.csv), not the per-nucleotide structure counts filter and its histogram.

## Inputs

- bpc_data.csv (per-position BPC values: index, nucleotide number, BPC) from RSCanner sliding-window calculation
- smoothed_Shannon_data.csv (per-position smoothed Shannon entropy values) from RSCanner
- Percentile cutoffs for BPC and Shannon entropy (default 0.5 each) and histogram window length (default 100 nt)

## Outputs

- structure_counts.csv (col1=index, col2=structure counts, col3=BPC, col4=smoothed Shannon Entropy)
- structure_counts_histogram.tiff (frequency distribution of structure counts across the RNA)

## How to apply

Load the BPC values (bpc_data.csv) and smoothed Shannon entropy values (smoothed_Shannon_data.csv) computed by RSCanner's centered sliding window (default 51 nt, 25-nt flanks, 1-nt steps). Apply the dual filter: a nucleotide is a structure count if its BPC is above the user-defined percentile cutoff (default = 50th percentile) AND its Shannon entropy is below the user-defined percentile cutoff (default = 50th percentile). Write the surviving positions to structure_counts.csv with columns index, structure counts, BPC, and smoothed Shannon Entropy. Then compute the frequency distribution of structure counts across the RNA using the default 100-nt histogram window and render it as structure_counts_histogram.tiff at the default image settings (7x3 inches, 300 dpi).

## Related tools

- **RSCanner** (implements the sliding-window BPC/entropy calculation, the dual-percentile structure counts filter, and the histogram output) — https://github.com/pylelab/RSCanner
- **R** (computing environment in which the RSCanner filter and plots are executed)
- **dplyr** (R package dependency used by the RSCanner script for data manipulation of structure counts)
- **ggplot2** (R package dependency used by the RSCanner script to render the structure counts histogram)
- **tidyverse** (R package dependency of the RSCanner script)

## Examples

```
Rscript ../RSCanner_CT_shannon.R ../sample_test_files/HCV_Jc1.ct ../sample_test_files/HCV_shannon.txt
```

## Evaluation signals

- structure_counts.csv exists with exactly four columns: index, structure counts, BPC, smoothed Shannon Entropy
- Every row marked as a structure count satisfies BPC above the global 50th-percentile cutoff AND smoothed Shannon entropy below the global 50th-percentile cutoff
- Rows failing either condition are absent from structure_counts.csv
- structure_counts_histogram.tiff is produced at default settings (7x3 inches, 300 dpi) using the default 100-nt histogram window
- Histogram shape matches Supplementary Fig. S2C when run on HCV_Jc1 sample inputs with default thresholds

## Limitations

- If an even window size is input, RSCanner silently uses the next odd number (e.g., input=50 becomes 51), so all calculations always use a centered odd-sized sliding window.
- At transcript ends RSCanner gradually truncates the window size (e.g., 5' end window #26=[1nt,51nt], #25=[1nt,50nt], #24=[1nt,49nt]), so end-nucleotide values are computed on smaller windows.
- Cutoffs are percentile-based and user-defined; changing them away from the 0.5 defaults changes which nucleotides qualify as structure counts.
- No version numbers are stated for R or the dependent R packages.

## Evidence

- [methods] Dual percentile cutoff definition of structure counts: "it computes nucleotide positions with BPC values above a user-defined cutoff (default = 50th percentile) and Shannon entropy values below a user-defined cutoff (default = 50th percentile)."
- [methods] Histogram output of structure counts: "RSCanner then outputs the frequency distribution of structure counts across the RNA as both heatmap and histogram plots."
- [readme] Centered sliding window parameters feeding the filter: "RSCanner uses a centered sliding window (default=51nt, 25nt flanks to each side of the center nucleotide, moved in steps of 1 nt across the RNA)"
- [readme] Default cutoff values prompted from the user: [evidence span withheld — anchor status: not_found]
