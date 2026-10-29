---
name: multi-mapping-read-allocation-benchmarking
description: Use when benchmarking single-cell TE quantification methods that must
  resolve multi-mapping reads (e.g., Alu subfamilies, HERVK isoforms) across scRNA-seq,
  scATAC-seq, or long-read datasets, using ARI/NMI clustering or simulation ground-truth
  accuracy as the comparison metric.
when_to_use_negative:
- Input is already a quantified TE feature/count table — this skill benchmarks read-allocation
  from aligned BAMs, not downstream matrix analysis
- Target TEs are long autonomously expressed elements like L1HS spanning several thousand
  bases, where MATES coverage-vector context is incomplete and estimated multi-mapping
  rates may be inaccurate
- The task requires distinguishing nearly identical TE copies (e.g., HERVK isoforms)
  lacking uniquely mappable flanking regions, which all compared methods handle poorly
edam_operation: http://edamontology.org/operation_0571
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0091
tools:
- name: MATES
  role: Deep-learning method under test that allocates multi-mapping reads to TE loci
    via flanking coverage vectors and an MLP-estimated multi-mapping rate; provides
    bam_processor, data_processor, MATES_model, TE_quantifier, TE_quantifier_LongRead
  repo: https://github.com/mcgilldinglab/MATES
- name: scTE
  role: Single-cell TE quantification baseline for comparison of clustering ARI/NMI
    and simulation read-recapture percentages
- name: SoloTE
  role: Single-cell TE quantification baseline for clustering comparison and locus-level
    comparison on shared datasets and long-read simulation R2
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-53114-7
    title: 'MATES: a deep learning-based model for locus-specific quantification of
      transposable elements in single cell'
  source_content_hash: sha256:26a0a6dcd56e3a202293e0a5bfc322af042ed3cffa8acdabfa3d7f25165eba33
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/mcgilldinglab/MATES
  repo_url: https://github.com/mcgilldinglab/MATES
  tools:
  - MATES
  - MATES_model
  - SoloTE
  - TE_quantifier
  - TE_quantifier_LongRead
  - bam_processor
  - data_processor
  - scTE
  edam_operation: http://edamontology.org/operation_0571
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# multi-mapping-read-allocation-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark locus-specific allocation of multi-mapping reads to transposable elements (TEs) in single-cell data, comparing MATES against alignment-based methods (scTE, SoloTE) via cell-clustering metrics (ARI/NMI) and simulation-grounded quantification accuracy (R2, percent reads recaptured).

## When to use

Use when benchmarking single-cell TE quantification methods that must resolve multi-mapping reads (e.g., Alu subfamilies, HERVK isoforms) across scRNA-seq, scATAC-seq, or long-read datasets, using ARI/NMI clustering or simulation ground-truth accuracy as the comparison metric.

## When NOT to use

- Input is already a quantified TE feature/count table — this skill benchmarks read-allocation from aligned BAMs, not downstream matrix analysis
- Target TEs are long autonomously expressed elements like L1HS spanning several thousand bases, where MATES coverage-vector context is incomplete and estimated multi-mapping rates may be inaccurate
- The task requires distinguishing nearly identical TE copies (e.g., HERVK isoforms) lacking uniquely mappable flanking regions, which all compared methods handle poorly

## Inputs

- BAM files of single-cell aligned reads (10X-format multi-cell BAMs or Smart-seq2 one-BAM-per-cell) from scRNA-seq, scATAC-seq, or long-read (scNanoRNAseq/PacBio) platforms
- TE reference annotation (e.g., TE_nooverlap.bed / TE_full.bed, RepeatMasker-derived) and gene/peak references
- Simulated reads with known ground-truth locus abundances (Polyester, e.g., 6 Alu subfamilies, HERVK TE1/TE2 isoform pair, 100-bp reads)
- Known cell type labels for clustering evaluation

## Outputs

- Locus-specific and subfamily-level TE count matrices per cell for MATES, scTE, and SoloTE
- Cell-clustering ARI and NMI distributions per method per perspective (Gene/Peak+TE, TE-only, Multi-mapping-TE-only) with one-sided t-test P values
- Simulation benchmark results (e.g., simulation_benchmark_results.csv) with locus-level R2, percentage of simulated reads captured, and isoform-level counts

## How to apply

Map raw reads with STAR/STARsolo, then segregate unique- vs multi-mapping reads per cell using MATES bam_processor (data_mode 10X or Smart_seq). Quantify TE loci with MATES, which builds unique (Vu) and multi-mapping (Vm) coverage vectors spanning 2001 bp centered on each TE start position, trains an AutoEncoder on unique-read vectors with TE family one-hot encoding, and estimates a locus-specific multi-mapping rate alpha_i via an MLP so total counts are URi + ceil(alpha_i x MRi). Run scTE and SoloTE on the identical BAM inputs, then build feature matrices under three perspectives (Gene/Peak+TE, TE-only, Multi-mapping-TE-only). Cluster cells (Louvain/Leiden in Scanpy) and compute Adjusted Rand Index and Normalized Mutual Information against known labels over N=10 random seeds; compare methods with one-sided Student's t-tests. For simulation-based validation, generate Polyester reads with known ground truth and score locus-level R2 correlations and percentage of simulated reads captured per method.

## Related tools

- **MATES** (Deep-learning method under test that allocates multi-mapping reads to TE loci via flanking coverage vectors and an MLP-estimated multi-mapping rate; provides bam_processor, data_processor, MATES_model, TE_quantifier, TE_quantifier_LongRead) — https://github.com/mcgilldinglab/MATES
- **scTE** (Single-cell TE quantification baseline for comparison of clustering ARI/NMI and simulation read-recapture percentages)
- **SoloTE** (Single-cell TE quantification baseline for clustering comparison and locus-level comparison on shared datasets and long-read simulation R2)

## Examples

```
from MATES import MATES_pipeline; mates = MATES_pipeline(TE_mode, data_mode, sample_list_file, bam_path_file); mates.preprocessing(); mates.run()
```

## Evaluation signals

- MATES ranks first across Gene/Peak+TE, TE-only, and Multi-mapping-TE-only clustering perspectives (Fig. 6a, b); reproduce with ARI/NMI over 10 random seeds and one-sided t-tests
- Locus-level MATES-vs-SoloTE check: MATES achieves ~10.52% higher ARI with one-sided t-test P ≈ 2.60e-12
- Simulation acceptance: MATES Alu R2 = 0.7420 with 96.14% of simulated reads captured (vs scTE 92.57%); simple-repeat locus R2 = 0.7479
- HERVK isoform counts match ground truth closely: MATES 4140/1295 vs ground truth 4126/1282 (TE1/TE2)
- Long-read simulation locus-level R2 = 0.4923 (MATES) vs 0.1178 (SoloTE strategy)

## Limitations

- MATES requires more computational resources (e.g., GPUs) than scTE and SoloTE for TE quantification
- Limited ability to quantify long autonomously expressed TEs such as L1HS; multi-mapping rate estimates may be inaccurate for TEs longer than 2001 bp due to incomplete flanking coverage
- Identical TE copies like HERVK isoforms with high sequence similarity and no uniquely mappable flanking regions are hard to distinguish

## Evidence

- [readme] bam_processor segregates unique vs multi-mapping reads per cell: [evidence span withheld — anchor status: external_doc]
- [results] MATES outperformed scTE and SoloTE in TE-only and multi-mapping-TE-only clustering: [evidence span withheld — anchor status: not_found]
