---
name: lincs-target-gene-mapping
description: Use when scRNA-seq clusters from drug-sensitive and drug-resistant myeloma
  samples (e.g., U266P vs U266VR) need per-subclone drug-target gene enrichment scoring
  to rank combination candidates such as 17-AAG + FK866.
when_to_use_negative:
- Input is bulk RNA-seq or pharmacogenomics data rather than single-cell clusters
  — the greedy set cover secDrug prediction pipeline handles that, not this target-gene
  mapping screen
- The drug's target genes are unknown or not derivable from LINCS — no target gene
  list means no enrichment signal can be mapped
- Only a single homogeneous cell population without subclonal structure is available,
  so between-cluster proportion testing (PropTest) is meaningless
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: secDrug
  role: Source repository providing the deposited HMCL scRNA-seq count matrices and
    pipeline code used for the single-cell target-gene enrichment screen
  repo: github.com/Ujjal-Mukherjee/secDrug
- name: CombinationDrugMyeloma
  role: R package exporting AssignGroup (bin per-cluster target-gene expression) and
    PropTest (between-cluster proportion testing) used to compute enrichment per subclone
- name: R
  role: Environment in which Seurat preprocessing/clustering, t-SNE embedding, and
    the CombinationDrugMyeloma enrichment functions are executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41408-022-00636-2
    title: 'secDrug: a pipeline to discover novel drug combinations to kill drug-resistant
      multiple myeloma cells using a greedy set cover algorithm and single-cell multi-omics'
  source_content_hash: sha256:40f2790e294db485a706b64dab8e4f02140cc7573d6dd708bf84ce9e260b0f7c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CombinationDrugMyeloma
  - R
  - secDrug
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# lincs-target-gene-mapping

## Summary

Maps LINCS-derived drug-target genes (17-AAG targets HSP90AA1/HSP90AB1 and FK866 target NAMPT) onto t-SNE-defined single-cell subclonal clusters to score per-cluster target-gene enrichment, using scRNA-seq as a screening tool to prioritize secDrug combination candidates in drug-resistant myeloma.

## When to use

Use when scRNA-seq clusters from drug-sensitive and drug-resistant myeloma samples (e.g., U266P vs U266VR) need per-subclone drug-target gene enrichment scoring to rank combination candidates such as 17-AAG + FK866.

## When NOT to use

- Input is bulk RNA-seq or pharmacogenomics data rather than single-cell clusters — the greedy set cover secDrug prediction pipeline handles that, not this target-gene mapping screen
- The drug's target genes are unknown or not derivable from LINCS — no target gene list means no enrichment signal can be mapped
- Only a single homogeneous cell population without subclonal structure is available, so between-cluster proportion testing (PropTest) is meaningless

## Inputs

- Deposited HMCL scRNA-seq count matrices (e.g., U266P vs U266VR drug-sensitive/resistant pairs)
- LINCS-derived 17-AAG target gene list (HSP90AA1, HSP90AB1)
- LINCS-derived FK866 target gene (NAMPT)

## Outputs

- Per-sample t-SNE cluster target-gene enrichment map (annotated figure reproducing Fig. 2)
- Per-cluster target-gene enrichment scores exported as CSV
- Between-cluster proportion test results (PropTest) for target-gene enrichment

## How to apply

Load the deposited HMCL scRNA-seq count matrices (e.g., U266P vs U266VR pairs) from github.com/Ujjal-Mukherjee/secDrug into R, then preprocess and cluster each sample with Seurat (QC filtering, normalization) and embed subclones with t-SNE dimensionality reduction. Map the LINCS-derived 17-AAG target genes (HSP90AA1, HSP90AB1) and the FK866 target gene NAMPT onto the t-SNE-defined subclonal clusters of each sample. Use the CombinationDrugMyeloma package functions AssignGroup to bin per-cluster target-gene expression into enriched/non-enriched groups and PropTest for between-cluster proportion testing to compute target-gene enrichment per subclone per sample. Export the per-sample t-SNE cluster target-gene enrichment scores as CSV and the annotated t-SNE map as a figure reproducing Fig. 2. The rationale is that subclonal expression of drug targets indicates which resistant clones a secDrug combination can kill, allowing scRNA-seq itself to act as the screening modality.

## Related tools

- **secDrug** (Source repository providing the deposited HMCL scRNA-seq count matrices and pipeline code used for the single-cell target-gene enrichment screen) — github.com/Ujjal-Mukherjee/secDrug
- **CombinationDrugMyeloma** (R package exporting AssignGroup (bin per-cluster target-gene expression) and PropTest (between-cluster proportion testing) used to compute enrichment per subclone)
- **R** (Environment in which Seurat preprocessing/clustering, t-SNE embedding, and the CombinationDrugMyeloma enrichment functions are executed)

## Examples

```
AssignGroup(cluster_expr, target_genes); PropTest(p1, p2, n)
```

## Evaluation signals

- The majority of single-cell subclonal clusters show enrichment for HSP90AA1/HSP90AB1 and NAMPT target genes, matching the pattern reported in Fig. 2
- Enrichment scores CSV contains one enrichment value per subclonal cluster per sample for all mapped target genes
- PropTest between-cluster comparisons yield statistically interpretable proportions (p1, p2 tested against cluster sizes n) with no empty clusters
- Annotated t-SNE map clearly labels which clusters are target-gene enriched for 17-AAG and FK866 targets in both drug-sensitive and drug-resistant samples
- The 17-AAG + FK866 combination is ranked as effective by the screen, consistent with the paper's proof-of-principle finding

## Limitations

- The screen predicts but does not itself validate — 17-AAG + FK866 required extensive in vitro, ex vivo, and functional validation in refractory/resistant myeloma models
- Pre-clinical and clinical validation in xenograft or mouse models has not yet been performed for novel secDrugs such as KIN001-102 and SB505124
- Findings are grounded in the HMCL panel and specific drug pairs (17-AAG, FK866); target-gene lists must exist in LINCS for other combinations to be screenable

## Evidence

- [abstract] scRNAseq target-gene enrichment screen: "we used single-cell RNA sequencing (scRNAseq) as a screening tool to predict top combination candidates based on the enrichment of target genes"
- [intro] subclonal target expression rationale: "we introduced single-cell transcriptomics as a novel screening tool for prioritizing secDrug combinations based on the subclonal expression of the drug targets"
- [full_text] mapping step and target genes: [evidence span withheld — anchor status: not_found]
- [methods] package functions: "Declared exports parsed from the repository (NAMESPACE / __init__ / entry points)."
