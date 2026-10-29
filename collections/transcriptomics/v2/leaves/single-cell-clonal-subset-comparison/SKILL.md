---
name: single-cell-clonal-subset-comparison
description: Use when single cells have already been assigned to clones/subclones
  (e.g., mtDNA subclones clone1.MT0 vs clone1.MT1) and the goal is to test whether
  those subclones are biologically distinct via DE genes and Hallmark pathway enrichment.
when_to_use_negative:
- Cells have not yet been assigned to clones — run the upstream mtDNA clonality pipeline
  (cellsnp-lite/MQuad/vireoSNP) first
- The question is identifying informative mtDNA variants or reconstructing clones,
  not testing expression differences between already-defined subclones
- Nuclear SNV or CNV lineage labels are the only clonal markers and mtDNA subclone
  structure is not of interest — the clone predictor in the GLM would not represent
  the intended contrast
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: edgeR
  role: Fits the single-cell GLM (cellular detection rate, plate, mtDNA clone as predictors)
    and performs the two-sided quasi-likelihood F-test for the subclone contrast
- name: IHW
  role: Independent hypothesis weighting multiple-testing correction for both gene-level
    and gene-set-level p-values, using average gene expression as the independent
    covariate
- name: limma
  role: Provides the two-sided camera gene-set test run on edgeR log2-fold-changes
    against the 50 Hallmark sets
- name: Molecular Signatures Database (MsigDB)
  role: Source of the 50 Hallmark gene sets tested for enrichment between subclones
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-28845-0
    title: MQuad enables clonal substructure discovery using single cell mitochondrial
      variants
  source_content_hash: sha256:769887e46c6e7bd27d9bac41ef38ba0521268c4a550328a66eee6f80d6d4ecf9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IHW
  - Molecular Signatures Database (MsigDB)
  - edgeR
  - limma
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# single-cell-clonal-subset-comparison

## Summary

Establishes that clonally defined single-cell subpopulations (e.g., mtDNA-defined subclones from MQuad/vireoSNP) are transcriptionally and functionally distinct, by running an edgeR quasi-likelihood differential expression test between subclones followed by limma camera enrichment of MSigDB Hallmark gene sets on the resulting log2-fold-changes.

## When to use

Use when single cells have already been assigned to clones/subclones (e.g., mtDNA subclones clone1.MT0 vs clone1.MT1) and the goal is to test whether those subclones are biologically distinct via DE genes and Hallmark pathway enrichment.

## When NOT to use

- Cells have not yet been assigned to clones — run the upstream mtDNA clonality pipeline (cellsnp-lite/MQuad/vireoSNP) first
- The question is identifying informative mtDNA variants or reconstructing clones, not testing expression differences between already-defined subclones
- Nuclear SNV or CNV lineage labels are the only clonal markers and mtDNA subclone structure is not of interest — the clone predictor in the GLM would not represent the intended contrast

## Inputs

- Single-cell gene expression count matrix (Smart-seq2 scRNA-seq fibroblast dataset)
- Per-cell mtDNA clone/subclone assignments from the clonality pipeline (e.g., clone1.MT0 / clone1.MT1 from task_003)
- MSigDB Hallmark gene sets (50 sets)
- Per-cell covariates: cellular detection rate and plate/batch identity

## Outputs

- de_gene_table.csv — gene-level differential expression statistics (log2FC, p-value, IHW-adjusted FDR) for the subclone contrast, significant genes at FDR < 0.1
- gene_set_enrichment_table.csv — camera test results for the 50 Hallmark gene sets with IHW-adjusted FDR, significant sets flagged at FDR < 0.05

## How to apply

Restrict the single-cell gene expression counts (Smart-seq2 fibroblast data) to cells within the clone of interest, splitting them by mitochondrial subclone label (e.g., clone1.MT0 vs clone1.MT1). Fit an edgeR generalized linear model with cellular detection rate, plate, and the assigned mitochondrial clone as predictor variables, and test the two subclones with the two-sided quasi-likelihood F-test. Apply multiple-testing correction with independent hypothesis weighting (IHW) using average gene expression as the independent covariate, retaining DE genes at FDR < 0.1 (the paper reports 847 such genes). Then run the two-sided camera test from limma on the 50 MSigDB Hallmark gene sets, using the edgeR log2-fold-changes as input, adjusting gene set p-values with IHW the same way and calling enrichments significant at FDR < 0.05. Rationale: the clone labels come from an mtDNA-based pipeline (cellsnp-lite/MQuad/vireoSNP), so confounders like detection rate and batch (plate) must be modeled explicitly to attribute expression differences to subclone identity rather than technical effects.

## Related tools

- **edgeR** (Fits the single-cell GLM (cellular detection rate, plate, mtDNA clone as predictors) and performs the two-sided quasi-likelihood F-test for the subclone contrast)
- **IHW** (Independent hypothesis weighting multiple-testing correction for both gene-level and gene-set-level p-values, using average gene expression as the independent covariate)
- **limma** (Provides the two-sided camera gene-set test run on edgeR log2-fold-changes against the 50 Hallmark sets)
- **Molecular Signatures Database (MsigDB)** (Source of the 50 Hallmark gene sets tested for enrichment between subclones)

## Evaluation signals

- de_gene_table.csv contains a gene-level statistics column with FDR values derived from IHW correction
- Count of significant DE genes at FDR < 0.1 is consistent with the paper's reported 847 genes for the clone1.MT0 vs clone1.MT1 contrast
- gene_set_enrichment_table.csv flags MYC targets, E2F targets, G2M checkpoint, and Reactive Oxygen Species (ROS) Pathway as significant at FDR < 0.05
- The GLM design includes cellular detection rate and plate alongside the mitochondrial clone predictor (technical confounders controlled)
- Camera test is two-sided and run on edgeR log2-fold-changes, with gene-set p-values adjusted by the same IHW scheme as the gene-level test

## Limitations

- Depends on the accuracy of upstream mtDNA clone assignments; noisy or misassigned clone labels (uninformative/noisy mtDNA SNVs confounding clonal inference) will invalidate the contrast
- Gene-level significance uses a relatively permissive FDR < 0.1 threshold; results should be interpreted alongside the stricter gene-set FDR < 0.05 enrichments
- Very small subclone sizes leave little power for the quasi-likelihood F-test and camera, and plate/detection-rate covariates may absorb genuine biological signal if confounded with subclone

## Evidence

- [full_text] edgeR GLM design with clone predictor: [evidence span withheld — anchor status: not_found]
- [full_text] IHW correction with expression covariate: [evidence span withheld — anchor status: not_found]
- [full_text] camera on Hallmark sets: [evidence span withheld — anchor status: not_found]
- [discussion] Hallmark gene set source: "Using 50 Hallmark gene sets from Molecular Signatures Database (MsigDB)43"
