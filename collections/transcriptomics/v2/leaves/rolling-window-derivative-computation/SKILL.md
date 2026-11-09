---
name: rolling-window-derivative-computation
description: Use when you need to convert a barcode count table (flexiplex_barcodes_counts.txt)
  into a filtered whitelist by finding the knee/inflection point of the barcode frequency-versus-rank
  curve, as in empty-droplet removal for single-cell data.
when_to_use_negative:
- Barcodes are already known (known-sequence mode) and only demultiplexing with flexiplex
  -d is needed — no empty-droplet filtering is required.
- The count data are not barcode frequencies ranked by abundance (e.g. gene count
  matrices), where the knee-plot inflection logic does not apply.
- Input barcodes should be corrected against a supplied whitelist only, without data-driven
  cutoff estimation.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0621
tools:
- name: flexiplex-filter
  role: Applies the inflection-point cutoff and whitelist filtering to the barcode
    count table, writing passing barcodes to the outfile
  repo: https://github.com/DavidsonGroup/flexiplex
- name: Flexiplex
  role: Upstream tool in discovery mode that produces flexiplex_barcodes_counts.txt,
    the input to the knee-plot filtering step
  repo: https://github.com/DavidsonGroup/flexiplex
- name: Python 3.9
  role: Environment for computing the rolling-window smoothing and numerical derivative
    (e.g. numpy gradient) of the knee plot
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae102
    title: 'Flexiplex: a versatile demultiplexer and search tool for omics data'
  source_content_hash: sha256:d7a773b0e11707d2215355f839625e22b99f30b521fb279408a81950c2ad48a0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DavidsonGroup/flexiplex
  tools:
  - Flexiplex
  - Python
  - Python 3.9
  - flexiplex-filter
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# rolling-window-derivative-computation

## Summary

Numerically estimates the derivative of a rank-frequency (knee plot) curve using a rolling window to locate its most negative slope, which flexiplex-filter uses as the inflection-point cutoff for removing empty droplets from discovered cellular barcodes.

## When to use

Use when you need to convert a barcode count table (flexiplex_barcodes_counts.txt) into a filtered whitelist by finding the knee/inflection point of the barcode frequency-versus-rank curve, as in empty-droplet removal for single-cell data.

## When NOT to use

- Barcodes are already known (known-sequence mode) and only demultiplexing with flexiplex -d is needed — no empty-droplet filtering is required.
- The count data are not barcode frequencies ranked by abundance (e.g. gene count matrices), where the knee-plot inflection logic does not apply.
- Input barcodes should be corrected against a supplied whitelist only, without data-driven cutoff estimation.

## Inputs

- flexiplex_barcodes_counts.txt (barcode frequency table from flexiplex discovery mode)
- Whitelist file (e.g. 3M-february-2018.txt) of known 10x barcodes

## Outputs

- filtered.txt (barcodes passing the inflection-point cutoff and whitelist)
- knee_plot.png (frequency-distribution plot of barcode counts with the inflection point marked)

## How to apply

Load the barcode count table produced by flexiplex in discovery mode and rank barcodes by read frequency to construct the knee plot. Smooth the frequency-versus-rank curve with a rolling window and compute its derivative numerically (e.g. numpy.gradient in Python 3.9). Select the rank at which the smoothed derivative is most negative — this is the inflection point, used as the barcode-count cutoff separating real cells from empty droplets. Invoke flexiplex-filter with --whitelist (e.g. 3M-february-2018.txt) on flexiplex_barcodes_counts.txt and --outfile filtered.txt so that only barcodes above the inflection-point cutoff and present in the whitelist are retained. The rationale is that the knee of the ranked count distribution marks the transition between high-count cellular barcodes and low-count ambient/empty-droplet barcodes.

## Related tools

- **flexiplex-filter** (Applies the inflection-point cutoff and whitelist filtering to the barcode count table, writing passing barcodes to the outfile) — https://github.com/DavidsonGroup/flexiplex
- **Flexiplex** (Upstream tool in discovery mode that produces flexiplex_barcodes_counts.txt, the input to the knee-plot filtering step) — https://github.com/DavidsonGroup/flexiplex
- **Python 3.9** (Environment for computing the rolling-window smoothing and numerical derivative (e.g. numpy gradient) of the knee plot)

## Examples

```
flexiplex-filter --whitelist 3M-february-2018.txt --outfile filtered.txt flexiplex_barcodes_counts.txt
```

## Evaluation signals

- The most negative point of the smoothed derivative curve lies on the visually identifiable knee of the barcode frequency-versus-rank plot.
- filtered.txt contains only barcodes whose counts exceed the inflection-point cutoff and that appear in the provided whitelist (e.g. 3M-february-2018.txt).
- knee_plot.png renders the barcode count distribution with the computed inflection point marked at the cutoff rank.
- The number of retained barcodes is consistent with the expected cell count for the library (barcodes above the knee, ambient barcodes below removed).

## Limitations

- The inflection-point method is an approximation: the derivative is estimated numerically from a rolling-window-smoothed curve, so results depend on window choice and may misplace the knee for atypical count distributions.
- The method assumes a clear separation between real-cell and empty-droplet barcodes; libraries with shallow sequencing or ambiguous knees may yield unreliable cutoffs.

## Evidence

- [methods] flexiplex-filter inflection point: "flexiplex-filter --whitelist 3M-february-2018.txt --outfile filtered.txt flexiplex_barcodes_counts.txt"
- [methods] filter steps from article: "Flexiplex can search for barcodes, filter against a whitelist, and remove empty droplets using an inflection point method."
- [readme] README barcode discovery usage: "Flexiplex can search for barcodes, filter against a whitelist, and remove empty droplets using an inflection point method."
- [other] task finding on derivative: [evidence span withheld — anchor status: not_found]
- [methods] uv invocation from methods: "uvx --from git+https://github.com/davidsongroup/flexiplex.git#subdirectory=scripts"
