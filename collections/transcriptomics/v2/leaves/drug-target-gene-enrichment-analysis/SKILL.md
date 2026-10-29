---
name: drug-target-gene-enrichment-analysis
description: Use when scRNA-seq count matrices from drug-sensitive vs drug-resistant
  myeloma samples (e.g., U266P vs U266VR) must be screened to rank combination candidates
  by subclonal enrichment of specific drug-target genes.
when_to_use_negative:
- Input is bulk RNA-seq or bulk pharmacogenomics data — this skill requires single-cell
  resolution to score per-subclone target-gene enrichment
- The candidate drug's target gene(s) are not known or not derivable from LINCS, so
  nothing can be mapped onto clusters
- The goal is in silico secDrug prediction itself (greedy set cover algorithm on the
  >70 HMCL pharmacogenomics panel), not single-cell screening of already-predicted
  combinations
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: R
  role: 'Language for the screening pipeline: Seurat preprocessing, t-SNE embedding,
    and running the CombinationDrugMyeloma enrichment functions'
  repo: github.com/Ujjal-Mukherjee/secDrug
- name: secDrug
  role: Source of the deposited HMCL scRNA-seq count matrices and LINCS-derived target-gene
    sets used for the screen
  repo: github.com/Ujjal-Mukherjee/secDrug
- name: CombinationDrugMyeloma
  role: R package whose AssignGroup(x, l) bins per-cluster target-gene expression
    and PropTest(p1, p2, n) performs between-cluster proportion testing for enrichment
    scoring
  repo: github.com/Ujjal-Mukherjee/secDrug
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41408-022-00636-2
    title: 'secDrug: a pipeline to discover novel drug combinations to kill drug-resistant
      multiple myeloma cells using a greedy set cover algorithm and single-cell multi-omics'
  source_content_hash: sha256:d25d84918a21ae23868d01bab0d24eb63d9ff0b63e3e52c72c0e39532361d223
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CombinationDrugMyeloma
  - R
  - secDrug
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# drug-target-gene-enrichment-analysis

## Summary

A single-cell RNA-seq screening skill that maps LINCS-derived drug-target genes (e.g., 17-AAG targets HSP90AA1/HSP90AB1 and FK866 target NAMPT) onto t-SNE-defined subclonal clusters of myeloma samples to prioritize secDrug combinations by per-cluster target-gene enrichment.

## When to use

Use when scRNA-seq count matrices from drug-sensitive vs drug-resistant myeloma samples (e.g., U266P vs U266VR) must be screened to rank combination candidates by subclonal enrichment of specific drug-target genes.

## When NOT to use

- Input is bulk RNA-seq or bulk pharmacogenomics data — this skill requires single-cell resolution to score per-subclone target-gene enrichment
- The candidate drug's target gene(s) are not known or not derivable from LINCS, so nothing can be mapped onto clusters
- The goal is in silico secDrug prediction itself (greedy set cover algorithm on the >70 HMCL pharmacogenomics panel), not single-cell screening of already-predicted combinations

## Inputs

- HMCL scRNA-seq count matrices (e.g., U266P vs U266VR drug-sensitive/resistant pairs) from the secDrug repository
- LINCS-derived drug-target gene lists (17-AAG: HSP90AA1, HSP90AB1; FK866: NAMPT)
- Seurat-processed per-sample cluster/t-SNE embeddings

## Outputs

- Per-sample t-SNE cluster target-gene enrichment map (annotated figure, reproducing Fig. 2)
- Per-subclone, per-sample target-gene enrichment scores exported as CSV
- Between-cluster proportion test results (PropTest) for target-gene enrichment comparisons

## How to apply

Load the deposited HMCL scRNA-seq count matrices from github.com/Ujjal-Mukherjee/secDrug into R, preprocess and cluster each sample with Seurat (QC filtering, normalization), and embed subclones with t-SNE. Map the LINCS-derived target genes — HSP90AA1/HSP90AB1 for 17-AAG and NAMPT for FK866 — onto the t-SNE-defined subclonal clusters of each sample. Use the CombinationDrugMyeloma package functions AssignGroup(x, l) to bin per-cluster target-gene expression into enrichment groups and PropTest(p1, p2, n) for between-cluster proportion testing to compute target-gene enrichment per subclone per sample. Export the per-sample t-SNE cluster target-gene enrichment map with enrichment scores as CSV and the annotated t-SNE map as a figure (reproducing Fig. 2). The screen is accepted if the majority of single-cell subclonal clusters show enrichment for the HSP90AA1/HSP90AB1 and NAMPT target genes.

## Related tools

- **R** (Language for the screening pipeline: Seurat preprocessing, t-SNE embedding, and running the CombinationDrugMyeloma enrichment functions) — github.com/Ujjal-Mukherjee/secDrug
- **secDrug** (Source of the deposited HMCL scRNA-seq count matrices and LINCS-derived target-gene sets used for the screen) — github.com/Ujjal-Mukherjee/secDrug
- **CombinationDrugMyeloma** (R package whose AssignGroup(x, l) bins per-cluster target-gene expression and PropTest(p1, p2, n) performs between-cluster proportion testing for enrichment scoring) — github.com/Ujjal-Mukherjee/secDrug

## Examples

```
AssignGroup(x, l); PropTest(p1, p2, n)
```

## Evaluation signals

- Majority of single-cell subclonal clusters in both drug-sensitive and drug-resistant samples show enrichment for HSP90AA1/HSP90AB1 and NAMPT, matching the Fig. 2 pattern
- Enrichment scores exported as CSV contain one entry per subclone per sample with no missing cluster annotations
- AssignGroup output bins every t-SNE cluster into a discrete target-gene expression group (high/low) without unassigned clusters
- PropTest comparisons between clusters yield interpretable proportions consistent with the visual t-SNE enrichment map

## Limitations

- Pipeline currently validated for two-drug combinations; model systems for testing more than two secDrugs are not yet developed
- Screening was demonstrated on a limited set of HMCL scRNA-seq pairs; testing in a wider panel of refractory and clonally derived acquired-resistant HMCLs is not yet done
- Predicted candidates (e.g., KIN001-102 and SB505124) lack pre-clinical/clinical xenograft or mouse-model validation

## Evidence

- [abstract] scRNAseq screening rationale: "we used single-cell RNA sequencing (scRNAseq) as a screening tool to predict top combination candidates based on the enrichment of target genes"
- [intro] subclonal target expression prioritization: "we introduced single-cell transcriptomics as a novel screening tool for prioritizing secDrug combinations based on the subclonal expression of the drug targets"
- [methods] CombinationDrugMyeloma exports: "Declared exports parsed from the repository (NAMESPACE / __init__ / entry points)."
- [discussion] single-cell multi-omics integration: "single-cell multi-omics (single-cell transcriptomics/scRNAseq and single-cell proteomics/CyTOF analysis)"
