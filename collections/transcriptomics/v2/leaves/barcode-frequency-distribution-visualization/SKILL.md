---
name: barcode-frequency-distribution-visualization
description: Use when a barcode count table from Flexiplex barcode discovery needs
  visual inspection of the frequency-versus-rank curve to verify the empty-droplet
  cutoff before filtering.
when_to_use_negative:
- Barcodes are already known and assigned (known-sequence mode) — use flexiplex with
  a barcode list (e.g. -k barcode_list.txt) rather than discovery plus knee-plot filtering.
- The count table is not from Flexiplex barcode discovery (e.g. an already-filtered
  whitelist such as 3M-february-2018.txt), so no knee structure exists to segment.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: flexiplex-filter
  role: consumes the barcode count table; applies the inflection-point cutoff visualized
    in the knee plot to produce the filtered barcode list
  repo: https://github.com/DavidsonGroup/flexiplex
- name: Python
  role: rolling-window smoothing, numerical derivative (numpy gradient) of the knee
    curve, and matplotlib rendering of knee_plot.png
- name: Flexiplex
  role: upstream barcode-discovery step that generates flexiplex_barcodes_counts.txt
    for the plot
  repo: https://github.com/DavidsonGroup/flexiplex
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae102
    title: 'Flexiplex: a versatile demultiplexer and search tool for omics data'
  source_content_hash: sha256:0993f0e0e3129eb535b812284f06478ca05a7c0906248e01656490328a8dc263
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DavidsonGroup/flexiplex
  tools:
  - Flexiplex
  - Python
  - flexiplex-filter
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# barcode-frequency-distribution-visualization

## Summary

Renders a knee plot of barcode read-frequency versus rank from a flexiplex_barcodes_counts.txt table, marking the computed inflection-point cutoff used by flexiplex-filter to remove empty droplets.

## When to use

Use when a barcode count table from Flexiplex barcode discovery needs visual inspection of the frequency-versus-rank curve to verify the empty-droplet cutoff before filtering.

## When NOT to use

- Barcodes are already known and assigned (known-sequence mode) — use flexiplex with a barcode list (e.g. -k barcode_list.txt) rather than discovery plus knee-plot filtering.
- The count table is not from Flexiplex barcode discovery (e.g. an already-filtered whitelist such as 3M-february-2018.txt), so no knee structure exists to segment.

## Inputs

- flexiplex_barcodes_counts.txt (barcode count table from Flexiplex barcode discovery)

## Outputs

- knee_plot.png (barcode count frequency-distribution plot with inflection point marked)
- computed inflection-point rank/cutoff used for barcode filtering

## How to apply

Load the barcode count table (flexiplex_barcodes_counts.txt) produced by the upstream Flexiplex barcode-discovery step and rank barcodes by read frequency to build the knee plot. Smooth the frequency-versus-rank curve with a rolling window and compute its derivative numerically (e.g. numpy gradient in Python 3.9). Select the rank at which the derivative is most negative as the inflection point, which is the cutoff flexiplex-filter uses for empty-droplet removal. Render the frequency-distribution (knee) plot of barcode counts with the computed inflection point marked and save it as knee_plot.png. Inspect the marked cutoff against the curve shape to confirm it falls at the knee between true cell barcodes and empty droplets.

## Related tools

- **flexiplex-filter** (consumes the barcode count table; applies the inflection-point cutoff visualized in the knee plot to produce the filtered barcode list) — https://github.com/DavidsonGroup/flexiplex
- **Python** (rolling-window smoothing, numerical derivative (numpy gradient) of the knee curve, and matplotlib rendering of knee_plot.png)
- **Flexiplex** (upstream barcode-discovery step that generates flexiplex_barcodes_counts.txt for the plot) — https://github.com/DavidsonGroup/flexiplex

## Examples

```
uvx --from git+https://github.com/davidsongroup/flexiplex.git#subdirectory=scripts flexiplex-filter --whitelist 3M-february-2018.txt --outfile filtered.txt flexiplex_barcodes_counts.txt
```

## Evaluation signals

- knee_plot.png exists and shows a monotone decreasing frequency-versus-rank curve with a clearly marked inflection point
- The marked inflection rank equals the rank where the smoothed curve's derivative is most negative
- The barcodes passing the cutoff match the contents of filtered.txt produced by flexiplex-filter with the same count table
- The rank-frequency curve exhibits the expected knee shape separating high-frequency cell barcodes from low-frequency empty-droplet barcodes

## Limitations

- The inflection-point cutoff is an approximation from a numerically differentiated, rolling-window-smoothed curve; noisy count tables may shift the detected knee.
- Barcodes must also pass the whitelist (e.g. 3M-february-2018.txt) check in flexiplex-filter; the knee cutoff alone does not guarantee a valid 10x barcode.

## Evidence

- [readme] Flexiplex can search for barcodes, filter against a whitelist, and remove empty droplets using an inflection point method.: "Flexiplex can search for barcodes, filter against a whitelist, and remove empty droplets using an inflection point method."
- [methods] flexiplex-filter invocation with whitelist and count table: "flexiplex-filter --whitelist 3M-february-2018.txt --outfile filtered.txt flexiplex_barcodes_counts.txt"
- [methods] uvx installation path for the scripts subdirectory: "uvx --from git+https://github.com/davidsongroup/flexiplex.git#subdirectory=scripts"
- [methods] Python 3.9 requirement for the flexiplex scripts: "ensure that `python3.9`, and `python-pip` are installed, then run: `make install`"
