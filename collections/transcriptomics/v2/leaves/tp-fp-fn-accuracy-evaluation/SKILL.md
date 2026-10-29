---
name: tp-fp-fn-accuracy-evaluation
description: Use when a locus- or family-level TE quantification tool needs an accuracy
  benchmark against simulated ground-truth data, especially where multi-mapping of
  young TEs complicates read assignment.
when_to_use_negative:
- Analyzing real (non-simulated) scRNA-Seq data where no ground-truth TE expression
  profile exists — TP/FP/FN labels cannot be assigned
- The tool's output is already aggregated at family level only and the question is
  locus-level accuracy, since FP/FN per locus cannot be resolved from metagene counts
- Input BAMs were aligned without multi-mapping-aware settings, so the multi-mapping
  conditions the benchmark is designed to stress are absent
edam_operation: http://edamontology.org/operation_0253
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: Minnow
  role: Simulates scRNA-Seq reads from ground-truth TE expression profiles to create
    the benchmark truth set
- name: STAR
  role: Aligns simulated reads with multi-mapping options --winAnchorMultimapNmax
    100, --outFilterMultimapNmax 100, --outSAMmultNmax 1 to produce BAM input
- name: SoloTE
  role: Method under evaluation; produces locus-level TE expression matrices from
    BAM files
  repo: https://github.com/bvaldebenitom/SoloTE
- name: scTE
  role: Comparator method; produces family-level (metagene) TE count matrices from
    the same BAMs
- name: BEDTools
  role: Used by SoloTE to assess read overlap with the TE annotation during quantification
  repo: https://github.com/arq5x/bedtools2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-022-04020-5
    title: SoloTE for improved analysis of transposable elements in single-cell RNA-Seq
      data using locus-specific expression
  source_content_hash: sha256:a501a92ad38cc5784d0ba581e7bcd74475be7d5b7d4d4756add5dcd163caafc8
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
  edam_operation: http://edamontology.org/operation_0253
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# tp-fp-fn-accuracy-evaluation

## Summary

Benchmark a TE-expression quantification method (e.g., SoloTE vs scTE) by simulating scRNA-Seq reads with known ground-truth TE expression and classifying each predicted expressed TE locus as True Positive, False Positive, or False Negative, with separate assessment of old (>10% divergence) vs young (≤10% divergence) TEs.

## When to use

Use when a locus- or family-level TE quantification tool needs an accuracy benchmark against simulated ground-truth data, especially where multi-mapping of young TEs complicates read assignment.

## When NOT to use

- Analyzing real (non-simulated) scRNA-Seq data where no ground-truth TE expression profile exists — TP/FP/FN labels cannot be assigned
- The tool's output is already aggregated at family level only and the question is locus-level accuracy, since FP/FN per locus cannot be resolved from metagene counts
- Input BAMs were aligned without multi-mapping-aware settings, so the multi-mapping conditions the benchmark is designed to stress are absent

## Inputs

- RepeatMasker-format TE annotations for hg38 and mm10 (BED with column-4 ID: locus|Subfamily:Family:Class|strand)
- Ground-truth TE expression profiles split into old (>10% divergence) and young (≤10% divergence) groups
- Simulated scRNA-Seq BAM files produced by Minnow + STAR (multi-mapping options enabled)
- Output expression matrices from the method under evaluation (SoloTE locus-level) and comparator (scTE family-level metagenes)

## Outputs

- TP/FP/FN classification per simulated expressed TE locus per method
- Aggregated benchmark table (e.g., benchmark_tp_fp_fn.csv) with 8 rows: 2 genomes × 2 TE groups (young/old) × 2 methods

## How to apply

