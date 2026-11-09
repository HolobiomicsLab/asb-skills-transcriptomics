---
name: rnaseq-index-benchmarking
description: Use when evaluating whether a pre-filter such as Raptor can replace or
  complement an exact index like Mantis for membership queries of transcripts/reads
  against thousands of SRA RNA-Seq experiments, with resource and FPR metrics.
when_to_use_negative:
- The input collection is a small set of genomes or a clustered RefSeq dataset with
  few bins — the paper's limiting factors are 'uneven bin sizes as well as a high
  number of bins', so tiny collections do not exercise the pre-filter's advantages
- Edit-distance-tolerant full-text alignment results are required rather than approximate
  set-membership pre-filtering — Raptor is a pre-filter, not a mapper like DREAM-Yara
- The index was built with an older Raptor release and queried with version 2.0.0
  — indices are not cross-version compatible
edam_operation: http://edamontology.org/operation_0338
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: Raptor
  role: Builds (20,20)- and (40,20)-minimizer binning directories at k=20 with 'raptor
    build' and searches transcript query sets with 'raptor search'; construction and
    query resources and FPR are the benchmarked metrics
  repo: https://github.com/seqan/raptor
- name: Mantis
  role: Exact-lookup baseline index over the same RNA-Seq experiments; its search
    results serve as ground truth for computing Raptor's false-positive rate and its
    time/RAM/space are the comparison target
  repo: https://github.com/splatlab/mantis
- name: TaxSBP
  role: Optional binning/packing helper for hierarchically clustering sequences into
    bins before index construction
  repo: https://github.com/pirovc/taxsbp
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2021.102782
    title: 'Raptor: A fast and space-efficient pre-filter for querying very large
      collections of nucleotide sequences'
  source_content_hash: sha256:e619eed54094d3b413ca4a8a1ba26313e85775bf371e659bb85b85c542b5b118
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seqan/raptor
  tools:
  - Mantis
  - Raptor
  - TaxSBP
  edam_operation: http://edamontology.org/operation_0338
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# rnaseq-index-benchmarking

## Summary

Benchmarks approximate sequence-search indices (Raptor binning directories vs Mantis) on a large human RNA-Seq experiment collection, comparing construction time/RAM/space and query speed, memory, and false-positive rate for transcript membership searches.

## When to use

Use when evaluating whether a pre-filter such as Raptor can replace or complement an exact index like Mantis for membership queries of transcripts/reads against thousands of SRA RNA-Seq experiments, with resource and FPR metrics.

## When NOT to use

- The input collection is a small set of genomes or a clustered RefSeq dataset with few bins — the paper's limiting factors are 'uneven bin sizes as well as a high number of bins', so tiny collections do not exercise the pre-filter's advantages
- Edit-distance-tolerant full-text alignment results are required rather than approximate set-membership pre-filtering — Raptor is a pre-filter, not a mapper like DREAM-Yara
- The index was built with an older Raptor release and queried with version 2.0.0 — indices are not cross-version compatible

## Inputs

- Human RNA-Seq experiment collection (NIH brain, breast, and blood tissue experiments) from Zenodo DOI 10.5281/zenodo.1186393
- Randomly sampled query sets of 100, 1,000, and 10,000 human transcripts (FASTA/ nucleotide sequences)
- Raptor and Mantis command-line tools installed (e.g. via bioconda)

## Outputs

- Construction-resources table: wall-clock build time, peak RAM, and on-disk index size for each Raptor binning directory and the Mantis index
- Query-performance table: per-query-set search time, search RAM, and false-positive rate of each Raptor index relative to Mantis ground truth

## How to apply

Download the human RNA-Seq experiment collection from Zenodo (DOI 10.5281/zenodo.1186393) and build two Raptor binning directories with 'raptor build' at k=20 using (20,20)- and (40,20)-minimizer schemes, plus a Mantis index over the same experiments, recording wall-clock time, peak RAM, and on-disk index size for each construction. Randomly sample three query sets of 100, 1,000, and 10,000 human transcripts and search each against both Raptor indices with 'raptor search' and against Mantis with its search command, recording per-query-set search time and RAM. Treat the Mantis results as ground truth and compute the false-positive rate of each Raptor index at every query scale; on the real RNA-Seq dataset Raptor with (40,20)-minimizers is 12-58 times faster than Mantis, uses only a fraction of its main memory, and stays specific with an FP rate of about 0.017. Assemble the recorded values into a construction-resources table and a query-performance table and check they are internally consistent with the ranges reported in the paper's benchmark tables for the same minimizer settings.

## Related tools

- **Raptor** (Builds (20,20)- and (40,20)-minimizer binning directories at k=20 with 'raptor build' and searches transcript query sets with 'raptor search'; construction and query resources and FPR are the benchmarked metrics) — https://github.com/seqan/raptor
- **Mantis** (Exact-lookup baseline index over the same RNA-Seq experiments; its search results serve as ground truth for computing Raptor's false-positive rate and its time/RAM/space are the comparison target) — https://github.com/splatlab/mantis
- **TaxSBP** (Optional binning/packing helper for hierarchically clustering sequences into bins before index construction) — https://github.com/pirovc/taxsbp

## Examples

```
raptor build --input rna-seq-experiments.list --output raptor_40_20 --kmer 20 --window 40 --parts 4 && raptor search --index raptor_40_20 --queries transcripts_1000.fasta --output results_1000.out
```

## Evaluation signals

- Construction table reports wall-clock time, peak RAM, and on-disk index size for both Raptor minimizer settings and Mantis, all built over the same 1,742-experiment collection
- Query table covers all three query scales (100, 1,000, 10,000 transcripts) with search time, RAM, and FPR recorded for each index
- FPR is computed against Mantis results as ground truth and falls near the paper's reported value of about 0.017 for (40,20)-minimizers
- Recorded speedups fall within the paper's reported 12-58x range for (40,20)-minimizers over Mantis
- Index sizes and search RAM are consistent with Raptor using 'only a fraction of the memory' of Mantis at equal query scales

## Limitations

- Raptor's main limiting factor is its susceptibility to uneven bin sizes as well as a high number of bins; processing several ten thousand bins leads to a moderate slowdown
- Compressed bitvector construction requires both the uncompressed and compressed bitvectors in main memory at some point, since SDSL does not support decompression of the compressed bitvector
- Indices built with an older version of Raptor cannot be used with version 2.0.0

## Evidence

- [results] Raptor with (40,20)-minimizers is 12-58 times faster than Mantis, uses only a fraction of its main memory, and remains specific with a low FP rate of about 0.017: [evidence span withheld — anchor status: not_found]
- [discussion] main limiting factor of our method is its susceptibility to uneven bin sizes as well as a high number of bins: "The main limiting factor of our method is its susceptibility to uneven bin sizes as well as a high number of bins."
- [discussion] NIH brain, breast, and blood tissue (2652 experiments)Pandey et al., 2018https://doi.org/10.5281/zenodo.1186393: "NIH brain, breast, and blood tissue (2652 experiments)Pandey et al., 2018https://doi.org/10.5281/zenodo.1186393"
- [readme] Each command will have a respective help page that can be shown via, e.g., `raptor build --help`.: "`raptor --help` will show available commands. Each command will have a respective help page that can be shown via, e.g.,
`raptor build --help`."
- [intro] only use a fraction of the memory and can further trade run time for main memory consumption: "In addition, we only use a fraction of the memory and can further trade run time for main memory consumption."
