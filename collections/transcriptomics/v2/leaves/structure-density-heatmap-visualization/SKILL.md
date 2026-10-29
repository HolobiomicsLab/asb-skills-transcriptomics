---
name: structure-density-heatmap-visualization
description: Use when you have per-nucleotide BPC and Shannon entropy (e.g. from a
  .ct or dot-bracket file plus shannon.txt) and need to map where structure-dense
  regions fall along the transcript.
when_to_use_negative:
- Input lacks a predicted secondary structure or per-nucleotide Shannon entropy —
  RSCanner requires both a .ct/.dot-bracket structure and a shannon.txt file, so it
  cannot scan raw sequences alone.
- You need 3D or base-resolution structure validation rather than a windowed 1D density
  map of structure content along the transcript.
- You need overlapped/stacked feature windows rather than fixed non-overlapping bins
  — RSCanner's heatmap bins use a default 100-nt non-overlapping window.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: RSCanner
  role: R package that scans structure + Shannon entropy, computes structure counts,
    and generates the heatmap and ranked bin table
  repo: https://github.com/pylelab/RSCanner
- name: R
  role: computing language in which RSCanner scripts and Rmd notebook are run
  repo: https://www.r-project.org/
- name: ggplot2
  role: renders the final structure-count heatmap/line plot TIFF and histogram plots
- name: dplyr
  role: data manipulation dependency for computing structure counts and ordered bin
    tables
- name: tidyverse
  role: data wrangling dependency of the RSCanner script
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad111
    title: 'RSCanner: rapid assessment and visualization of RNA structure content'
  source_content_hash: sha256:31597d1dff8636b068ab1ed898071dc3ddb7874c9bc81fe07768d81be4af40b8
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
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# structure-density-heatmap-visualization

## Summary

Visualize the density of RNA secondary structure across a long transcript by binning RSCanner structure counts (high BPC, low Shannon entropy positions) into fixed non-overlapping windows and rendering a final line plot with a 1D heatmap overlay, plus a ranked table of % structure content per bin.

## When to use

Use when you have per-nucleotide BPC and Shannon entropy (e.g. from a .ct or dot-bracket file plus shannon.txt) and need to map where structure-dense regions fall along the transcript.

## When NOT to use

- Input lacks a predicted secondary structure or per-nucleotide Shannon entropy — RSCanner requires both a .ct/.dot-bracket structure and a shannon.txt file, so it cannot scan raw sequences alone.
- You need 3D or base-resolution structure validation rather than a windowed 1D density map of structure content along the transcript.
- You need overlapped/stacked feature windows rather than fixed non-overlapping bins — RSCanner's heatmap bins use a default 100-nt non-overlapping window.

## Inputs

- RNA secondary structure in CT (connectivity table) format: six columns after one header line (HCV_Jc1.ct)
- RNA secondary structure in dot-bracket notation FASTA format (HCV_Jc1.dot)
- Positional Shannon entropy as a two-column tab-delimited file, col1 = index, col2 = Shannon entropy values with no header (HCV_shannon.txt)

## Outputs

- structure_counts_heatmap.tiff — final line plot with 1D heatmap overlay of structure-count density (default 7 x 3 inches, 300 dpi)
- structure_counts_histogram.tiff — histogram of structure counts
- structure_counts.csv — col1=index, col2=structure counts, col3=BPC, col4=smoothed Shannon Entropy
- ordered_structure_table.csv — col1=index, col2=bin number, col3=% structure content, col4=bin start (nt), col5=bin end (nt)

## How to apply

