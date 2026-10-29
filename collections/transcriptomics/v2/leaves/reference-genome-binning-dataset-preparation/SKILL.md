---
name: reference-genome-binning-dataset-preparation
description: Use when benchmarking a sequence-search pre-filter or distributed read
  mapper against reference collections that must be partitioned into bins, or when
  reproducing the Raptor Table 7 DREAM-Yara experiments on clustered NCBI RefSeq data.
when_to_use_negative:
- The task requires genome assembly or single-genome annotation rather than partitioning
  many reference genomes into search bins.
- Benchmarks must use real, uneven clinical/metagenomic samples — the paper notes
  the main limiting factor of the method is its susceptibility to uneven bin sizes
  and a high number of bins.
- Only a small collection of genomes is involved, where a plain FM-index mapper without
  binning is sufficient.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: TaxSBP
  role: Hierarchically structured bin packing over the NCBI Taxonomy used to cluster
    reference sequences into bins for the benchmark datasets
  repo: https://github.com/pirovc/taxsbp
- name: Raptor
  role: 'Consumer of the prepared bin sets: builds IBF/minimizer indices for the 64-
    and 1,024-bin clustered RefSeq benchmarks'
  repo: https://github.com/seqan/raptor
- name: DREAM-Yara
  role: Distributed read mapper re-implemented with Raptor's IBF, mapped against the
    prepared clustered bin sets in the case study
  repo: https://github.com/seqan/dream_yara
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2021.102782
    title: 'Raptor: A fast and space-efficient pre-filter for querying very large
      collections of nucleotide sequences'
  source_content_hash: sha256:5081d0b5626dc1177d177eace4472ab27c2847dbf2b239f4aaafc4bc39ddde9e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pirovc/taxsbp
  tools:
  - DREAM-Yara
  - Raptor
  - TaxSBP
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# reference-genome-binning-dataset-preparation

## Summary

Prepare taxonomically clustered reference-genome bin sets (e.g., Clustered NCBI RefSeq with 64 or 1,024 bins) and matching artificial query read sets, using TaxSBP hierarchical bin packing over the NCBI Taxonomy, so that pre-filter/mapping benchmarks like the Raptor DREAM-Yara case study can be reproduced.

## When to use

Use when benchmarking a sequence-search pre-filter or distributed read mapper against reference collections that must be partitioned into bins, or when reproducing the Raptor Table 7 DREAM-Yara experiments on clustered NCBI RefSeq data.

## When NOT to use

- The task requires genome assembly or single-genome annotation rather than partitioning many reference genomes into search bins.
- Benchmarks must use real, uneven clinical/metagenomic samples — the paper notes the main limiting factor of the method is its susceptibility to uneven bin sizes and a high number of bins.
- Only a small collection of genomes is involved, where a plain FM-index mapper without binning is sufficient.

## Inputs

