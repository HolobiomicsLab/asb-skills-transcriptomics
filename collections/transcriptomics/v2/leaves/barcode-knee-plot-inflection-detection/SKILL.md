---
name: barcode-knee-plot-inflection-detection
description: Use when a barcode count table (e.g. flexiplex_barcodes_counts.txt from
  flexiplex in discovery mode) must be filtered to keep only barcodes above the empty-droplet
  knee, and a whitelist such as 3M-february-2018.txt is to be intersected with the
  cutoff.
when_to_use_negative:
- Barcodes are already known (known-sequence demultiplexing mode) so no knee-plot
  cutoff is needed — just run flexiplex with -d 10x3v3 -k barcode_list.txt.
- The count table comes from a non-droplet (plate-based or bespoke) library where
  there is no empty-droplet population to remove.
- Only raw read demultiplexing is required and no cell-barcode calling / empty-droplet
  filtering step is intended.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: flexiplex-filter
  role: Applies the inflection-point cutoff plus whitelist filtering to the barcode
    count table, writing the surviving barcodes to the outfile
  repo: https://github.com/DavidsonGroup/flexiplex
- name: Flexiplex
  role: Upstream tool run in barcode-discovery mode (-f 0) that produces flexiplex_barcodes_counts.txt
    from long-read FASTQ
  repo: https://davidsongroup.github.io/flexiplex/
- name: Python 3.9
  role: Numeric smoothing (rolling window) and derivative computation (e.g. numpy.gradient)
    to locate the most-negative-slope inflection rank
- name: uv
  role: Package manager used to invoke flexiplex-filter scripts via uvx from the flexiplex
    git subdirectory
  repo: https://github.com/davidsongroup/flexiplex
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae102
    title: 'Flexiplex: a versatile demultiplexer and search tool for omics data'
  source_content_hash: sha256:e233e0d8bffd600de84b13ae46451375d3ff7558a5075dba5ad7cd78f1adf3d4
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
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# barcode-knee-plot-inflection-detection

## Summary

Detects the knee-plot inflection point that separates real cellular barcodes from empty droplets in single-cell barcode-discovery output, by numerically differentiating the smoothed barcode frequency-versus-rank curve and selecting the rank with the most negative derivative as the cutoff.

## When to use

Use when a barcode count table (e.g. flexiplex_barcodes_counts.txt from flexiplex in discovery mode) must be filtered to keep only barcodes above the empty-droplet knee, and a whitelist such as 3M-february-2018.txt is to be intersected with the cutoff.

## When NOT to use

- Barcodes are already known (known-sequence demultiplexing mode) so no knee-plot cutoff is needed — just run flexiplex with -d 10x3v3 -k barcode_list.txt.
- The count table comes from a non-droplet (plate-based or bespoke) library where there is no empty-droplet population to remove.
- Only raw read demultiplexing is required and no cell-barcode calling / empty-droplet filtering step is intended.

## Inputs

- flexiplex_barcodes_counts.txt (barcode frequency table from flexiplex barcode-discovery mode)
- 3M-february-2018.txt (10x 737k barcode whitelist)
- Python 3.9 environment with numpy (rolling-window smoothing and numeric derivative)

## Outputs

- filtered.txt (whitelisted barcodes passing the inflection-point cutoff)
- knee_plot.png (barcode count frequency-distribution plot with the inflection point marked)

## How to apply

Load the barcode count table produced by the upstream flexiplex barcode-discovery run and rank barcodes by read frequency to build the knee plot. Smooth the frequency-versus-rank curve with a rolling window and compute its derivative numerically (e.g. numpy.gradient in Python 3.9), since a raw discrete curve is too noisy for direct derivative estimation. Select the rank at which the derivative is most negative as the inflection point — this is the knee separating true cell barcodes from empty-droplet droplets per the method Flexiplex uses to remove empty droplets. Then run flexiplex-filter with --whitelist 3M-february-2018.txt on flexiplex_barcodes_counts.txt so barcodes passing both the inflection cutoff and the 10x 737k whitelist are written via --outfile (filtered.txt). Finally render the frequency-distribution knee plot with the inflection point marked and save as knee_plot.png for visual QA.

## Related tools

- **flexiplex-filter** (Applies the inflection-point cutoff plus whitelist filtering to the barcode count table, writing the surviving barcodes to the outfile) — https://github.com/DavidsonGroup/flexiplex
- **Flexiplex** (Upstream tool run in barcode-discovery mode (-f 0) that produces flexiplex_barcodes_counts.txt from long-read FASTQ) — https://davidsongroup.github.io/flexiplex/
- **Python 3.9** (Numeric smoothing (rolling window) and derivative computation (e.g. numpy.gradient) to locate the most-negative-slope inflection rank)
- **uv** (Package manager used to invoke flexiplex-filter scripts via uvx from the flexiplex git subdirectory) — https://github.com/davidsongroup/flexiplex

## Examples

```
uvx --from git+https://github.com/davidsongroup/flexiplex.git#subdirectory=scripts flexiplex-filter --whitelist 3M-february-2018.txt --outfile filtered.txt flexiplex_barcodes_counts.txt
```

## Evaluation signals

- knee_plot.png shows the computed inflection point lying on the visible knee of the log-scale barcode frequency-versus-rank curve, above the long flat tail of low-frequency barcodes.
- filtered.txt contains only barcodes that are both present in 3M-february-2018.txt and at ranks above (higher count than) the inflection-point rank.
- The number of barcodes kept is in the expected range for the library (e.g. roughly the expected number of loaded cells for a 10x run), not orders of magnitude off.
- The most-negative-derivative rank is stable to small changes in the rolling-window width (sanity check on smoothing parameter choice).
- flexiplex-filter exits cleanly via uvx and the outfile is non-empty, one barcode per line.

## Limitations

- The derivative-based inflection detection is an approximation of the knee; strong smoothing choices can shift the cutoff rank and misclassify borderline barcodes.
- The cutoff depends on the upstream flexiplex discovery-mode output (flexiplex_barcodes_counts.txt); errors in barcode discovery propagate into the knee plot.
- Whitelist intersection (3M-february-2018.txt) means non-10x or chemistry-mismatched barcodes are discarded regardless of their frequency.

## Evidence

- [methods] empty-droplet inflection method: "flexiplex-filter --whitelist 3M-february-2018.txt --outfile filtered.txt flexiplex_barcodes_counts.txt"
- [methods] inflection point method: "Flexiplex can search for barcodes, filter against a whitelist, and remove empty droplets using an inflection point method."
- [readme] barcode discovery mode: "Flexiplex can search for barcodes, filter against a whitelist, and remove empty droplets using an inflection point method."
- [methods] installation/runtime env: "ensure that `python3.9`, and `python-pip` are installed, then run: `make install`"
