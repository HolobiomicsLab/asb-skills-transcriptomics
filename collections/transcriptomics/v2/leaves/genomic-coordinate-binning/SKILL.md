---
name: genomic-coordinate-binning
description: Use when per-nucleotide scores (e.g. RSCanner structure counts) must
  be aggregated into fixed non-overlapping windows along a transcript to rank regions,
  e.g. locating high-structure-density segments in a long RNA.
when_to_use_negative:
- When per-nucleotide resolution is required (e.g. mapping individual base-paired
  positions) — binning to 100-nt windows deliberately aggregates away single-nucleotide
  detail.
- When the input is already a binned/region-level table (e.g. an existing ordered_structure_table.csv)
  rather than per-nucleotide structure counts.
- When regions of interest have irregular or biologically defined boundaries rather
  than the fixed non-overlapping windows RSCanner tiles along the RNA.
edam_operation: http://edamontology.org/operation_3801
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_0097
tools:
- name: RSCanner
  role: Performs the sliding-window scan and outputs the per-nucleotide structure
    counts that are then binned into non-overlapping windows; also renders the binned
    heatmap and histogram plots.
  repo: https://github.com/pylelab/RSCanner
- name: R
  role: Computing language in which RSCanner's binning scripts (RSCanner_CT_shannon.R
    / RSCanner_dotbracket_shannon.R) are implemented and run from the terminal.
- name: tidyverse
  role: R package dependency used by the RSCanner scripts for data manipulation during
    binning and table generation.
- name: dplyr
  role: R package dependency used by the RSCanner scripts for aggregating per-nucleotide
    counts into bins and ordering bins by % structure content.
- name: ggplot2
  role: R package dependency used to render the binned structure-count density as
    the final heatmap/histogram TIFF figures.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad111
    title: 'RSCanner: rapid assessment and visualization of RNA structure content'
  source_content_hash: sha256:35f0275ae0d8da2a899a103bf1f41f7d70d70f3f51e59eb5919010344add2cc4
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
  edam_operation: http://edamontology.org/operation_3801
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_0097
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# genomic-coordinate-binning

## Summary

Tiles a transcript with non-overlapping fixed-length coordinate bins (RSCanner default: 100 nt) and aggregates per-nucleotide structure counts into percent structure content per bin, enabling ranked comparison of sub-regions across an RNA such as the HCV_Jc1 genome.

## When to use

Use when per-nucleotide scores (e.g. RSCanner structure counts) must be aggregated into fixed non-overlapping windows along a transcript to rank regions, e.g. locating high-structure-density segments in a long RNA.

## When NOT to use

- When per-nucleotide resolution is required (e.g. mapping individual base-paired positions) — binning to 100-nt windows deliberately aggregates away single-nucleotide detail.
- When the input is already a binned/region-level table (e.g. an existing ordered_structure_table.csv) rather than per-nucleotide structure counts.
- When regions of interest have irregular or biologically defined boundaries rather than the fixed non-overlapping windows RSCanner tiles along the RNA.

## Inputs

- structure_counts.csv (col1=index, col2=structure counts, col3=BPC, col4=smoothed Shannon Entropy) from RSCanner_CT_shannon.R or RSCanner_dotbracket_shannon.R
- Transcript length / coordinate bounds (user-defined integer lower bound=1 and upper bound=length of transcript)
- Bin size parameter (default 100 nt)

## Outputs

- ordered_structure_table.csv (col1=index, col2=bin number, col3=% structure content, col4=bin start (nt), col5=bin end (nt))
- Binned structure-count density used in structure_counts_heatmap.tiff (line plot with 1D heatmap overlay, default 7 x 3 inches, 300 dpi)
- structure_counts_histogram.tiff (frequency distribution of binned structure counts)

## How to apply

After RSCanner's sliding-window scan (51-nt centered windows, 1-nt step, truncated at RNA ends) produces structure_counts.csv (index, structure counts, BPC, smoothed Shannon Entropy), tile the transcript with non-overlapping bins of the default 100-nt window length (user-defined 'Integer window length for histogram and heatmap: 100'). For each bin, sum the per-nucleotide structure counts and divide by bin length to compute percent structure content. Rank bins by descending percent structure content and write ordered_structure_table.csv with columns: col1=index, col2=bin number, col3=% structure content, col4=bin start (nt), col5=bin end (nt). These binned counts also drive the frequency-distribution heatmap/histogram outputs; the binning rationale is to convert noisy per-nucleotide signals into discrete regions comparable across a long RNA. The last bin may be shorter than 100 nt if the transcript length is not divisible by the bin size.

## Related tools

- **RSCanner** (Performs the sliding-window scan and outputs the per-nucleotide structure counts that are then binned into non-overlapping windows; also renders the binned heatmap and histogram plots.) — https://github.com/pylelab/RSCanner
- **R** (Computing language in which RSCanner's binning scripts (RSCanner_CT_shannon.R / RSCanner_dotbracket_shannon.R) are implemented and run from the terminal.)
- **tidyverse** (R package dependency used by the RSCanner scripts for data manipulation during binning and table generation.)
- **dplyr** (R package dependency used by the RSCanner scripts for aggregating per-nucleotide counts into bins and ordering bins by % structure content.)
- **ggplot2** (R package dependency used to render the binned structure-count density as the final heatmap/histogram TIFF figures.)

## Examples

```
mkdir sample_outputs && cd sample_outputs && Rscript ../RSCanner_CT_shannon.R ../sample_test_files/HCV_Jc1.ct ../sample_test_files/HCV_shannon.txt
```

## Evaluation signals

- ordered_structure_table.csv exists with exactly five columns: index, bin number, % structure content, bin start (nt), bin end (nt).
- Bins tile the transcript non-overlapping: consecutive bin start/end coordinates are contiguous (bin start = previous bin end + 1), covering the user-defined lower bound (1) to upper bound (transcript length).
- Each bin's % structure content equals summed structure counts divided by bin length; verify ordering is strictly descending by % structure content.
- structure_counts_heatmap.tiff renders at default 7 x 3 inches and 300 dpi and its density overlay matches the binned counts.
- For HCV_Jc1, the highest-scoring bins in ordered_structure_table.csv overlap the RNA structural elements flagged in the reference figure (Fig. 1C).

## Limitations

- Bin size is user-defined (default 100 nt); the choice of bin length changes which regions rank highest, and results are sensitive to this parameter.
- The final bin can be truncated if the transcript length is not a multiple of the bin size, complicating direct comparison of % structure content across bins.
- Structure counts depend on upstream percentile cutoffs (default 50th percentile for both BPC and Shannon entropy), so binning results inherit sensitivity to those cutoffs.
- No version numbers for R or the R packages are stated in the article, so exact reproducibility of the binning environment is not guaranteed.

## Evidence

- [readme] Binning into non-overlapping 100-nt windows with percent structure content: "Integer window length for histogram and heatmap: 100"
- [readme] Ordered bin table schema: "ordered_structure_table.csv              - output tab delimited file, col1=index, col2=bin number, col3=% structure content, col4=bin start (nt), col5=bin end (nt)"
- [methods] RSCanner outputs binned frequency distribution as heatmap and histogram: "RSCanner then outputs the frequency distribution of structure counts across the RNA as both heatmap and histogram plots."
- [readme] Default image parameters for the binned figure: "Integer width of images: 7<br/>
Integer height of images: 3<br/>
Integer resolution of images (dpi): 300"
- [readme] Transcript bounds for binning: "Integer lower bound: 1<br/>
Integer upper bound: length of transcript"
