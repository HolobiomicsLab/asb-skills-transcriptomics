---
name: te-expression-quantification-benchmarking
description: Use when validating or comparing a single-cell RNA-Seq transposable element
  quantification method against known ground truth, especially when assessing accuracy
  on repetitive young TEs where multi-mapping read assignment is problematic.
when_to_use_negative:
- Input data is real (non-simulated) scRNA-Seq — no ground truth exists, so TP/FP/FN
  classification against Minnow profiles is not applicable.
- The goal is gene expression quantification rather than transposable element expression,
  or the quantification target is gene-TE linear modeling rather than method accuracy.
- Evaluating a method that cannot process multi-mapping alignments (e.g. tools requiring
  uniquely mapped reads only), since the benchmark hinges on multi-mapped reads.
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0108
tools:
- name: SoloTE
  role: Candidate method under benchmark; produces locus-level TE expression matrices
    from BAM files
  repo: https://github.com/bvaldebenitom/SoloTE
- name: scTE
  role: Comparison baseline method; produces family-level (metagene) TE count matrices
- name: Minnow
  role: Simulates scRNA-Seq reads from ground-truth TE expression profiles
- name: STAR
  role: Aligns simulated reads with multi-mapping options --winAnchorMultimapNmax
    100, --outFilterMultimapNmax 100, --outSAMmultNmax 1
- name: BEDTools
  role: Used by SoloTE for TE-annotation overlap assessment within its pipeline invoked
    in benchmarking
  repo: https://github.com/arq5x/bedtools2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-022-04020-5
    title: SoloTE for improved analysis of transposable elements in single-cell RNA-Seq
      data using locus-specific expression
  source_content_hash: sha256:8424cd27e6403518b0e05accdc794ef2154a69d2ade9d09890dc292bef6d5c9e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bvaldebenitom/SoloTE
  tools:
  - BEDTools
  - Minnow
  - STAR
  - SoloTE
  - scTE
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0108
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# te-expression-quantification-benchmarking

## Summary

Benchmarks TE expression quantification methods (SoloTE vs scTE) by comparing tool outputs against simulated scRNA-Seq ground truth, classifying expressed TE loci as TP/FP/FN, with special attention to old (>10% divergence) versus young (≤10% divergence) TEs where multi-mapping confounds locus-specific assignment.

## When to use

Use when validating or comparing a single-cell RNA-Seq transposable element quantification method against known ground truth, especially when assessing accuracy on repetitive young TEs where multi-mapping read assignment is problematic.

## When NOT to use

- Input data is real (non-simulated) scRNA-Seq — no ground truth exists, so TP/FP/FN classification against Minnow profiles is not applicable.
- The goal is gene expression quantification rather than transposable element expression, or the quantification target is gene-TE linear modeling rather than method accuracy.
- Evaluating a method that cannot process multi-mapping alignments (e.g. tools requiring uniquely mapped reads only), since the benchmark hinges on multi-mapped reads.

## Inputs

- RepeatMasker-format TE annotations (.rm.out or BED) for hg38 and mm10 with column-4 IDs of form locus|Subfamily:Family:Class|strand
- Ground-truth TE expression profiles stratified by divergence (young ≤10%, old >10%)
- Simulated scRNA-Seq BAM files aligned with STAR multi-mapping options
- Per-tool TE expression matrices (locus-level from SoloTE, family-level metagene from scTE)

## Outputs

- benchmark_tp_fp_fn.csv aggregating TP/FP/FN counts across 2 genomes × 2 TE groups × 2 methods
- Per-locus TP/FP/FN classifications for each simulated expressed TE locus

## How to apply

Derive ground-truth TE expression profiles from hg38 or mm10 RepeatMasker-format annotations (column-4 ID formatted as locus|Subfamily:Family:Class|strand), stratifying TEs into young (divergence from consensus ≤10%) and old (>10%) groups. Simulate scRNA-Seq reads with Minnow from those profiles, then align with STAR using the multi-mapping options --winAnchorMultimapNmax 100, --outFilterMultimapNmax 100, and --outSAMmultNmax 1 to produce BAM files. Run each candidate tool on the same BAM files — SoloTE via 'SoloTE_pipeline.py' for locus-level matrices, scTE for family-level (metagene) matrices. Compare each method's TE expression matrix to the Minnow ground truth, classifying each simulated expressed TE locus as True Positive, False Positive, or False Negative, then aggregate into rows stratified by genome (hg38/mm10) × TE group (young/old) × method.

## Related tools

- **SoloTE** (Candidate method under benchmark; produces locus-level TE expression matrices from BAM files) — https://github.com/bvaldebenitom/SoloTE
- **scTE** (Comparison baseline method; produces family-level (metagene) TE count matrices)
- **Minnow** (Simulates scRNA-Seq reads from ground-truth TE expression profiles)
- **STAR** (Aligns simulated reads with multi-mapping options --winAnchorMultimapNmax 100, --outFilterMultimapNmax 100, --outSAMmultNmax 1)
- **BEDTools** (Used by SoloTE for TE-annotation overlap assessment within its pipeline invoked in benchmarking) — https://github.com/arq5x/bedtools2

## Examples

```
python SoloTE_pipeline.py --threads 8 --bam simulated_hg38.bam --teannotation hg38_te.bed --outputprefix hg38_sim --outputdir ./out
```

## Evaluation signals

- Comparison of each method's TE expression matrix against Minnow ground truth yields non-zero TP, FP, and FN counts for every genome × TE-group × method combination in benchmark_tp_fp_fn.csv.
- The 8-row aggregation (2 genomes × 2 TE groups × 2 methods) is fully populated with no missing rows.
- Simulated reads align successfully with STAR using all three multi-mapping options (--winAnchorMultimapNmax 100, --outFilterMultimapNmax 100, --outSAMmultNmax 1), producing valid BAMs.
- TP/FP/FN classification is applied consistently to both locus-level (SoloTE) and family-level (scTE) output matrices against the same Minnow ground truth.
- Young (≤10% divergence) and old (>10% divergence) TE groups are correctly stratified in the output table.

## Limitations

- SoloTE showed a higher proportion of false positives and false negatives for young TEs, which are the most problematic TEs to assess in terms of genomic locations.
- Multi-mapping confounds locus-specific read assignment for TEs with low divergence from consensus; simulated accuracy may not transfer to organisms with a greater proportion of transpositionally-active, genetically intact TE copies.
- Comparison is on simulated data (hg38/mm10), not experimentally validated biological data.

## Evidence

- [results] Simulated hg38/mm10 benchmark design: "we simulated data from the human genome hg38 version, and from the mouse genome mm10 version"
- [discussion] Young/old stratification threshold: "TEs were divided into young and old, if their divergence from the consensus sequence was ≤10% or >10%, respectively"
- [discussion] STAR multi-mapping options: [evidence span withheld — anchor status: not_found]
- [readme] SoloTE CLI invocation: "python SoloTE_pipeline.py --threads NumberOfThreads --bam BAMfile --teannotation BEDfile --outputprefix Prefix --outputdir OutputDirectory"
- [readme] BEDTools dependency in the pipeline: "BEDTools v2.29.2 or higher (https://github.com/arq5x/bedtools2/releases)"
- [intro] scTE family-level limitation motivating benchmark: "which summarizes TE expression on metagenes corresponding to TE families, thus, losing the genomic location from which each TE is expressed"
