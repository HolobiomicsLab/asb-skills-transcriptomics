---
name: empty-droplet-removal-inflection-point
description: Use when flexiplex discovery mode has produced a read-barcode assignment
  and barcode count file (flexiplex_barcodes_counts.txt) from long-read single-cell
  FASTQ and true cell barcodes must be separated from empty-droplet barcodes.
when_to_use_negative:
- When cell barcodes are already known and a curated barcode list is provided — run
  flexiplex directly in known-barcode mode (e.g. `-d 10x3v3 -k barcode_list.txt`)
  rather than discovery plus filtering.
- When the input is not single-cell droplet data with empty droplets (e.g. bulk reads
  or plate-based single-cell data with no droplet barcode frequency curve), inflection-point
  empty-droplet removal does not apply.
edam_operation: http://edamontology.org/operation_3220
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: flexiplex-filter
  role: Filters discovered barcodes against a whitelist and removes empty droplets
    using an inflection point method
  repo: https://davidsongroup.github.io/flexiplex/
- name: flexiplex
  role: Runs barcode discovery mode (`-f 0`) to produce flexiplex_barcodes_counts.txt
    consumed by flexiplex-filter; optionally re-run with `-k` to assign reads to filtered
    barcodes
  repo: https://github.com/DavidsonGroup/flexiplex
- name: conda
  role: Installation channel for flexiplex (`conda install -c bioconda -c conda-forge
    flexiplex`)
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae102
    title: 'Flexiplex: a versatile demultiplexer and search tool for omics data'
  source_content_hash: sha256:bc0ac7ff7284cece42c48967e5fec8aa772696fe33bf5b9c635907ecbba3304a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Flexiplex
  - conda
  - flexiplex
  - flexiplex-filter
  edam_operation: http://edamontology.org/operation_3220
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# empty-droplet-removal-inflection-point

## Summary

Identifies real cells among discovered cellular barcodes by filtering candidate barcodes against a whitelist and removing empty droplets using an inflection point method, as implemented by Flexiplex's flexiplex-filter in barcode discovery workflows on noisy long-read single-cell data.

## When to use

Use when flexiplex discovery mode has produced a read-barcode assignment and barcode count file (flexiplex_barcodes_counts.txt) from long-read single-cell FASTQ and true cell barcodes must be separated from empty-droplet barcodes.

## When NOT to use

- When cell barcodes are already known and a curated barcode list is provided — run flexiplex directly in known-barcode mode (e.g. `-d 10x3v3 -k barcode_list.txt`) rather than discovery plus filtering.
- When the input is not single-cell droplet data with empty droplets (e.g. bulk reads or plate-based single-cell data with no droplet barcode frequency curve), inflection-point empty-droplet removal does not apply.

## Inputs

- flexiplex_barcodes_counts.txt (read-barcode assignments and barcode counts from flexiplex discovery mode)
- barcode whitelist file (e.g. 3M-february-2018.txt short-read-derived whitelist)

## Outputs

- filtered.txt (list of whitelist-matching barcodes after empty-droplet removal via inflection point)
- new_reads.fastq (reads demultiplexed/assigned to the filtered cell barcodes, optional re-run)

## How to apply

After running flexiplex in discovery mode (e.g. `flexiplex -f 0 reads.fastq`), which writes barcode counts and read-barcode assignments to flexiplex_barcodes_counts.txt, run flexiplex-filter with a short-read-derived barcode whitelist (e.g. 3M-february-2018.txt for the 248-cell scmixology 2 benchmark) to keep only whitelist-matching barcodes. flexiplex-filter then removes empty droplets using an inflection point method on the barcode frequency distribution, writing the retained barcodes to filtered.txt. Optionally re-run flexiplex with a preset chemistry (e.g. `-d 10x3v3`) and `-k filtered.txt` barcode list to assign reads to the filtered cell barcodes. The rationale is that discovered barcodes include empty-droplet noise, so whitelist intersection plus the inflection point knee in the barcode count curve distinguishes genuine cells from ambient barcodes, as validated on scmixology 2 where Flexiplex reported the highest number of reads with concordant cell-line assignment (<5% discordant).

## Related tools

- **flexiplex-filter** (Filters discovered barcodes against a whitelist and removes empty droplets using an inflection point method) — https://davidsongroup.github.io/flexiplex/
- **flexiplex** (Runs barcode discovery mode (`-f 0`) to produce flexiplex_barcodes_counts.txt consumed by flexiplex-filter; optionally re-run with `-k` to assign reads to filtered barcodes) — https://github.com/DavidsonGroup/flexiplex
- **conda** (Installation channel for flexiplex (`conda install -c bioconda -c conda-forge flexiplex`))

## Examples

```
flexiplex-filter --whitelist 3M-february-2018.txt --outfile filtered.txt flexiplex_barcodes_counts.txt
```

## Evaluation signals

- flexiplex_barcodes_counts.txt exists and is non-empty before running flexiplex-filter (it is the required input).
- filtered.txt contains only barcodes present in the whitelist file after empty-droplet removal.
- new_reads.fastq exists after the optional re-run with `-k filtered.txt`, containing reads assigned to the filtered barcodes.
- Downstream concordance check: on scmixology 2-style benchmarks with known cell-line composition, <5% of reads should have discordant cell-line assignment.
- The inflection point (knee) in the barcode frequency distribution should fall at a barcode count consistent with the expected number of loaded cells (e.g. ~248 cells in scmixology 2).

## Limitations

- The inflection point method relies on a clear knee in the barcode count distribution; datasets without a sharp separation between real and empty droplets may misclassify barcodes.
- Whitelist filtering assumes a short-read-derived whitelist is available and matches the chemistry; barcodes absent from the whitelist (e.g. novel or heavily errored barcodes) are discarded.
- Flexiplex allows imperfect matches based on Levenshtein distance, so very high edit-distance settings could let erroneous barcodes through the discovery stage before filtering.

## Evidence

- [methods] Flexiplex can search for barcodes, filter against a whitelist, and remove empty droplets using an inflection point method.: "Flexiplex can search for barcodes, filter against a whitelist, and remove empty droplets using an inflection point method."
- [methods] flexiplex-filter command: "flexiplex-filter --whitelist 3M-february-2018.txt --outfile filtered.txt flexiplex_barcodes_counts.txt"
- [results] scmixology 2 concordance finding: [evidence span withheld — anchor status: not_found]
- [readme] README barcode discovery description: "**Barcode discovery directly from long reads**  
Flexiplex can search for barcodes, filter against a whitelist, and remove empty droplets using an inflection point method."
