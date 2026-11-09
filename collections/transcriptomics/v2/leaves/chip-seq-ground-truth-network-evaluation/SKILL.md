---
name: chip-seq-ground-truth-network-evaluation
description: Use when a ranked TF–gene edge list has been produced from scRNA-seq
  data (e.g. Tabula Muris tissue data) and needs quantitative validation against ChIP-Atlas-derived
  ground-truth GRNs per cell type using AUROC and EPR.
when_to_use_negative:
- No ChIP-seq ground truth exists for the tissue or cell type being evaluated (only
  tissues with adequate ChIP-Atlas data are benchmarkable)
- Available ChIP-seq experiments have fewer than 50 peaks or come from non-physiological
  conditions (gene knockouts, adeno-associated virus treatment)
- The target interactions of interest live in large protein complexes, which are missed
  both by the current model and in the ground-truth networks
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scGATE
  role: Produces the BF-ranked directed TF–gene network (scGATE_edge mode) that is
    evaluated against the ChIP-Atlas ground truth
  repo: github.com/CompBioIPM/scGATE
- name: ChIP-Atlas
  role: Source of mouse TF ChIP-seq experiments from which the per-cell-type ground-truth
    GRNs are generated
- name: R 4.1.3
  role: Runtime for scGATE 0.1.0 and the metric computation; tested on Windows and
    Linux
- name: arrow
  role: Reads the .parquet base-GRN files (e.g. Cusanovich2018_Spleen_peak_base_GRN_dataframe.parquet)
    that feed the inferred network being evaluated
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae180
    title: Single-cell multi-omics analysis identifies context-specific gene regulatory
      gates and mechanisms
  source_content_hash: sha256:a92a695c0d8272cd02002f922378f9942331ac8a72d0548624bb0fceffe6606a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/CompBioIPM/scGATE
  tools:
  - ChIP-Atlas
  - R 4.1.3
  - VGAM
  - arrow
  - doParallel
  - doSNOW
  - foreach
  - scGATE
  - truncnorm
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# chip-seq-ground-truth-network-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark an inferred (e.g. scGATE) TF–gene network against cell-type-specific ground-truth GRNs built from mouse ChIP-seq data in the ChIP-Atlas database, ranking predicted edges and computing AUROC and EPR per dataset. This is the evaluation half of the Tabula Muris scGATE workflow.

## When to use

Use when a ranked TF–gene edge list has been produced from scRNA-seq data (e.g. Tabula Muris tissue data) and needs quantitative validation against ChIP-Atlas-derived ground-truth GRNs per cell type using AUROC and EPR.

## When NOT to use

- No ChIP-seq ground truth exists for the tissue or cell type being evaluated (only tissues with adequate ChIP-Atlas data are benchmarkable)
- Available ChIP-seq experiments have fewer than 50 peaks or come from non-physiological conditions (gene knockouts, adeno-associated virus treatment)
- The target interactions of interest live in large protein complexes, which are missed both by the current model and in the ground-truth networks

## Inputs

- BF-ranked directed TF–gene edge list from scGATE_edge() (e.g. head: Ctcf→Rps19, Batf→Rps19, Stat5b→Rplp0)
- Cell-type-specific ground-truth GRNs derived from mouse TF ChIP-seq data in ChIP-Atlas
- Tabula Muris tissue scRNA-seq datasets (five tissues: Spleen, Lung, Liver, Kidney, Heart; GSE109774) with matching Cusanovich scATAC-seq data (GSE111586)

## Outputs

- Per-cell-type network quality metrics: AUROC and EPR
- Average number of regulating TFs per target gene (reference value ~3.39 for Spleen-10X_P4_7)
- Method-vs-tool comparison across the 8 Tabula Muris scRNA-seq benchmark datasets

## How to apply

After running scGATE_edge() on preprocessed scRNA-seq data (e.g. Tabula_Muris2018_Spleen-10X_P4_7 with Cusanovich2018 Spleen scATAC-seq base GRN), evaluate the BF-ranked directed TF–gene edges against ground-truth GRNs generated from mouse TF ChIP-seq data in ChIP-Atlas, matching each ranked network to the ground truth for the corresponding cell type. Build the ground truth from ChIP-Atlas ChIP-seq experiments, excluding datasets with fewer than 50 peaks and non-physiological conditions such as gene knockouts or adeno-associated virus treatment. Compute network quality metrics — AUROC and EPR — per cell type, plus the average number of regulating TFs per target gene, and compare the inferred method against other benchmarked network inference tools. The reference Tabula Muris Spleen-10X_P4_7 run reports ~3.39 average regulators per target gene and ranks Ctcf→Rps19 (BF ≈ 2002.419), Batf→Rps19 (BF ≈ 2001.388), and Stat5b→Rplp0 (BF ≈ 1840.046) at the head of the ranked edge list.

## Related tools

- **scGATE** (Produces the BF-ranked directed TF–gene network (scGATE_edge mode) that is evaluated against the ChIP-Atlas ground truth) — github.com/CompBioIPM/scGATE
- **ChIP-Atlas** (Source of mouse TF ChIP-seq experiments from which the per-cell-type ground-truth GRNs are generated)
- **R 4.1.3** (Runtime for scGATE 0.1.0 and the metric computation; tested on Windows and Linux)
- **arrow** (Reads the .parquet base-GRN files (e.g. Cusanovich2018_Spleen_peak_base_GRN_dataframe.parquet) that feed the inferred network being evaluated)

## Examples

```
edges <- scGATE_edge(data = data, base_GRN = candidate_tf_target, h_act = 7)
```

## Evaluation signals

- Head of ranked_edge_list matches the reference: Ctcf→Rps19 with BF ≈ 2002.419, Batf→Rps19 with BF ≈ 2001.388, Stat5b→Rplp0 with BF ≈ 1840.046
- Average number of regulating TFs per target gene reproduces the reported ~3.39
- AUROC and EPR are computed per cell type against the corresponding ChIP-Atlas ground-truth GRN, and scGATE should win in 7 of 8 Tabula Muris scRNA-seq datasets
- ChIP-seq datasets contributing to the ground truth satisfy the ≥50-peak and physiological-condition filters
- Input count matrix was preprocessed with library size normalization and rescaled into the (0,1) interval before inference

## Limitations

- Only ChIP-seq data with ≥50 peaks obtained under physiological conditions (no gene knockouts, no adeno-associated virus treatment) can contribute to ground truth, restricting coverage
- Large protein complex interactions are missed both in the scGATE model and in the ground-truth networks, biasing evaluation
- Evaluation depends on a pre-selected candidate TF list (e.g. Tabula_Muris2018_Spleen-10X_P4_7_tf_lists.txt); scGATE itself only handles <200 genes with refined candidate TFs

## Evidence
