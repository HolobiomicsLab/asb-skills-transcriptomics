---
name: sequence-read-query-execution
description: Use when a set of sequencing reads must be searched against a pre-built
  Raptor IBF/MIBF index over 64 or 1,024 bins, and run time, memory, and bin-assignment
  accuracy must be measured per index size.
when_to_use_negative:
- The target index was built with an older Raptor version, since indices built with
  an older version cannot be used with version 2.0.0
- The reference sequences have highly uneven bin sizes or very many bins, which Raptor
  explicitly identifies as its main limiting factor
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Raptor
  role: Constructs the IBF/MIBF binning-directory index ('raptor build') and executes
    the read queries against it ('raptor search'), recording the bin assignments used
    for FP counting.
  repo: https://github.com/seqan/raptor
- name: TaxSBP
  role: Related clustering tool used to bin sequences (e.g. clustered NCBI RefSeq
    datasets) prior to indexing; not required for the artificial-dataset search itself.
  repo: https://github.com/pirovc/taxsbp
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2021.102782
    title: 'Raptor: A fast and space-efficient pre-filter for querying very large
      collections of nucleotide sequences'
  source_content_hash: sha256:6649dc6438fd451f055dd3eeffb21aca41211a6dd9f5980d499164c1a1cc9916
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seqan/raptor
  tools:
  - Raptor
  - TaxSBP
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# sequence-read-query-execution

## Summary

Executes approximate sequence search of large query sets (e.g. 2^20 NGS reads) against Raptor binning-directory indices built from interleaved Bloom filters (IBF/MIBF), recording search time, peak RAM, and false-positive bin assignments across index configurations.

## When to use

Use when a set of sequencing reads must be searched against a pre-built Raptor IBF/MIBF index over 64 or 1,024 bins, and run time, memory, and bin-assignment accuracy must be measured per index size.

## When NOT to use

- The target index was built with an older Raptor version, since indices built with an older version cannot be used with version 2.0.0
- The reference sequences have highly uneven bin sizes or very many bins, which Raptor explicitly identifies as its main limiting factor

## Inputs

- Pre-built Raptor binning directory (IBF or MIBF index, 1–8 GiB, compressed or uncompressed)
- Query FASTA/FASTQ of simulated next-generation sequencing reads (2^20 reads)
- Artificial dataset partitioned into 64 or 1,024 bins

## Outputs

- Per-configuration search results (bin assignments per read)
- Measured construction time, search time, and peak RAM per index configuration
- Count of false-positive bin assignments per configuration
- Performance table (e.g. table1.csv) with one row per index configuration

## How to apply

Install Raptor via 'conda install -c bioconda -c conda-forge raptor', or compile from source for host-system optimization as the paper notes. Build (19,19)-IBF and (23,19)-MIBF binning directories with 'raptor build' over the artificial 64-bin and 1,024-bin dataset for IBF sizes of 1, 2, 4, and 8 GiB, in both uncompressed and compressed form. Then run 'raptor search' for each index to query the 2^20 simulated reads, capturing construction time, search time, peak RAM, and the number of false-positive bin assignments. The 1 GiB (19,19) configuration is expected to produce inflated false positives (e.g. 604,533 FPs at b=64) while doubling the IBF size to 2 GiB drops them to near zero (189 FPs), so results should be compiled per configuration to expose this size/accuracy trade-off before writing the performance table (e.g. table1.csv).

## Related tools

- **Raptor** (Constructs the IBF/MIBF binning-directory index ('raptor build') and executes the read queries against it ('raptor search'), recording the bin assignments used for FP counting.) — https://github.com/seqan/raptor
- **TaxSBP** (Related clustering tool used to bin sequences (e.g. clustered NCBI RefSeq datasets) prior to indexing; not required for the artificial-dataset search itself.) — https://github.com/pirovc/taxsbp

## Examples

```
raptor search --help && raptor build --help
```

## Evaluation signals

- table1.csv exists with one row per index configuration (2 minimizer schemes x 4 IBF sizes x 2 bin counts x 2 compression states)
- False-positive bin assignments for the 1 GiB (19,19) index are large (order 10^5 for b=64, e.g. 604,533) while 2 GiB and larger configs drop to near zero (e.g. 189 FPs)
- Search time and peak RAM values are recorded for every configuration and vary monotonically/sensibly with IBF size and compression state
- Bin-assignment output is consistent with Raptor's reported near-zero false-positive performance on simulated reads for adequately sized indices
- Raptor and binning-directory commands exit without error and their help pages ('raptor build --help', 'raptor search --help') match the flags used

## Limitations

- Susceptibility to uneven bin sizes and to a high number of bins is the main limiting factor of the method
- Processing several ten thousand bins leads to a moderate slowdown
- Compressed-bitvector construction requires both uncompressed and compressed bitvectors in main memory, and SDSL does not support decompressing the compressed bitvector
- Indices built with an older version of Raptor cannot be used with version 2.0.0
- Allocating too small an IBF (1 GiB with (19,19) minimizers) produces a high number of false-positive bin assignments

## Evidence

- [abstract] winnowing minimizers define representative k-mers with IBF membership: "Raptor uses winnowing minimizers to define a set of representative k-mers, an extension of the interleaved Bloom filters (IBFs) as a set membership data structure and probabilistic thresholding for"
- [abstract] performance tested on simulated and real datasets: "We test and show the performance and limitations of the new features using simulated and real datasets."
- [discussion] main limitation: uneven bin sizes and many bins: "The main limiting factor of our method is its susceptibility to uneven bin sizes as well as a high number of bins."
- [readme] install Raptor via bioconda: "conda install -c bioconda -c conda-forge raptor"
- [readme] compile from source for host optimization: "There may be performance benefits when compiling from source, as Raptor can be optimized for the host system."
- [readme] each raptor command has a help page: "Each command will have a respective help page that can be shown via, e.g., `raptor build --help`."
