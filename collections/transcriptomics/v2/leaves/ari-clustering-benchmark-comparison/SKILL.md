---
name: ari-clustering-benchmark-comparison
description: Use when comparing TE quantification methods on scRNA-seq data in terms
  of downstream cell-clustering fidelity, e.g., testing whether a new method (MATES)
  beats scTE/SoloTE on ARI against gene-defined clusters.
when_to_use_negative:
- Input is already a feature table or published ARI values — this skill requires raw
  alignments re-quantified by each competing method
- The quantification target is long autonomously expressed TEs such as L1HS spanning
  several thousand bases, where MATES coverage-vector flanking context is incomplete
- The data come from mammalian species for which MATES auto-builds references — this
  skill targets the non-mammalian (Arabidopsis/Drosophila) extension path requiring
  custom reference building
tools:
- name: MATES
  role: Locus-specific TE quantification producing cell-by-TE count matrices for the
    benchmarked method (TE_quantifier module)
  repo: https://github.com/mcgilldinglab/MATES
- name: scTE
  role: Existing single-cell TE quantification method used as a baseline in the ARI
    comparison
- name: SoloTE
  role: Existing single-cell TE quantification method used as a baseline in the ARI
    comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-53114-7
    title: 'MATES: a deep learning-based model for locus-specific quantification of
      transposable elements in single cell'
  source_content_hash: sha256:15884a856f8b08b1fd85cdc994fef965a6b0c74458874526fe3e3c81dbd18dc3
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
  - SoloTE
  - scTE
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# ari-clustering-benchmark-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark locus-specific single-cell TE quantification methods (MATES, scTE, SoloTE) by clustering cells on TE features and scoring agreement with gene-expression-based reference clustering via Adjusted Rand Index (ARI), with block-permutation empirical p-values and marker-TE recovery.

## When to use

Use when comparing TE quantification methods on scRNA-seq data in terms of downstream cell-clustering fidelity, e.g., testing whether a new method (MATES) beats scTE/SoloTE on ARI against gene-defined clusters.

## When NOT to use

- Input is already a feature table or published ARI values — this skill requires raw alignments re-quantified by each competing method
- The quantification target is long autonomously expressed TEs such as L1HS spanning several thousand bases, where MATES coverage-vector flanking context is incomplete
- The data come from mammalian species for which MATES auto-builds references — this skill targets the non-mammalian (Arabidopsis/Drosophila) extension path requiring custom reference building

## Inputs

- scRNA-seq alignments (BAM files) for Arabidopsis thaliana and Drosophila melanogaster
- Cell-by-TE count matrices produced by MATES, scTE, and SoloTE
- Gene-expression count matrices used to define the reference clustering
- TE and gene reference genome files (GTF + RepeatMasker) for non-mammalian species

## Outputs

- ARI values per method, species, and feature setting (TE-only and Gene+TE)
- Empirical p-values from block-permutation tests of each ARI
- List of marker TEs for recovered cell populations (e.g., ATCopia66LTR, ATCopia41LTR, GYPSY12_LTR)
- Comparison results table (e.g., ari_comparison_results.csv)

## How to apply

Quantify TE expression at locus-specific level for each alignment using MATES (TE_quantifier module), scTE, and SoloTE, then build cell-by-TE count matrices for two feature settings per dataset: TE-only counts and combined gene-expression + TE counts. Cluster cells in each setting (Leiden/Louvain community detection on the kNN graph) and compute the ARI of each TE-based clustering against the gene-expression-based reference clustering. Assess significance of each ARI with block-permutation tests, computing empirical p-values by permuting cluster labels in blocks to preserve local structure. Identify marker TEs for recovered populations (e.g., ATCopia66LTR, ATCopia41LTR in Arabidopsis; GYPSY12_LTR in Drosophila) and check they match the gene-defined populations. Finally, assemble all ARIs, p-values, and markers into a results table and verify each reproduced value matches the reported figure (MATES Arabidopsis TE-only ARI 0.3514, Gene+TE ARI 0.6668; SoloTE 0.3200/0.5916; scTE 0.3315/0.5775; Drosophila MATES 0.3188 vs scTE 0.3078) within a small numerical tolerance.

## Related tools

- **MATES** (Locus-specific TE quantification producing cell-by-TE count matrices for the benchmarked method (TE_quantifier module)) — https://github.com/mcgilldinglab/MATES
- **scTE** (Existing single-cell TE quantification method used as a baseline in the ARI comparison)
- **SoloTE** (Existing single-cell TE quantification method used as a baseline in the ARI comparison)

## Examples

```
from MATES import MATES_pipeline; mates = MATES_pipeline(TE_mode, data_mode, sample_list_file, bam_path_file); mates.run()
```

## Evaluation signals

- Reproduced MATES Arabidopsis ARI values match reported 0.3514 (TE-only) and 0.6668 (Gene+TE) within tolerance; SoloTE 0.3200/0.5916 and scTE 0.3315/0.5775 likewise
- Drosophila TE-only clustering gives MATES ARI = 0.3188 vs scTE ARI = 0.3078
- MATES ARIs exceed both baseline methods under each feature setting on both species
- Block-permutation tests yield empirical p-values for every reported ARI
- Marker TE list contains ATCopia66LTR and ATCopia41LTR (Arabidopsis) and GYPSY12_LTR (Drosophila), consistent with gene-defined cell populations

## Limitations

- MATES requires more computational resources (GPUs) than its counterparts scTE and SoloTE for quantifying TEs
- Challenges distinguishing identical TE copies such as HERVK isoforms due to high sequence similarity and lack of uniquely mappable flanking regions
- Multi-mapping rate estimation may be inaccurate for TEs longer than 2001 bp due to incomplete flanking coverage

## Evidence

- [intro] Non-mammalian extension of the benchmark: "These advancements must also extend to non-mammalian species, where TE dynamics play a crucial role in understanding broader biological phenomena"
- [abstract] MATES allocates multi-mapping reads to loci: "here we introduce MATES, a deep-learning approach that accurately allocates multi-mapping reads to specific loci of TEs"
- [readme] Marker TE identification supports clustering benchmark: "MATES shows improved performance over existing methods, enhancing the accuracy of TE quantification and aiding in the identification of marker TEs for identified cell populations"
- [readme] Non-mammalian reference support: [evidence span withheld — anchor status: external_doc]
- [discussion] Computational cost relative to baselines: "typically requires more computational resources such as GPUs than its counterparts scTE and SoloTE for quantifying TEs"
