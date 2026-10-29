---
name: empty-droplet-filtering-single-cell
description: Use when you have a barcode frequency count table (e.g. flexiplex_barcodes_counts.txt)
  from single-cell long-read or short-read data and need to distinguish real cell
  barcodes from empty droplets before demultiplexing.
when_to_use_negative:
- Barcodes are already known and supplied in a barcode_list.txt for known-sequence
  demultiplexing mode (flexiplex -k), so no empty-droplet cutoff estimation is needed
- Input is raw FASTQ/FASTA reads rather than a barcode frequency count table — run
  barcode discovery first
- The chemistry preset (e.g. 10x3v3) already enforces a whitelist, making an inflection-point
  cutoff redundant
edam_operation: http://edamontology.org/operation_3220
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: flexiplex-filter
  role: Applies the whitelist filter and inflection-point empty-droplet cutoff to
    the barcode counts file
  repo: https://github.com/DavidsonGroup/flexiplex
- name: Python 3.9
  role: Computes the rolling-window smoothing and numeric derivative of the knee plot
    to locate the inflection point
- name: uv
  role: Package manager used to run flexiplex-filter scripts via uvx from the flexiplex
    repository
  repo: https://github.com/davidsongroup/flexiplex
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae102
    title: 'Flexiplex: a versatile demultiplexer and search tool for omics data'
  source_content_hash: sha256:5b58e1fbae46f76204045ce6fa8ca355a79aed60d7e89779c3ef2ee5cb1aba40
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DavidsonGroup/flexiplex
  tools:
  - Flexiplex
  - Python
  - Python 3.9
  - flexiplex-filter
  - uv
  edam_operation: http://edamontology.org/operation_3220
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# empty-droplet-filtering-single-cell

## Summary

Removes empty droplets from discovered cellular barcodes by filtering the barcode count table against a whitelist and cutting at the knee-plot inflection point, as implemented by flexiplex-filter in the Flexiplex demultiplexing pipeline.

## When to use

Use when you have a barcode frequency count table (e.g. flexiplex_barcodes_counts.txt) from single-cell long-read or short-read data and need to distinguish real cell barcodes from empty droplets before demultiplexing.

## When NOT to use

- Barcodes are already known and supplied in a barcode_list.txt for known-sequence demultiplexing mode (flexiplex -k), so no empty-droplet cutoff estimation is needed
- Input is raw FASTQ/FASTA reads rather than a barcode frequency count table — run barcode discovery first
- The chemistry preset (e.g. 10x3v3) already enforces a whitelist, making an inflection-point cutoff redundant

## Inputs

- flexiplex_barcodes_counts.txt (barcode count table from flexiplex barcode discovery)
- 3M-february-2018.txt (10x barcode whitelist)

## Outputs

- filtered.txt (barcodes passing the inflection-point cutoff and whitelist)
- knee_plot.png (barcode count frequency-distribution plot with inflection point marked)

## How to apply

Load the barcode count table produced by the upstream barcode-discovery step (flexiplex_barcodes_counts.txt), rank barcodes by read frequency, and build the knee plot of frequency versus rank. Smooth the curve with a rolling window and numerically approximate its derivative (e.g. numpy.gradient in Python 3.9); the rank where the derivative is most negative is taken as the inflection-point cutoff for empty-droplet removal. Run flexiplex-filter with --whitelist 3M-february-2018.txt on the counts file, using --outfile to write barcodes that pass both the inflection-point cutoff and the whitelist. Optionally render a knee plot (knee_plot.png) with the inflection point marked to verify the cutoff visually.

## Related tools

- **flexiplex-filter** (Applies the whitelist filter and inflection-point empty-droplet cutoff to the barcode counts file) — https://github.com/DavidsonGroup/flexiplex
- **Python 3.9** (Computes the rolling-window smoothing and numeric derivative of the knee plot to locate the inflection point)
- **uv** (Package manager used to run flexiplex-filter scripts via uvx from the flexiplex repository) — https://github.com/davidsongroup/flexiplex

## Examples

```
uvx --from git+https://github.com/davidsongroup/flexiplex.git#subdirectory=scripts flexiplex-filter --whitelist 3M-february-2018.txt --outfile filtered.txt flexiplex_barcodes_counts.txt
```

## Evaluation signals

- Knee plot (knee_plot.png) shows a marked inflection point falling at the elbow of the ranked barcode frequency curve, not at an extreme rank
- filtered.txt contains only barcodes present in flexiplex_barcodes_counts.txt that also appear in the 3M-february-2018.txt whitelist
- Number of barcodes retained is consistent with the expected cell number for the library (barcodes above the inflection point are kept; the long negative-derivative tail below is dropped)
- The most-negative-derivative rank computed from the smoothed curve matches the cutoff visible at the plot's elbow

## Limitations

- The inflection point is an approximation from a numerically differentiated, rolling-window-smoothed knee curve, so noisy count distributions can shift the cutoff
- Filtering against 3M-february-2018.txt restricts barcodes to the 10x whitelist; novel or non-10x barcodes are excluded even if above the cutoff
- The cutoff is data-dependent and must be re-estimated per barcode count table rather than reused across libraries

## Evidence

- [methods] Flexiplex can search for barcodes, filter against a whitelist, and remove empty droplets using an inflection point method.: "Flexiplex can search for barcodes, filter against a whitelist, and remove empty droplets using an inflection point method."
- [methods] flexiplex-filter invocation: "flexiplex-filter --whitelist 3M-february-2018.txt --outfile filtered.txt flexiplex_barcodes_counts.txt"
- [other] knee-plot derivative method: [evidence span withheld — anchor status: not_found]
- [methods] uv-run scripts: "uvx --from git+https://github.com/davidsongroup/flexiplex.git#subdirectory=scripts"
- [readme] Barcode discovery from long reads: "Flexiplex can search for barcodes, filter against a whitelist, and remove empty droplets using an inflection point method."
