---
name: nucleotide-sequence-dataset-partitioning
description: Use when a nucleotide sequence collection (FASTA bins, e.g. 64-bin or
  1,024-bin partitionings) must be split into bins and the effectiveness of a minimizer-based
  pre-filter index (representative k-mer set) must be measured.
when_to_use_negative:
- The collection has highly uneven bin sizes — the paper states the main limiting
  factor is 'its susceptibility to uneven bin sizes as well as a high number of bins'
- The analysis requires exact full-text search or edit-distance-sensitive indexing
  (word/minimizer-based indices like Raptor's binning directory are pre-filters, not
  FM-index replacements)
- The input is already a built Raptor index — note indices built with an older version
  of raptor cannot be used with version 2.0.0, so mixing index versions is not supported
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Raptor
  role: Computes winnowing-minimizer representative k-mer sets per bin and builds
    the interleaved Bloom filter binning directory used to derive the effective text
    ratio
  repo: https://github.com/seqan/raptor
- name: TaxSBP
  role: Hierarchically structured bin packing of sequences into bins using the NCBI
    Taxonomy database (the partitioning step upstream of indexing)
  repo: https://github.com/pirovc/taxsbp
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2021.102782
    title: 'Raptor: A fast and space-efficient pre-filter for querying very large
      collections of nucleotide sequences'
  source_content_hash: sha256:da38be0c26373c0f5bb1c76cbf1a7fbabf0d854657cb0f7f9c68a5aa48f47855
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seqan/raptor
  tools:
  - Raptor
  - TaxSBP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# nucleotide-sequence-dataset-partitioning

## Summary

Partitions a large nucleotide sequence collection into bins (e.g., 64 or 1,024 bins of NCBI RefSeq or an artificial dataset) and quantifies the effective text ratio r(k) of Raptor's winnowing-minimizer representative k-mers across k-mer sizes k=12..20, as reported in Table 8 of the Raptor paper.

## When to use

Use when a nucleotide sequence collection (FASTA bins, e.g. 64-bin or 1,024-bin partitionings) must be split into bins and the effectiveness of a minimizer-based pre-filter index (representative k-mer set) must be measured.

## When NOT to use

- The collection has highly uneven bin sizes — the paper states the main limiting factor is 'its susceptibility to uneven bin sizes as well as a high number of bins'
- The analysis requires exact full-text search or edit-distance-sensitive indexing (word/minimizer-based indices like Raptor's binning directory are pre-filters, not FM-index replacements)
- The input is already a built Raptor index — note indices built with an older version of raptor cannot be used with version 2.0.0, so mixing index versions is not supported

## Inputs

- Artificial dataset partitioned into 64 bins (nucleotide FASTA) from github.com/seqan/raptor
- Artificial dataset partitioned into 1,024 bins (nucleotide FASTA) from github.com/seqan/raptor
- Original (unpartitioned) text length for normalization of n(k)
- k-mer size range k=12..20

## Outputs

- Effective text ratio table (effective_text_ratio.tsv) with r(k) for k=12..20 for 64 and 1,024 bins
- Counts of distinct winnowing-minimizer representative k-mers per partitioning (effective text length n(k))

## How to apply

Download the artificial dataset (64-bin and 1,024-bin partitions) from the Raptor repository (github.com/seqan/raptor; its `util` subdirectory contains applications and scripts related to papers). Install Raptor via bioconda ('conda install -c bioconda -c conda-forge raptor') and confirm command availability with 'raptor --help'; each command has a help page, e.g. 'raptor build --help'. For each k in 12..20, run Raptor's minimizer computation per bin to obtain the set of winnowing-minimizer representative k-mers, count the distinct representative k-mers per partitioning to derive the effective text length n(k), and normalize n(k) by the original text length to obtain the effective text ratio r(k) for both binnings. Tabulate r(k) for k=12..20 at 64 and 1,024 bins and validate against Table 8: r drops from 62.84 (64 bins) and 225.98 (1,024 bins) at k=12 to 1.01 at k=19 and 1.00 at k=20 for both binnings. Rationale: Raptor uses winnowing minimizers to define a set of representative k-mers stored in an interleaved Bloom filter (IBF), so r(k) quantifies how much the pre-filter compresses the text before losing information.

## Related tools

- **Raptor** (Computes winnowing-minimizer representative k-mer sets per bin and builds the interleaved Bloom filter binning directory used to derive the effective text ratio) — https://github.com/seqan/raptor
- **TaxSBP** (Hierarchically structured bin packing of sequences into bins using the NCBI Taxonomy database (the partitioning step upstream of indexing)) — https://github.com/pirovc/taxsbp

## Examples

```
conda install -c bioconda -c conda-forge raptor && raptor --help
```

## Evaluation signals

- r(12) for the 64-bin partitioning reproduces to ≈62.84 and for the 1,024-bin partitioning to ≈225.98 (Table 8)
- r(19) ≈ 1.01 and r(20) = 1.00 for BOTH the 64-bin and 1,024-bin partitionings (Table 8)
- r(k) decreases monotonically as k increases from 12 to 20 for each binning
- Distinct representative k-mer counts per partitioning yield n(k) that, normalized by the original text length, sum consistently across bins
- Command availability check: 'raptor --help' lists the expected subcommands (e.g., build)

## Limitations

- The method is susceptible to uneven bin sizes and to a high number of bins; an adaptive IBF for uneven bin sizes was still under development
- Raptor indices built with an older version cannot be used with version 2.0.0
- P(Cn) has no tractable formulation and is estimated experimentally by sampling; probabilistic thresholding is approximate

## Evidence

- [abstract] winnowing minimizers define representative k-mers: "Raptor uses winnowing minimizers to define a set of representative k-mers, an extension of the interleaved Bloom filters (IBFs)"
- [discussion] limiting factor: uneven bins and many bins: "The main limiting factor of our method is its susceptibility to uneven bin sizes as well as a high number of bins."
- [discussion] artificial dataset location: "Artificial data set (64 and 1024 bins)This paperhttps://github.com/seqan/raptor"
- [readme] install and help commands: "conda install -c bioconda -c conda-forge raptor"
- [readme] paper-related scripts in repo: "The subdirectory `util` contains applications and scripts related to papers."