Run RSCanner (RSCanner_CT_shannon.R for CT input, or RSCanner_dotbracket_shannon.R for dot-bracket FASTA) with the default 51-nt centered sliding window (25-nt flanks, 1-nt step, truncated at the RNA ends) to compute BPC and median smoothed Shannon entropy per window. Define structure counts as nucleotide positions with BPC above the 50th percentile cutoff and Shannon entropy below the 50th percentile cutoff, written to structure_counts.csv (index, structure counts, BPC, smoothed Shannon Entropy). Bin these counts into non-overlapping 100-nt windows tiling the transcript (e.g. HCV_Jc1), computing percent structure content per bin, then order bins by descending % structure content into ordered_structure_table.csv (index, bin number, % structure content, bin start nt, bin end nt). Render the final line plot with 1D heatmap overlay as structure_counts_heatmap.tiff via ggplot2 (default 7 x 3 inches, 300 dpi). The rationale is that structure counts concentrate where stable paired regions co-occur with low positional entropy, so windowed density pinpoints high-structure regions in long RNAs.

## Related tools

- **RSCanner** (R package that scans structure + Shannon entropy, computes structure counts, and generates the heatmap and ranked bin table) — https://github.com/pylelab/RSCanner
- **R** (computing language in which RSCanner scripts and Rmd notebook are run) — https://www.r-project.org/
- **ggplot2** (renders the final structure-count heatmap/line plot TIFF and histogram plots)
- **dplyr** (data manipulation dependency for computing structure counts and ordered bin tables)
- **tidyverse** (data wrangling dependency of the RSCanner script)

## Examples

```
mkdir sample_outputs && cd sample_outputs && Rscript ../RSCanner_CT_shannon.R ../sample_test_files/HCV_Jc1.ct ../sample_test_files/HCV_shannon.txt
```

## Evaluation signals

- structure_counts.csv exists with the exact schema: col1=index, col2=structure counts, col3=BPC, col4=smoothed Shannon Entropy
- ordered_structure_table.csv exists with columns: index, bin number, % structure content, bin start (nt), bin end (nt), with bins in descending % structure content order
- structure_counts_heatmap.tiff and structure_counts_histogram.tiff are produced at the specified image dimensions (7 x 3 inches, 300 dpi)
- Highest-scoring bins in ordered_structure_table.csv overlap known RNA structural elements flagged in the reference figure (e.g. Fig. 1C for HCV_Jc1)
- Structure counts only include positions with BPC above the 50th percentile and Shannon entropy below the 50th percentile cutoffs, per the user-defined parameters (0.5 default for both)

## Limitations

- Cutoffs for BPC and Shannon entropy are user-defined percentiles (default 0.5), so structure-count density — and thus heatmap bins — shifts if different percentiles are chosen.
- Window truncation at the RNA 5' and 3' ends means end windows have progressively fewer nucleotides, so structure content at transcript ends is estimated from smaller samples.
- No version numbers for R or the R packages are stated, so exact numeric reproduction of the heatmap may vary across R/tidyverse versions.

## Evidence

- [abstract] RSCanner then generates an intuitive heatmap enabling users to rapidly pinpoint regions likely to contain a high or low density of discrete RNA structures: "RSCanner then generates an intuitive heatmap enabling users to rapidly pinpoint regions likely to contain a high or low density of discrete RNA structures"
- [methods] BPC above cutoff, Shannon entropy below cutoff defines structure counts: "it computes nucleotide positions with BPC values above a user-defined cutoff (default = 50th percentile) and Shannon entropy values below a user-defined cutoff (default = 50th percentile)"
- [methods] heatmap and histogram outputs of structure counts: "RSCanner then outputs the frequency distribution of structure counts across the RNA as both heatmap and histogram plots."
- [readme] ordered_structure_table.csv schema from README usage block: "ordered_structure_table.csv              - output tab delimited file, col1=index, col2=bin number, col3=% structure content, col4=bin start (nt), col5=bin end (nt)"
- [readme] window truncation at RNA ends: "When dealing with the ends, RSCanner will gradually truncate the window size (i.e., for 5' end windows - window #26=[1nt,51nt], window#25=[1nt,50nt], window#24 =[1nt,49nt], etc)"
