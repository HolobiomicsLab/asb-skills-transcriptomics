---
name: non-mammalian-scrnaseq-processing
description: Use when TE quantification must be benchmarked or applied on non-mammalian
  scRNA-seq data (e.g., Arabidopsis or Drosophila 10X alignments) and you need ARI-based
  comparison of TE-derived versus gene-derived cell clustering across MATES, scTE,
  and SoloTE.
when_to_use_negative:
- Input data is human or mouse scRNA/scATAC — the standard MATES build_reference.py
  pipeline applies and this non-mammalian benchmarking extension is unnecessary
- You only need bulk-level TE quantification — bulk methods (Squire, TEtranscripts,
  Telescope) exist for that and single-cell clustering agreement is not the goal
- Target TEs are long autonomously expressed elements (e.g., L1HS-like, several thousand
  bases) where MATES's 2001 bp coverage vectors yield unreliable multi-mapping rate
  estimates
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0749
tools:
- name: MATES
  role: Deep-learning method for locus-specific TE quantification; TE_quantifier module
    produces cell-by-TE count matrices from Arabidopsis/Drosophila alignments
  repo: https://github.com/mcgilldinglab/MATES
- name: scTE
  role: Baseline single-cell TE quantification method compared against MATES via clustering
    ARI
- name: SoloTE
  role: Baseline single-cell TE quantification method compared against MATES via clustering
    ARI (Arabidopsis)
- name: bedtools
  role: Used for optional trimming of the TE reference against the gene reference
    to remove overlaps
- name: samtools
  role: Required dependency installed alongside MATES for BAM processing
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-53114-7
    title: 'MATES: a deep learning-based model for locus-specific quantification of
      transposable elements in single cell'
  source_content_hash: sha256:248616da72eda3786c893630d3bd72e47fdfdc7d3647ad481433eb76ff517c53
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
  - bedtools
  - samtools
  - scTE
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# non-mammalian-scrnaseq-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarking locus-specific TE quantification from non-mammalian (Arabidopsis thaliana and Drosophila melanogaster) scRNA-seq by comparing MATES against scTE and SoloTE via cell-clustering agreement (ARI) with gene-expression reference clustering. Demonstrates that MATES's deep-learning multi-mapping read allocation extends beyond human/mouse to plant and fly single-cell data.

## When to use

Use when TE quantification must be benchmarked or applied on non-mammalian scRNA-seq data (e.g., Arabidopsis or Drosophila 10X alignments) and you need ARI-based comparison of TE-derived versus gene-derived cell clustering across MATES, scTE, and SoloTE.

## When NOT to use

- Input data is human or mouse scRNA/scATAC — the standard MATES build_reference.py pipeline applies and this non-mammalian benchmarking extension is unnecessary
- You only need bulk-level TE quantification — bulk methods (Squire, TEtranscripts, Telescope) exist for that and single-cell clustering agreement is not the goal
- Target TEs are long autonomously expressed elements (e.g., L1HS-like, several thousand bases) where MATES's 2001 bp coverage vectors yield unreliable multi-mapping rate estimates

## Inputs

- Aligned scRNA-seq BAM files for Arabidopsis thaliana and Drosophila melanogaster (10X format)
- Species-specific TE reference (RepeatMasker) and gene reference (GTF) for Arabidopsis/Drosophila
- Cell barcode lists matching each BAM/sample
- Gene-expression count matrices used as the reference clustering basis

## Outputs

- Cell-by-TE locus-specific count matrices for MATES, scTE, and SoloTE
- ARI values comparing TE-based (and Gene+TE) clustering to gene-expression reference clustering, with block-permutation empirical p-values
- Marker TE lists per cell population (e.g., ATCopia66LTR, ATCopia41LTR, GYPSY12_LTR)
- ari_comparison_results.csv aggregating ARIs, p-values, and marker TEs

## How to apply

First build species-specific TE/Gene references for Arabidopsis thaliana and Drosophila melanogaster (MATERS auto-generates only Human/Mouse; for Arabidopsis and Drosophila use the shared GTF, RepeatMasker files, and the build_reference tutorial script). Run the MATES TE_quantifier module to quantify locus-specific TE expression from the aligned BAMs, in parallel with scTE and SoloTE, producing cell-by-TE count matrices. Build two feature settings per dataset — TE-only and combined gene-expression + TE counts — then cluster cells (Leiden/Louvain on the kNN graph) and compute the Adjusted Rand Index of each TE-based clustering against the gene-expression reference clustering. Assess ARI significance with block-permutation tests that permute cluster labels in blocks to preserve local structure. Identify marker TEs per recovered population and verify expected markers (ATCopia66LTR, ATCopia41LTR in Arabidopsis; GYPSY12_LTR in Drosophila). Reproduced ARIs must match reported values within numerical tolerance (e.g., MATES Arabidopsis TE-only 0.3514, Gene+TE 0.6668; scTE 0.3315/0.5775; Drosophila MATES 0.3188 vs scTE 0.3078).

## Related tools

- **MATES** (Deep-learning method for locus-specific TE quantification; TE_quantifier module produces cell-by-TE count matrices from Arabidopsis/Drosophila alignments) — https://github.com/mcgilldinglab/MATES
- **scTE** (Baseline single-cell TE quantification method compared against MATES via clustering ARI)
- **SoloTE** (Baseline single-cell TE quantification method compared against MATES via clustering ARI (Arabidopsis))
- **bedtools** (Used for optional trimming of the TE reference against the gene reference to remove overlaps)
- **samtools** (Required dependency installed alongside MATES for BAM processing)

## Examples

```
from MATES import MATES_pipeline; mates = MATES_pipeline(TE_mode, data_mode, sample_list_file, bam_path_file); mates.preprocessing(); mates.run()
```

## Evaluation signals

- Reproduced MATES Arabidopsis ARIs within tolerance of reported values: TE-only 0.3514, Gene+TE 0.6668 (SoloTE 0.3200/0.5916; scTE 0.3315/0.5775)
- Drosophila TE-only clustering ARI for MATES matches 0.3188 (vs scTE 0.3078)
- Marker TE lists contain ATCopia66LTR and ATCopia41LTR (Arabidopsis) and GYPSY12_LTR (Drosophila), consistent with gene-defined cell populations
- Block-permutation tests yield empirical p-values for each ARI that support the reported ranking MATES > SoloTE > scTE on Arabidopsis Gene+TE
- All three methods' counts come from the same underlying alignments, so cell sets and clustering settings must be identical across methods before computing ARI

## Limitations

- MATES auto-generates TE/gene references only for Human and Mouse; Arabidopsis and Drosophila references must be assembled from a shared external folder (GTF + RepeatMasker + example script), adding setup overhead
- Multi-mapping rate estimation may be inaccurate for TEs longer than 2001 bp due to incomplete flanking coverage
- MATES requires more computational resources (e.g., GPUs) than scTE and SoloTE for TE quantification
- Trimming TE-gene overlaps (exclusive mode) may unintentionally distort TE quantification where TEs intersect neighboring genes, and no quantitative guidance is given on when to opt out

## Evidence

- [results] MATES outperforms scTE and SoloTE in non-mammalian species with ARI comparisons on Arabidopsis and Drosophila: [evidence span withheld — anchor status: not_found]
- [readme] Non-mammalian references are not auto-generated: [evidence span withheld — anchor status: external_doc]
