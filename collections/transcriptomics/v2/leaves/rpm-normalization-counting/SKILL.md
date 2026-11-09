---
name: rpm-normalization-counting
description: Use when per-sample tncRNA subclass counts (tRF-5, tRF-3, tRF-1, 5'tRH,
  3'tRH, tRNA halves) must be expressed in a library-size-independent unit or reconciled
  against the paper's species-level totals.
when_to_use_negative:
- Reads have not yet been classified as tncRNAs mapping exclusively to the artificial
  chromosomes (masked tRNA plus 50 nt leader/trailer) — normalization must not include
  ambiguous non-tRNA assignments
- Paired-end or long-read data instead of the single-end Illumina sRNA-seq FASTQ inputs
  the tncRNA Toolkit pipeline was built for
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3170
tools:
- name: tncRNA Toolkit
  role: Produces the per-sample tncRNA classification tables (subtypes, loci, read
    counts) that serve as input to RPM normalization and species-level aggregation
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: pandas
  role: Loads classification tables, groups by species and tncRNA class, sums unique-read
    and distinct-locus counts, and writes species_tncRNA_summary.csv
- name: biopython
  role: pairwise2 module records the local alignment of each tncRNA to its sequence
    of origin alongside its RPM value
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:aa838a7f9857dc95659c37add39c01d36325b283c92193274830c354ff368af7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/skbinfo/tncRNA-Toolkit
  repo_url: https://github.com/skbinfo/tncRNA-Toolkit
  tools:
  - Bowtie1
  - NCBI SRA Toolkit (v2.10)
  - biopython
  - pandas
  - python3
  - samtools
  - tRNAscan-SE (v2.0.6)
  - tncRNA Toolkit
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7894736842105263
n_claims: 19
---

# rpm-normalization-counting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Normalizes plant tncRNA abundances to Reads Per Million (RPM = reads mapped to progenitor tRNA ÷ PMF, where PMF = total mapped reads/10^6) and aggregates per-sample tncRNA counts into per-species, per-subtype summary tables.

## When to use

Use when per-sample tncRNA subclass counts (tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH, tRNA halves) must be expressed in a library-size-independent unit or reconciled against the paper's species-level totals.

## When NOT to use

- Reads have not yet been classified as tncRNAs mapping exclusively to the artificial chromosomes (masked tRNA plus 50 nt leader/trailer) — normalization must not include ambiguous non-tRNA assignments
- Paired-end or long-read data instead of the single-end Illumina sRNA-seq FASTQ inputs the tncRNA Toolkit pipeline was built for

## Inputs

- Per-sample tncRNA classification output tables from the tncRNAs.py output directory (read counts and locus information per classified read)
- Species labels and tncRNA subtype annotations per read

## Outputs

- RPM-normalized abundance value for each tncRNA
- species_tncRNA_summary.csv aggregating unique reads and distinct loci per species and per tncRNA class
- Local alignment of each tncRNA to its sequence of origin (biopython pairwise2)

## How to apply

After the tncRNAs.py classification step, load the per-sample output tables with pandas and group classified reads by species and tncRNA class (tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH), summing both unique-read counts and distinct locus counts. For expression-level aggregation, compute RPM = number of reads mapped to the progenitor tRNA divided by PMF, where PMF = total mapped reads/10^6, exactly following the paper's formula. Record each tncRNA's local alignment to its sequence of origin with the pairwise2 module of biopython alongside its RPM value. Compile the per-species totals into a single summary table written as species_tncRNA_summary.csv. Validate by checking that row counts per species and class sums reconcile with the reference values reported in the article (tncRNAs from 2,448 sRNA-seq samples across six angiosperms).

## Related tools

- **tncRNA Toolkit** (Produces the per-sample tncRNA classification tables (subtypes, loci, read counts) that serve as input to RPM normalization and species-level aggregation) — https://github.com/skbinfo/tncRNA-Toolkit
- **pandas** (Loads classification tables, groups by species and tncRNA class, sums unique-read and distinct-locus counts, and writes species_tncRNA_summary.csv)
- **biopython** (pairwise2 module records the local alignment of each tncRNA to its sequence of origin alongside its RPM value)

## Examples

```
python3 tncRNAs.py -s < species name > -i < processed small RNA reads > -o < output dir >
```

## Evaluation signals

- Each tncRNA carries both a subtype label and an RPM value computed as reads mapped to progenitor tRNA ÷ PMF, with PMF = total mapped reads/10^6
- Aggregated species-level totals in species_tncRNA_summary.csv reconcile with article reference values (e.g. Arabidopsis thaliana 1,676 tncRNAs; O. sativa 243; Z. mays 221)
- Only reads mapping exclusively to the artificial chromosomes (masked tRNA plus 50 nt flanks) contribute to the summed counts
- Sum of class-level counts (tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH) per species equals the per-species total
- Arabidopsis higher total tncRNA abundance reflects its larger sample size while unique tncRNA relative abundance stays comparable to the other five plants

## Limitations

- RPM reflects mapping depth per progenitor tRNA but does not distinguish isodecoders (tRNAs with the same anticodon), whose role in tncRNA biogenesis the authors note requires further exploration
- Arabidopsis totals are inflated by its greater sample size among the 2,448 sRNA-seq samples, so raw totals are not directly comparable across species
- Conventional RNA-seq modification biases can affect tncRNA counts; the authors suggest switching to tRNA-seq to overcome these biases

## Evidence

- [results] RPM formula: [evidence span withheld — anchor status: not_found]
- [other] pairwise2 alignment: [evidence span withheld — anchor status: not_found]