- Tab-separated sequence info file: sequence id, sequence length, taxonomic id [, specialization]
- nodes.dmp (and optionally merged.dmp) from the NCBI Taxonomy taxdump (ftp://ftp.ncbi.nih.gov/pub/taxonomy/taxdump.tar.gz)
- NCBI RefSeq Archaea and Bacteria reference genomes (doi:10.5281/zenodo.4647988)
- Pre-clustered bin sets: Clustered NCBI RefSeq 64 bins (doi:10.5281/zenodo.4650188) and 1,024 bins (doi:10.5281/zenodo.4651078)
- Artificial query read sets for the 64- and 1,024-bin experiments

## Outputs

- Tab-separated bin assignment file: sequence id, seq.start, seq.end, sequence length, taxonomic id, bin id [, specialization]
- Bin-partitioned reference collections (64-bin and 1,024-bin clustered RefSeq sets) ready for IBF/index construction
- Paired artificial query read sets used to probe each bin set

## How to apply

Cluster the reference sequences into bins with TaxSBP, a hierarchically structured bin packing algorithm adapted to the NCBI Taxonomy database: provide a tab-separated input of sequence id, sequence length, and taxonomic id, plus nodes.dmp (and optionally merged.dmp) from the NCBI Taxonomy dump; control bin count via a maximum bin length (-l) or approximate number of bins (-b), and optionally fragment sequences (-f) with overlap (-a) or bin exclusively by a rank (-e). Use the specialization field to cluster beyond taxonomy (e.g., by strain name or assembly accession). For the Raptor DREAM-Yara case study, download the pre-built clustered bin sets instead: 64 bins from Zenodo doi:10.5281/zenodo.4650188 and 1,024 bins from Zenodo doi:10.5281/zenodo.4651078, together with the corresponding artificial query read sets. The resulting bin-partitioned references are then indexed (e.g., 16 GiB IBF for the original 19-mer DREAM-Yara, or minimizer-based (23,19)/(31,19) IBFs) to measure build/mapping time, RAM, and unmapped-read counts.

## Related tools

- **TaxSBP** (Hierarchically structured bin packing over the NCBI Taxonomy used to cluster reference sequences into bins for the benchmark datasets) — https://github.com/pirovc/taxsbp
- **Raptor** (Consumer of the prepared bin sets: builds IBF/minimizer indices for the 64- and 1,024-bin clustered RefSeq benchmarks) — https://github.com/seqan/raptor
- **DREAM-Yara** (Distributed read mapper re-implemented with Raptor's IBF, mapped against the prepared clustered bin sets in the case study) — https://github.com/seqan/dream_yara

## Examples

```
conda install -c bioconda -c conda-forge taxsbp; taxsbp.py -i seqinfo.tsv -n nodes.dmp -l 10000000 -f 999500 -a 500
```

## Evaluation signals

- Bin assignment output is a tab-separated file with the exact fields '#id, st, end, len, tax, bin [, spec]' as specified in the TaxSBP README.
- Bin count matches the target configuration (64 bins vs 1,024 bins) when using the Zenodo-published clustered RefSeq sets (doi:10.5281/zenodo.4650188 / 4651078).
- Total assigned sequence length is conserved: with fragmentation (-f) and overlap (-a), fragment intervals (e.g., length 55 from 50 bp fragments with 5 bp overlap) cover the original sequences.
- Exclusive binning by a taxonomic rank (-e) produces one cluster per sub-tree of that rank, with the exclusive rank's taxid printed instead of the original.
- Downstream benchmark reproduces Table 7 expectations: mapping memory drops to 17.5 GiB for (23,19)-minimizers and 15.2 GiB for (31,19)-minimizers at 64 bins, with unmapped reads increasing only by 2 and 14.

## Limitations

- TaxSBP binning depends on up-to-date NCBI Taxonomy nodes.dmp/merged.dmp; sequences with missing taxid fall back to a placeholder specialization.
- Uneven bin sizes and a very high number of bins are the main limiting factor for the downstream Raptor/IBF search.
- Indices built with an older Raptor version cannot be used with version 2.0.0, so re-binned datasets must be re-indexed after upgrades.
- Pre-built DREAM-Yara indices are hosted on an external FTP server (https://ftp.imp.fu-berlin.de/pub/raptor/), not a versioned archive.

## Evidence

- [discussion] TaxSBP taxonomy clustering: "TaxSBPVitor Pirohttps://github.com/pirovc/taxsbp"
- [discussion] Clustered RefSeq 64-bin dataset: "Clustered NCBI RefSeq (64 bins)This paperhttps://doi.org/10.5281/zenodo.4650188"
- [discussion] Clustered RefSeq 1024-bin dataset: "Clustered NCBI RefSeq (1024 bins)This paperhttps://doi.org/10.5281/zenodo.4651078"
- [readme] TaxSBP installation via bioconda: "conda install -c bioconda -c conda-forge taxsbp"
- [readme] Specialization field for clustering beyond taxonomy: [evidence span withheld — anchor status: external_doc]
- [readme] TaxSBP input format: [evidence span withheld — anchor status: external_doc]
