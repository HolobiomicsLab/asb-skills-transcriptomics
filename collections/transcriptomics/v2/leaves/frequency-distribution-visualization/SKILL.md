---
name: frequency-distribution-visualization
description: Use when you have RSCanner structure counts (positions above BPC percentile
  cutoff and below Shannon entropy percentile cutoff) and need to visualize how structural
  content is distributed along the transcript.
when_to_use_negative:
- The transcript's secondary structure and Shannon entropy inputs have not yet been
  scanned by RSCanner, so no structure counts exist to bin into a frequency distribution
- The goal is a position-resolved heatmap of structural content rather than an aggregate
  histogram (use the heatmap output instead)
- You need raw BPC or smoothed Shannon entropy line plots (bpcplot.tiff / smoothed_Shannonplot.tiff)
  rather than structure count frequencies
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0097
- http://edamontology.org/topic_0092
tools:
- name: RSCanner
  role: computes structure counts by sliding-window BPC and Shannon entropy filtering,
    then outputs the frequency distribution as histogram and heatmap plots
  repo: https://github.com/pylelab/RSCanner
- name: ggplot2
  role: R graphics package dependency used to render the structure counts histogram
    TIFF
- name: R
  role: computing environment in which RSCanner scripts run the histogram generation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad111
    title: 'RSCanner: rapid assessment and visualization of RNA structure content'
  source_content_hash: sha256:09a08044b012a62285e7815b349aa72bf5f893372a1c94b584f5831c1732d861
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
  - http://edamontology.org/topic_0097
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# frequency-distribution-visualization

## Summary

Renders the frequency distribution of RSCanner 'structure counts' (nucleotides passing BPC and Shannon entropy percentile cutoffs) across an RNA transcript as a histogram, using ggplot2 over a default 100-nt window.

## When to use

Use when you have RSCanner structure counts (positions above BPC percentile cutoff and below Shannon entropy percentile cutoff) and need to visualize how structural content is distributed along the transcript.

## When NOT to use

- The transcript's secondary structure and Shannon entropy inputs have not yet been scanned by RSCanner, so no structure counts exist to bin into a frequency distribution
- The goal is a position-resolved heatmap of structural content rather than an aggregate histogram (use the heatmap output instead)
- You need raw BPC or smoothed Shannon entropy line plots (bpcplot.tiff / smoothed_Shannonplot.tiff) rather than structure count frequencies

## Inputs

- structure_counts.csv (tab delimited: col1=index, col2=structure counts, col3=BPC, col4=smoothed Shannon Entropy)
- User-defined histogram window length (default 100 nt)
- Image parameters: width 7, height 3, resolution 300 dpi

## Outputs

- structure_counts_histogram.tiff — histogram figure of structure count frequency across the RNA

## How to apply

After RSCanner classifies nucleotide positions as structure counts (BPC above a user-defined cutoff, default 50th percentile, and Shannon entropy below a user-defined cutoff, default 50th percentile), compute the frequency of structure counts across the RNA in windows of 100 nt (the default histogram window). Render the distribution with ggplot2 as structure_counts_histogram.tiff using the default image settings of 7 inches width, 3 inches height, and 300 dpi. The histogram complements the structure_counts_heatmap.tiff output, both describing how discrete RNA structural regions cluster along the transcript; for HCV_Jc1 the histogram is shown in Supplementary Fig. S2C. Validation relies on both structure_counts.csv (index, structure count, BPC, smoothed Shannon Entropy columns) and the histogram TIFF being produced.

## Related tools

- **RSCanner** (computes structure counts by sliding-window BPC and Shannon entropy filtering, then outputs the frequency distribution as histogram and heatmap plots) — https://github.com/pylelab/RSCanner
- **ggplot2** (R graphics package dependency used to render the structure counts histogram TIFF)
- **R** (computing environment in which RSCanner scripts run the histogram generation)

## Examples

```
mkdir sample_outputs; cd sample_outputs; Rscript ../RSCanner_CT_shannon.R ../sample_test_files/HCV_Jc1.ct ../sample_test_files/HCV_shannon.txt
```

## Evaluation signals

- structure_counts_histogram.tiff is produced alongside structure_counts_heatmap.tiff, bpcplot.tiff, and smoothed_Shannonplot.tiff
- structure_counts.csv exists with the four documented columns: index, structure counts, BPC, smoothed Shannon Entropy
- Histogram binning uses the default 100-nt window; image dimensions are 7x3 inches at 300 dpi
- Positions binned into the histogram match the subset in structure_counts.csv where BPC exceeds the 50th-percentile cutoff AND smoothed Shannon entropy falls below the 50th-percentile cutoff
- Re-running on the sample HCV_Jc1.ct + HCV_shannon.txt inputs reproduces the Supplementary Fig. S2C-style histogram

## Limitations

- If an even window size is supplied, RSCanner uses the next odd number (e.g. input 50 becomes 51), so histogram/heatmap window lengths must be handled accordingly
- The histogram aggregates over a 100-nt window and does not resolve individual nucleotide-level structure counts; use the heatmap or CSV for positional detail
- No version numbers are stated for R or the required packages (tidyverse, dplyr, ggplot2), which may affect reproducibility

## Evidence

- [methods] it computes nucleotide positions with BPC values above a user-defined cutoff (default = 50th percentile) and Shannon entropy values below a user-defined cutoff: "it computes nucleotide positions with BPC values above a user-defined cutoff (default = 50th percentile) and Shannon entropy values below a user-defined cutoff (default = 50th percentile)"
- [methods] RSCanner then outputs the frequency distribution of structure counts across the RNA as both heatmap and histogram plots.: "RSCanner then outputs the frequency distribution of structure counts across the RNA as both heatmap and histogram plots."
- [readme] structure_counts_histogram.tiff          - output histogram figure: "structure_counts_histogram.tiff          - output histogram figure"
- [readme] Integer window length for histogram and heatmap: 100: "Integer window length for histogram and heatmap: 100"
