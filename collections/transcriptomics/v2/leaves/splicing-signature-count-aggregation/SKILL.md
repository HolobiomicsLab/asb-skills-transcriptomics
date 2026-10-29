---
name: splicing-signature-count-aggregation
description: Use when you already hold fortuna signature-count output (cnt.tsv with
  '|' joined subexon IDs and counts) and need DEXSeq-compatible per-subexon counts
  for alternative splicing analysis.
when_to_use_negative:
- If you only have raw FASTQ reads and no fortuna signature counts — you must first
  run the fortuna pipeline (processGTF.sh, --index, pseudoalignment) to produce cnt.tsv.
- If you need kallisto TCCs or Yanagi bins rather than DEXSeq per-subexon counts —
  this aggregation targets DEXSeq-style counting units specifically.
- If your counts come from an align-and-count workflow rather than fortuna's pseudoalignment
  equivalence classes — the signature-to-subexon semantics differ.
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3320
- http://edamontology.org/topic_0203
tools:
- name: fortuna
  role: Provides the '--trans' command that aggregates '-incnt' signature counts into
    '-outcnt' per-subexon (DEXSeq-style) counts
  repo: https://github.com/canzarlab/fortuna
- name: kallisto
  role: Underlying pseudoaligner whose equivalence classes yield the elementary signature
    counts that are aggregated
  repo: https://github.com/pachterlab/kallisto
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad419
    title: Counting pseudoalignments to novel splicing events
  source_content_hash: sha256:2dab7ed93c8fcf32f6e86aa149b8f9e0072dfca8136cae28fa87d9403460b530
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/canzarlab/fortuna
  tools:
  - fortuna
  - kallisto
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3320
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# splicing-signature-count-aggregation

## Summary

Converts fortuna's elementary splicing signature counts (pipe-separated subexon IDs with read counts, e.g. cnt.tsv) into DEXSeq-style per-subexon counts by aggregating each subexon's count as the sum of all mapping signatures that contain it, without reprocessing raw reads or alignments.

## When to use

Use when you already hold fortuna signature-count output (cnt.tsv with '|' joined subexon IDs and counts) and need DEXSeq-compatible per-subexon counts for alternative splicing analysis.

## When NOT to use

- If you only have raw FASTQ reads and no fortuna signature counts — you must first run the fortuna pipeline (processGTF.sh, --index, pseudoalignment) to produce cnt.tsv.
- If you need kallisto TCCs or Yanagi bins rather than DEXSeq per-subexon counts — this aggregation targets DEXSeq-style counting units specifically.
- If your counts come from an align-and-count workflow rather than fortuna's pseudoalignment equivalence classes — the signature-to-subexon semantics differ.

## Inputs

- fortuna signature-counts table (cnt.tsv): tab-separated lines of '|'-joined subexon IDs (NodeId from the processGTF.sh-segmented GTF) followed by an integer read count

## Outputs

- DEXSeq-style per-subexon count table (tab-separated, '-outcnt' output), one count per subexon, each equal to the sum of counts of all containing signatures

## How to apply

Load the signature-counts table (e.g. cnt.tsv) in which each tab-separated line holds a mapping signature — a sequence of subexon IDs from the segmented GTF joined by '|' — followed by the read count. Run fortuna's '--trans' command with '-incnt' pointing to the signature-counts file and '-outcnt' to the output file; fortuna aggregates signature counts into the larger per-subexon counting units used by DEXSeq. The rationale is that the signature counts are the most elementary splicing units, and DEXSeq counts are simply a coarser aggregation over them. Validate on a small hand-checkable example (e.g. the Fig. 2 subexon configuration) that every output subexon count equals the sum of all signature counts whose '|'-separated signature contains that subexon ID. Subexon IDs originate from the GTF file pre-processed with bin/processGTF.sh (field NodeId), so the output stays consistent with that segmented annotation.

## Related tools

- **fortuna** (Provides the '--trans' command that aggregates '-incnt' signature counts into '-outcnt' per-subexon (DEXSeq-style) counts) — https://github.com/canzarlab/fortuna
- **kallisto** (Underlying pseudoaligner whose equivalence classes yield the elementary signature counts that are aggregated) — https://github.com/pachterlab/kallisto

## Examples

```
fortuna --trans -incnt cnt.tsv -outcnt subexon_counts.tsv
```

## Evaluation signals

- Output is a tab-separated table with one integer count per subexon ID from the segmented GTF.
- Hand-check on the Fig. 2 subexon configuration: each subexon count equals the sum of all signature counts whose '|'-separated signature contains that subexon.
- Sum of per-subexon counts is consistent with the underlying signature counts (no signatures dropped or double-counted).
- Subexon IDs in the output match the NodeId field of the processGTF.sh-preprocessed input GTF (e.g. IDs like 223832 in cnt.tsv).

## Limitations

- Aggregation is only valid for counts produced by fortuna from kallisto equivalence classes over a processGTF.sh-segmented annotation; signature semantics from other tools will not map correctly.
- The aggregation discards signature-level resolution — reads shared across subexons are counted for every subexon they contain, which is the DEXSeq convention, not an isoform-resolution count.
- Fortuna requires C++11, CMake >= 2.8.12, zlib and autoconf 2.69 to compile; Mac OS users may need 'brew install autoconf@2.69'.

## Evidence

- [full_text] DEXSeq-style counts are derived from signature counts by a simple aggregation: for every subexon s, sum the counts of all mapping signatures that contain that subexon.: [evidence span withheld — anchor status: not_found]
- [full_text] Run fortuna '--trans' with '-incnt' and '-outcnt': [evidence span withheld — anchor status: not_found]
- [readme] cnt.tsv signature format: "The above example implies that a signature comprised of subexons 223832, 223833, and 223836 has a count of 2274."
- [intro] counts can be aggregated to DEXSeq units: "Computed counts can be directly used for alternative splicing analysis or added up to larger units such as those used by DEXSeq [3] (E1)."
