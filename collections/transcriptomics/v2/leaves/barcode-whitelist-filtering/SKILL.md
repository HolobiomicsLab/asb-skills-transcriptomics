---
name: barcode-whitelist-filtering
description: Use when you have a barcode count table from barcode discovery on single-cell
  reads (FASTQ/FASTA) and need to keep only whitelist-matching barcodes and drop empty
  droplets before assigning reads to cells.
when_to_use_negative:
- The barcodes of interest are already known (e.g. the You et al. 2023 hiPSC set of
  1022 true + 10000 decoy barcodes) — skip discovery/filtering and run flexiplex directly
  in known-barcode mode with -k.
- Input is a count matrix or already-assigned BAM with cell barcodes — filtering operates
  on barcode frequency counts from discovery mode, not on downstream expression outputs.
edam_operation: http://edamontology.org/operation_3220
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: flexiplex
  role: Runs barcode discovery mode (-f 0) on the FASTQ to produce the read-barcode
    assignments and barcode counts consumed by flexiplex-filter; can be re-run in
    known-barcode mode with -k and the filtered list
  repo: https://davidsongroup.github.io/flexiplex/
- name: flexiplex-filter
  role: Filters discovered barcodes against the whitelist and removes empty droplets
    via the knee-plot inflection-point cutoff, writing filtered.txt
  repo: https://github.com/davidsongroup/flexiplex
provenance:
  source_task_ids:
  - task_004
  - task_003
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae102
    title: 'Flexiplex: a versatile demultiplexer and search tool for omics data'
  source_content_hash: sha256:7f3e450883daa1d524300152dfcae57e9bd5801dd16e6547e93f3f78d226a083
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FLAMES
  - Flexiplex
  - Python
  - conda
  - flexiplex
  - flexiplex-filter
  - scTagger
  edam_operation: http://edamontology.org/operation_3220
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# barcode-whitelist-filtering

## Summary

Filter discovered cellular barcodes against a known whitelist (e.g. 3M-february-2018.txt) and remove empty droplets using flexiplex-filter's knee-plot inflection-point method, producing a high-confidence filtered barcode list for downstream demultiplexing.

## When to use

Use when you have a barcode count table from barcode discovery on single-cell reads (FASTQ/FASTA) and need to keep only whitelist-matching barcodes and drop empty droplets before assigning reads to cells.

## When NOT to use

- The barcodes of interest are already known (e.g. the You et al. 2023 hiPSC set of 1022 true + 10000 decoy barcodes) — skip discovery/filtering and run flexiplex directly in known-barcode mode with -k.
- Input is a count matrix or already-assigned BAM with cell barcodes — filtering operates on barcode frequency counts from discovery mode, not on downstream expression outputs.

## Inputs

- Long-read single-cell FASTQ file (e.g. ONT reads) for the barcode discovery step
- flexiplex_barcodes_counts.txt — barcode frequency/count table produced by flexiplex discovery mode
- Barcode whitelist text file (e.g. 3M-february-2018.txt 10x whitelist)

## Outputs

- filtered.txt — whitelist-matching barcodes surviving the inflection-point empty-droplet cutoff
- flexiplex_barcodes_counts.txt — intermediate read-barcode assignment and count table
- new_reads.fastq — reads reassigned to the filtered barcodes (optional known-barcode re-run)

## How to apply

First run flexiplex in barcode discovery mode (e.g. `flexiplex -f 0 reads.fastq`) on the long-read FASTQ to produce flexiplex_barcodes_counts.txt, a table of read-barcode assignments and barcode counts. Then run flexiplex-filter with `--whitelist 3M-february-2018.txt --outfile filtered.txt` on that count table: it ranks barcodes by read frequency to build a knee plot, smooths the frequency-versus-rank curve with a rolling window, computes its numerical derivative, and selects the rank where the derivative is most negative as the inflection-point cutoff. Barcodes are kept only if they pass this cutoff AND match the whitelist; the result written to filtered.txt contains only real, whitelist-supported cellular barcodes. The filtered list can then be supplied back to flexiplex in known-barcode mode (e.g. with `-d 10x3v3 -k filtered.txt`) to assign reads to cells. This two-stage discovery-then-filter design is what Flexiplex uses to 'search for barcodes, filter against a whitelist, and remove empty droplets using an inflection point method'.

## Related tools

- **flexiplex** (Runs barcode discovery mode (-f 0) on the FASTQ to produce the read-barcode assignments and barcode counts consumed by flexiplex-filter; can be re-run in known-barcode mode with -k and the filtered list) — https://davidsongroup.github.io/flexiplex/
- **flexiplex-filter** (Filters discovered barcodes against the whitelist and removes empty droplets via the knee-plot inflection-point cutoff, writing filtered.txt) — https://github.com/davidsongroup/flexiplex

## Examples

```
flexiplex -f 0 reads.fastq; flexiplex-filter --whitelist 3M-february-2018.txt --outfile filtered.txt flexiplex_barcodes_counts.txt
```

## Evaluation signals

- flexiplex_barcodes_counts.txt exists and is a non-empty count table produced by the discovery step before filtering
- filtered.txt contains only barcodes present in the supplied whitelist (e.g. 3M-february-2018.txt) after empty-droplet removal
- The number of barcodes in filtered.txt is markedly smaller than the total discovered barcodes, consistent with an inflection-point cutoff on the knee plot
- A rendered knee plot shows the computed inflection point at the rank where the smoothed frequency curve has its most negative derivative
- Downstream re-run with `flexiplex -d 10x3v3 -k filtered.txt` yields new_reads.fastq with reads assigned only to filtered barcodes

## Limitations

- The inflection-point empty-droplet method approximates the knee-plot derivative with a rolling window, so the cutoff is an approximation and may misclassify low-count real cells or high-count empty droplets.
- Filtering is restricted to whitelist-matching barcodes; novel barcodes absent from the whitelist (e.g. older 10x v2 chemistry lists) will be discarded.

## Evidence

- [methods] whitelist + inflection filtering: "flexiplex-filter --whitelist 3M-february-2018.txt --outfile filtered.txt flexiplex_barcodes_counts.txt"
- [methods] inflection point method claim: "Flexiplex can search for barcodes, filter against a whitelist, and remove empty droplets using an inflection point method."
- [other] inflection-point derivative definition: [evidence span withheld — anchor status: not_found]
- [readme] discovery mode command: "Flexiplex can search for barcodes, filter against a whitelist, and remove empty droplets using an inflection point method."
