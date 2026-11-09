---
name: transcript-query-set-sampling
description: Use when benchmarking a pre-filter such as Raptor on RNA-Seq collections
  and you need reproducible transcript query sets of increasing size plus ground-truth
  membership calls to compute false-positive rates.
when_to_use_negative:
- The query objects are whole genomes or contigs rather than transcripts/reads — Raptor
  is designed for approximately searching many queries such as NGS reads or transcripts,
  and uneven bin sizes or a high number of bins are its main limiting factors.
- An exact, edit-distance-tolerant full-text search is required rather than an approximate
  pre-filter step before more expensive downstream search.
- The collection is very small (a handful of bins), where a full-text index such as
  Mantis alone is cheap enough and the pre-filter comparison is unnecessary.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3308
tools:
- name: Raptor
  role: Builds the minimizer-based binning directory and performs approximate transcript
    membership searches ('raptor build' / 'raptor search'); queried at each query-set
    scale to measure speed, RAM, and FPR.
  repo: https://github.com/seqan/raptor
- name: Mantis
  role: Exact set-membersity index over the same RNA-Seq experiments; its search results
    serve as ground truth for computing Raptor's false-positive rate and as the speed/memory
    baseline.
  repo: https://github.com/splatlab/mantis
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2021.102782
    title: 'Raptor: A fast and space-efficient pre-filter for querying very large
      collections of nucleotide sequences'
  source_content_hash: sha256:169293c9fb7048bb63a0dbc0b248ba2a5a89fdf4668100519e06ae8af5d02467
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seqan/raptor
  tools:
  - Mantis
  - Raptor
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# transcript-query-set-sampling

## Summary

Samples transcript query sets of graded sizes (e.g., 100, 1,000, and 10,000 human transcripts) to benchmark approximate sequence-search pre-filters like Raptor against Mantis on real human RNA-Seq collections, using the exact-search results as ground truth for specificity estimation.

## When to use

Use when benchmarking a pre-filter such as Raptor on RNA-Seq collections and you need reproducible transcript query sets of increasing size plus ground-truth membership calls to compute false-positive rates.

## When NOT to use

- The query objects are whole genomes or contigs rather than transcripts/reads — Raptor is designed for approximately searching many queries such as NGS reads or transcripts, and uneven bin sizes or a high number of bins are its main limiting factors.
- An exact, edit-distance-tolerant full-text search is required rather than an approximate pre-filter step before more expensive downstream search.
- The collection is very small (a handful of bins), where a full-text index such as Mantis alone is cheap enough and the pre-filter comparison is unnecessary.

## Inputs

- Collection of human RNA-Seq experiments (e.g., NIH brain, breast, and blood tissue, 2652 experiments, Zenodo DOI 10.5281/zenodo.1186393)
- Raptor binning directory built with (20,20)- and/or (40,20)-minimizers at k=20
- Mantis index over the same experiments (ground-truth membership source)
- FASTA/FASTQ transcript sequences for sampling query sets of 100, 1,000, and 10,000 human transcripts

## Outputs

- Construction resource table: wall-clock time, peak RAM, and on-disk index size per index (Raptor vs Mantis)
- Query performance table: search time and RAM per query-set size (100/1,000/10,000 transcripts)
- False-positive rate of each Raptor index at every query scale, computed against Mantis ground-truth results

## How to apply

Download the human RNA-Seq experiment collection (e.g., the 2,652-experiment NIH brain, breast, and blood tissue set at DOI 10.5281/zenodo.1186393) and build a Raptor binning directory with 'raptor build' using a chosen minimizer scheme, e.g., (20,20)- or (40,20)-minimizers at k=20, alongside a Mantis index over the same experiments. Randomly sample three transcript query sets of 100, 1,000, and 10,000 human transcripts so that search cost and specificity can be evaluated as a function of query-batch size. Search each query set against both the Raptor indices ('raptor search') and the Mantis index, recording per-query-set search time and peak RAM. Treat the Mantis results as ground truth and compute the false-positive rate of each Raptor index at every query scale; the paper reports a low FP rate of about 0.017 for (40,20)-minimizers, with Raptor 12-58 times faster than Mantis. Assemble construction resources (time/RAM/on-disk size) and query metrics (time, RAM, FPR) into two tables and check that recorded values fall within the ranges reported in the paper's benchmark tables for the same minimizer settings.

## Related tools

- **Raptor** (Builds the minimizer-based binning directory and performs approximate transcript membership searches ('raptor build' / 'raptor search'); queried at each query-set scale to measure speed, RAM, and FPR.) — https://github.com/seqan/raptor
- **Mantis** (Exact set-membersity index over the same RNA-Seq experiments; its search results serve as ground truth for computing Raptor's false-positive rate and as the speed/memory baseline.) — https://github.com/splatlab/mantis

## Examples

```
conda install -c bioconda -c conda-forge raptor
```

## Evaluation signals

- False-positive rate of each Raptor index at every query scale (100/1,000/10,000 transcripts), computed against Mantis ground truth, stays low (paper reports ~0.017 for (40,20)-minimizers).
- Search time for Raptor with (40,20)-minimizers falls in the 12-58x speedup range over Mantis reported on the 1,742-experiment RNA-Seq dataset.
- Construction table entries (wall-clock time, peak RAM, on-disk index size) for Raptor and Mantis are internally consistent and comparable to the paper's benchmark tables for the same minimizer settings.
- Query-set sizes are exactly 100, 1,000, and 10,000 transcripts and are sampled randomly from the human transcript set, so per-size metrics are comparable across tools.
- Both Raptor indices and the Mantis index are built over the identical experiment collection so membership comparisons are valid.

## Limitations

- Raptor is susceptible to uneven bin sizes as well as a high number of bins, which is its main limiting factor.
- Processing several ten thousand bins leads to a moderate slowdown.
- Indices built with an older version of raptor cannot be used with version 2.0.0, so index and tool versions must match when re-running benchmark searches.

## Evidence

- [abstract] Raptor uses winnowing minimizers, an extension of interleaved Bloom filters (IBFs), and probabilistic thresholding for minimizers: "Raptor uses winnowing minimizers to define a set of representative k-mers, an extension of the interleaved Bloom filters (IBFs) as a set membership data structure and probabilistic thresholding for"
- [intro] Raptor compared to Mantis and COBS on speed and memory: "compare it with the state-of-the-art methods Mantis and COBS. Our comparison shows that Raptor is up to times faster than the competitors"
- [discussion] Real RNA-Seq benchmark collection (Zenodo DOI 10.5281/zenodo.1186393): "NIH brain, breast, and blood tissue (2652 experiments)Pandey et al., 2018https://doi.org/10.5281/zenodo.1186393"
- [discussion] Motivating use case: transcript membership queries across human RNA-Seq files: "query all existing human RNA-Seq files in the SRA for the presence of transcripts"
- [readme] Raptor commands have per-command help pages for building and searching: "`raptor --help` will show available commands. Each command will have a respective help page that can be shown via, e.g.,
`raptor build --help`."
