---
name: tncrna-count-aggregation-per-species
description: Use when you have per-sample tncRNA classification tables from a tncRNA
  Toolkit run and need species-level summaries of total and unique tncRNA counts per
  subtype for cross-species comparison.
when_to_use_negative:
- Input is raw FASTQ or unaligned BAM — classification must already be done by the
  tncRNA Toolkit before aggregation
- Comparing raw total counts across species with very different sample sizes (e.g.
  Arabidopsis vs maize) to claim biological differences in abundance
- Aggregating reads classified by a different pipeline with incompatible subtype definitions
  (no standardized tncRNA nomenclature exists, so class labels may not match tRF-5/tRF-3/tRF-1/tRH
  conventions)
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3170
tools:
- name: tncRNA Toolkit
  role: Produces the per-sample tncRNA classification outputs (tRF-5, tRF-3, tRF-1,
    tRNA halves) that are aggregated by this skill
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: pandas
  role: Used to load, group-by species and tncRNA class, and sum unique-read/locus
    counts from the classification tables
- name: python3
  role: Runtime for the aggregation script and for the upstream tncRNAs.py classification
    step
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:46444abfd25de9ad337242ab7f9f82ecc39d037b32863552dbd5ba45b14e3026
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
  - pandas
  - python3
  - tncRNA Toolkit
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# tncRNA-count-aggregation-per-species

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aggregates per-sample tncRNA classification outputs from the tncRNA Toolkit (tncRNAs.py) into per-species and per-subtype (tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH) totals of unique reads and distinct loci, reconciling with the 43,153 tncRNAs reported across 2,448 sRNA-seq samples from six angiosperms.

## When to use

Use when you have per-sample tncRNA classification tables from a tncRNA Toolkit run and need species-level summaries of total and unique tncRNA counts per subtype for cross-species comparison.

## When NOT to use

- Input is raw FASTQ or unaligned BAM — classification must already be done by the tncRNA Toolkit before aggregation
- Comparing raw total counts across species with very different sample sizes (e.g. Arabidopsis vs maize) to claim biological differences in abundance
- Aggregating reads classified by a different pipeline with incompatible subtype definitions (no standardized tncRNA nomenclature exists, so class labels may not match tRF-5/tRF-3/tRF-1/tRH conventions)

## Inputs

- Per-sample tncRNA classification output tables from tncRNAs.py (-o output directory), with read counts and locus information per classified read
- Species-to-sample mapping for the six angiosperms (A. thaliana, M. truncatula, S. lycopersicum, C. arietinum, O. sativa, Z. mays)
- Total mapped reads per sample (for RPM normalization)

## Outputs

- species_tncRNA_summary.csv: per-species total and unique tncRNA counts, split by tncRNA subtype
- Per-class unique-read and distinct-locus counts per species
- Optionally RPM-normalized expression aggregates per tncRNA

## How to apply

Load the per-sample classification output tables from the tncRNAs.py output directory with pandas, keeping the read count and locus information recorded for each classified read. Group reads by species and by tncRNA class (tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH), summing unique-read counts and counting distinct loci per species. Where expression-level aggregation is needed, compute RPM = reads mapped to the progenitor tRNA / (total mapped reads / 10^6), following the paper's PMF formula. Write the per-species totals to a single summary table (e.g. species_tncRNA_summary.csv) and validate that row counts and class sums reconcile with published reference values (e.g. A. thaliana 1,676; O. sativa 243; Z. mays 221). Note that Arabidopsis shows higher total abundance simply due to greater sample size, so compare relative unique-sequence abundance across species rather than raw totals.

## Related tools

- **tncRNA Toolkit** (Produces the per-sample tncRNA classification outputs (tRF-5, tRF-3, tRF-1, tRNA halves) that are aggregated by this skill) — https://github.com/skbinfo/tncRNA-Toolkit
- **pandas** (Used to load, group-by species and tncRNA class, and sum unique-read/locus counts from the classification tables)
- **python3** (Runtime for the aggregation script and for the upstream tncRNAs.py classification step)

## Examples

```
python3 tncRNAs.py -s Arabidopsis_thaliana -i sample.fq -o outdir/ && python3 aggregate_species_counts.py outdir/ --summary species_tncRNA_summary.csv
```

## Evaluation signals

- Aggregated per-species totals reconcile with article reference values (A. thaliana 1,676 tncRNAs; O. sativa 243; Z. mays 221) and sum consistently across the six angiosperms
- Sum of per-subtype counts equals the per-species total for each species (class-wise sums check)
- Locus counts are counted as distinct loci, not read counts — verify no double-counting of multi-mapped loci (reads aligned >50 times were already discarded with bowtie '-m 50')
- Only reads classified as mapping exclusively to the artificial chromosomes (tRNA ± 50 nt) are included in totals
- Arabidopsis total counts exceed other species but unique-sequence relative abundance is comparable — a sanity signal that sample-size effects are understood

## Limitations

- Total tncRNA abundance is confounded by sample size: Arabidopsis showed higher abundance due to greater sample size, not biology
- A single standardized nomenclature for tncRNA subtypes does not exist, so aggregated class labels may not map cleanly to other pipelines (tDRmapper, MINTmap, SPORTS1.0)
- Multi-mapped reads aligned more than 50 times were discarded upstream, so repeat-derived tncRNAs are underrepresented in the totals

## Evidence

- [other] Total tncRNAs in Arabidopsis showed higher abundance due to its greater sample size: [evidence span withheld — anchor status: not_found]