Build ground truth by deriving TE expression profiles from RepeatMasker-format annotations (column-4 ID: locus|Subfamily:Family:Class|strand) and splitting TEs into young (divergence from consensus ≤10%) and old (>10%) groups. Simulate scRNA-Seq reads with Minnow from those profiles for each genome (hg38, mm10), then align with STAR using the multi-mapping options --winAnchorMultimapNmax 100, --outFilterMultimapNmax 100, and --outSAMmultNmax 1 to produce BAM files. Run the tool under evaluation (e.g., 'python SoloTE_pipeline.py --threads N --bam file --teannotation BEDfile --outputprefix P --outputdir D') and the comparator (scTE, family-level metagenes) on the same BAMs. Compare each method's output matrix to the Minnow ground truth, labeling each simulated expressed TE locus as TP, FP, or FN. Aggregate into a table like benchmark_tp_fp_fn.csv with 8 rows (2 genomes × 2 TE groups × 2 methods) and interpret elevated FP/FN rates in the young TE group as the key failure mode.

## Related tools

- **Minnow** (Simulates scRNA-Seq reads from ground-truth TE expression profiles to create the benchmark truth set)
- **STAR** (Aligns simulated reads with multi-mapping options --winAnchorMultimapNmax 100, --outFilterMultimapNmax 100, --outSAMmultNmax 1 to produce BAM input)
- **SoloTE** (Method under evaluation; produces locus-level TE expression matrices from BAM files) — https://github.com/bvaldebenitom/SoloTE
- **scTE** (Comparator method; produces family-level (metagene) TE count matrices from the same BAMs)
- **BEDTools** (Used by SoloTE to assess read overlap with the TE annotation during quantification) — https://github.com/arq5x/bedtools2

## Examples

```
python SoloTE_pipeline.py --threads 8 --bam simulated_hg38.bam --teannotation hg38_te.bed --outputprefix sim_hg38 --outputdir results/
```

## Evaluation signals

- Output CSV contains exactly 8 aggregated rows: 2 genomes (hg38, mm10) × 2 TE groups (young/old) × 2 methods (SoloTE, scTE)
- Every simulated expressed TE locus is labeled TP, FP, or FN with no unlabeled entries; TP + FN equals the number of simulated expressed loci per condition
- Young TE group (≤10% divergence) shows the elevated FP/FN pattern described for the hardest-to-map TEs, serving as a sanity check on the benchmark's sensitivity
- Both methods consume identical STAR-aligned BAMs (same alignment parameters), so TP/FP/FN differences are attributable to quantification, not alignment
- SoloTE locus-level output IDs match the BED column-4 format locus|Subfamily:Family:Class|strand, enabling direct locus-wise comparison to ground truth

## Limitations

- Young TEs are the most problematic: SoloTE showed a higher proportion of false positives and false negatives for young TEs, whose multi-mapping complicates locus-specific assignment
- With short 100-nucleotide 3'-end 10X reads, not all TEs can be uniquely mappable, biasing locus-level accuracy for very young elements
- SoloTE does not yet integrate an Expectation–Maximization algorithm for multi-mapping read allocation, limiting multi-mapping read resolution
- SoloTE's locus-level approach assumes most TE copies are divergent; in organisms with a greater proportion of transpositionally-active TEs, most copies could be genetically intact and multi-mapping

## Evidence

- [other] In the simulated hg38 and mm10 datasets, SoloTE showed a higher proportion of false positives and false negatives for young TEs: [evidence span withheld — anchor status: not_found]
- [discussion] TEs classified as young/old by divergence: "TEs were divided into young and old, if their divergence from the consensus sequence was ≤10% or >10%, respectively"
- [discussion] STAR alignment options: [evidence span withheld — anchor status: not_found]
- [discussion] SoloTE output matrix compatibility: "a new matrix having both genes and TEs expression per cell is obtained as output, which is compatible with downstream analysis tools, such as Seurat67"
- [readme] SoloTE CLI usage: "python SoloTE_pipeline.py --threads NumberOfThreads --bam BAMfile --teannotation BEDfile --outputprefix Prefix --outputdir OutputDirectory"
