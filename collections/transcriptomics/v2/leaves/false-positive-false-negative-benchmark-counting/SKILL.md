---
name: false-positive-false-negative-benchmark-counting
description: Use when you need to measure how an approximate sequence search index
  (e.g., a minimizer interleaved Bloom filter with probabilistic thresholding) trades
  off false-positive and false-negative bin assignments across different index sizes
  or bin counts.
when_to_use_negative:
- You need exact-match (0 FP / 0 FN) results rather than evaluating the accuracy loss
  introduced by probabilistic thresholding with the adapted k-mer lemma.
- Your index uses uneven bin sizes or very high bin counts, where the paper notes
  the method's main limiting factors and results would not be comparable to Table
  3.
- You are benchmarking a different pre-filter (e.g., Mantis or COBS) whose output
  schema differs from Raptor's bin assignments without adapting the ground-truth comparison.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Raptor
  role: Builds the (23,19)-MIBF indices ('raptor build') and performs the minimizer-thresholded
    searches ('raptor search') whose bin assignments are scored as FP/FN against ground
    truth
  repo: https://github.com/seqan/raptor
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2021.102782
    title: 'Raptor: A fast and space-efficient pre-filter for querying very large
      collections of nucleotide sequences'
  source_content_hash: sha256:56d2b627786602d8012de5e30a48e52016e1fd55c10d2db96319f7b687ed91c2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seqan/raptor
  tools:
  - Raptor
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# false-positive-false-negative-benchmark-counting

## Summary

Benchmarking skill that quantifies the specificity (false positives) and sensitivity loss (false negatives) of Raptor's probabilistic thresholding (the adapted k-mer lemma, yielding a threshold of ≈41%) for (23,19)-minimizer IBF (MIBF) indices, reproducing the FP/FN counts of Table 3 of the Raptor paper.

## When to use

Use when you need to measure how an approximate sequence search index (e.g., a minimizer interleaved Bloom filter with probabilistic thresholding) trades off false-positive and false-negative bin assignments across different index sizes or bin counts.

## When NOT to use

- You need exact-match (0 FP / 0 FN) results rather than evaluating the accuracy loss introduced by probabilistic thresholding with the adapted k-mer lemma.
- Your index uses uneven bin sizes or very high bin counts, where the paper notes the method's main limiting factors and results would not be comparable to Table 3.
- You are benchmarking a different pre-filter (e.g., Mantis or COBS) whose output schema differs from Raptor's bin assignments without adapting the ground-truth comparison.

## Inputs

- Artificial dataset (64 and 1,024 bins) distributed with the Raptor repository
- Simulated reads with known ground-truth bin assignments
- Minimizer IBF (MIBF) indices built with (23,19)-minimizers at 1, 2, 4, and 8 GiB for 64 and 1,024 bins

## Outputs

- FP/FN counts per index configuration (false-positive and false-negative read-to-bin assignments)
- fp_fn_table.tsv reproducing Table 3

## How to apply

Obtain the artificial dataset distributed with the Raptor repository (64 and 1,024 bins) and install Raptor via bioconda. Build minimizer IBF (MIBF) indices with the (23,19)-minimizer scheme at sizes of 1, 2, 4, and 8 GiB for both 64 and 1,024 bins using 'raptor build', then search the simulated reads against each of the eight indices with 'raptor search' using the probabilistic (adapted k-mer lemma) minimizer threshold (≈41%). Compare each search's reported bin assignments against the ground-truth bin of every read: false positives are reads reported in bins that do not contain them, false negatives are reads missing from their true bin. Assemble the FP/FN counts into a table matching Table 3 (fp_fn_table.tsv); correctness is validated when the 64-bin FN counts reproduce the reported values 96/37/22/24 for the 1/2/4/8 GiB indices.

## Related tools

- **Raptor** (Builds the (23,19)-MIBF indices ('raptor build') and performs the minimizer-thresholded searches ('raptor search') whose bin assignments are scored as FP/FN against ground truth) — https://github.com/seqan/raptor

## Examples

```
conda install -c bioconda -c conda-forge raptor; raptor build --kmer 23 --window 19 --size 4gib --bins 64 --output mibf_64bin_4gib; raptor search --index mibf_64bin_4gib --reads sim_reads.fq --threshold probabilistic
```

## Evaluation signals

- The 64-bin FN counts reproduce the reported values 96/37/22/24 for the 1/2/4/8 GiB MIBF indices.
- The resulting probabilistic threshold is ≈41%, matching the adapted k-mer lemma configuration in Table 3.
- Eight index configurations are evaluated (64 and 1,024 bins × 1, 2, 4, 8 GiB) and each yields both an FP and an FN count.
- Every read's reported bin assignments are compared to exactly one ground-truth bin, so FP + FN counts partition the observed misassignments consistently across configurations.
- Output table matches the structure of Table 3 (rows = index sizes, columns = 64 vs 1,024 bins).

## Limitations

- P(Cn) has no tractable formulation; the thresholding behavior is estimated experimentally by sampling, so exact FP/FN values depend on the dataset and sampled threshold.
- Raptor's main limiting factor is its susceptibility to uneven bin sizes as well as a high number of bins, which can inflate FP/FN counts outside this controlled artificial setup.
- Indices built with an older version of raptor cannot be used with version 2.0.0, so index version must match the search binary.

## Evidence

- [abstract] Raptor uses winnowing minimizers and probabilistic thresholding: "Raptor uses winnowing minimizers to define a set of representative k-mers, an extension of the interleaved Bloom filters (IBFs) as a set membership data structure and probabilistic thresholding for"
- [readme] conda install raptor via bioconda: "conda install -c bioconda -c conda-forge raptor"
- [methods] raptor commands have help pages: "Each command will have a respective help page that can be shown via, e.g., `raptor build --help`."
- [discussion] susceptibility to uneven bin sizes and high bin counts: "The main limiting factor of our method is its susceptibility to uneven bin sizes as well as a high number of bins."
- [discussion] P(Cn) not tractable, estimated experimentally: "We tried to find a tractable formulation of Cn, but did not succeed and leave this as an open problem."
