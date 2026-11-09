---
name: hmcl-drug-resistance-expression-profiling
description: Use when scRNA-seq count matrices from drug-sensitive vs drug-resistant
  HMCL pairs (e.g., U266P vs U266VR) must be screened for subclonal enrichment of
  drug-target genes to prioritize combination therapy candidates.
when_to_use_negative:
- The goal is in silico prediction of secDrug candidates from pharmacogenomics data
  via the greedy set cover algorithm, not single-cell target-gene screening of a predicted
  combination.
- Input is bulk RNA-seq transcriptome data rather than single-cell count matrices;
  subclonal cluster-level enrichment cannot be computed.
- Validation requires functional/ex vivo assays (CRISPR editing, CyTOF on primary
  bone marrow cells) — this skill only screens candidate combinations, it does not
  confirm efficacy.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Software environment in which Seurat preprocessing/t-SNE clustering and the
    secDrug/CombinationDrugMyeloma functions (AssignGroup, PropTest) are executed
  repo: github.com/Ujjal-Mukherjee/secDrug
- name: secDrug
  role: Computational pipeline providing the deposited HMCL scRNA-seq count matrices
    and the screening context for prioritizing secDrug combinations
  repo: github.com/Ujjal-Mukherjee/secDrug
- name: CombinationDrugMyeloma
  role: Package supplying AssignGroup(x, l) for per-cluster target-gene expression
    binning and PropTest(p1, p2, n) for between-cluster proportion testing
  repo: github.com/Ujjal-Mukherjee/secDrug
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41408-022-00636-2
    title: 'secDrug: a pipeline to discover novel drug combinations to kill drug-resistant
      multiple myeloma cells using a greedy set cover algorithm and single-cell multi-omics'
  source_content_hash: sha256:fe9c5e7d277f5af16dd6521aebeef6da999a7f9f191e0e2c3dc37e82a439107f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CombinationDrugMyeloma
  - R
  - secDrug
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# hmcl-drug-resistance-expression-profiling

## Summary

A single-cell RNA-seq screening skill that profiles drug-target gene enrichment across subclonal clusters of human myeloma cell lines (HMCLs) to prioritize secDrug combination candidates such as 17-AAG + FK866. It translates target-gene expression (HSP90AA1/HSP90AB1 for 17-AAG, NAMPT for FK866) into ranked combination evidence.

## When to use

Use when scRNA-seq count matrices from drug-sensitive vs drug-resistant HMCL pairs (e.g., U266P vs U266VR) must be screened for subclonal enrichment of drug-target genes to prioritize combination therapy candidates.

## When NOT to use

- The goal is in silico prediction of secDrug candidates from pharmacogenomics data via the greedy set cover algorithm, not single-cell target-gene screening of a predicted combination.
- Input is bulk RNA-seq transcriptome data rather than single-cell count matrices; subclonal cluster-level enrichment cannot be computed.
- Validation requires functional/ex vivo assays (CRISPR editing, CyTOF on primary bone marrow cells) — this skill only screens candidate combinations, it does not confirm efficacy.

## Inputs

- Deposited HMCL scRNA-seq count matrices (e.g., U266P vs U266VR drug-sensitive/resistant pairs) from github.com/Ujjal-Mukherjee/secDrug
- LINCS-derived drug target gene lists: 17-AAG targets (HSP90AA1, HSP90AB1) and FK866 target (NAMPT)
- Seurat-formatted single-cell expression objects after QC filtering and normalization

## Outputs

- Per-sample t-SNE cluster map annotated with target-gene enrichment (reproducing Fig. 2)
- Per-cluster, per-sample target-gene enrichment score table exported as CSV
- Between-cluster proportion test results from PropTest(p1, p2, n)
- Grouped per-cluster target-gene expression bins from AssignGroup(x, l)

## How to apply

Load the deposited HMCL scRNA-seq count matrices from github.com/Ujjal-Mukherjee/secDrug into R, then preprocess and cluster each sample with Seurat (QC filtering, normalization) and embed subclones with t-SNE. Map the LINCS-derived 17-AAG target genes (HSP90AA1, HSP90AB1) and the FK866 target gene (NAMPT) onto the t-SNE-defined subclonal clusters of each sample. Use the CombinationDrugMyeloma package functions — AssignGroup(x, l) to bin per-cluster target-gene expression and PropTest(p1, p2, n) for between-cluster proportion testing — to compute target-gene enrichment per subclone per sample. Export per-cluster enrichment scores as CSV and an annotated t-SNE enrichment map reproducing Fig. 2. The screen is validated when the majority of single-cell subclonal clusters show enrichment for the HSP90AA1/HSP90AB1 and NAMPT target genes.

## Related tools

- **R** (Software environment in which Seurat preprocessing/t-SNE clustering and the secDrug/CombinationDrugMyeloma functions (AssignGroup, PropTest) are executed) — github.com/Ujjal-Mukherjee/secDrug
- **secDrug** (Computational pipeline providing the deposited HMCL scRNA-seq count matrices and the screening context for prioritizing secDrug combinations) — github.com/Ujjal-Mukherjee/secDrug
- **CombinationDrugMyeloma** (Package supplying AssignGroup(x, l) for per-cluster target-gene expression binning and PropTest(p1, p2, n) for between-cluster proportion testing) — github.com/Ujjal-Mukherjee/secDrug

## Examples

```
groups <- AssignGroup(x, target_genes); prop_res <- PropTest(p1, p2, n)
```

## Evaluation signals

- The majority of single-cell subclonal clusters show enrichment for HSP90AA1/HSP90AB1 and NAMPT, matching the enrichment pattern reported in Fig. 2.
- Enrichment scores export as a CSV keyed by sample and t-SNE cluster with per-cluster bins from AssignGroup.
- Between-cluster differences in target-gene-expressing cell proportions are confirmed by PropTest(p1, p2, n).
- t-SNE clusters derive from Seurat QC filtering and normalization of each sample (e.g., U266P vs U266VR), not from unprocessed counts.
- Target gene lists match the LINCS-derived 17-AAG (HSP90AA1, HSP90AB1) and FK866 (NAMPT) targets rather than arbitrary gene sets.

## Limitations

- Pre-clinical and clinical validation in xenograft or mouse models has not yet been performed for novel secDrugs such as KIN001-102 and SB505124.
- Testing of predicted secDrugs in a wider panel of HMCLs representing refractory and clonally derived acquired resistant cell lines has not yet been done.
- The pipeline is currently limited to two-drug combinations; model systems for testing more than two secDrugs are not yet developed.

## Evidence

- [abstract] scRNA-seq screening showed that the majority of single-cell clusters in drug-sensitive and drug-resistant myeloma express high levels of the 17-AAG target genes HSP90AA1/HSP90AB1 and the FK866 target gene NAMPT: "we used single-cell RNA sequencing (scRNAseq) as a screening tool to predict top combination candidates based on the enrichment of target genes"
- [methods] The screen maps target genes onto subclonal clusters and tests proportions between clusters.: [evidence span withheld — anchor status: not_found]
- [discussion] The screening tool sits within an integrated single-cell multi-omics pipeline.: "single-cell multi-omics (single-cell transcriptomics/scRNAseq and single-cell proteomics/CyTOF analysis)"
- [abstract] The output of the pipeline is ranked combination candidates with confidence scores.: "outputs a list of the top drug combination candidates with rank and confidence scores"
