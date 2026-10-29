---
name: differential-gene-expression-analysis
description: Use when a gene-by-cell Smart-seq2 (or comparable) scRNA-seq count matrix
  is paired with per-cell mtDNA clone/subclone labels and the goal is to test if two
  subclones are transcriptionally distinct and which Hallmark pathways separate them.
when_to_use_negative:
- The goal is to identify informative mtDNA variants or assign cells to clones — use
  the cellSNP/cellsnp-lite + MQuad + vireoSNP clonality pipeline instead; this skill
  consumes its clone assignments downstream.
- Input is raw FASTQ/BAM reads rather than a gene-by-cell count matrix with clone
  labels — counts must be produced first (e.g. STAR-aligned Smart-seq2 processing)
  before this skill applies.
- The two subclones share mtDNA-defined labels because the upstream variant calling
  failed — a null or spurious DE result would reflect clone misassignment, not biology.
edam_operation: http://edamontology.org/operation_3565
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: edgeR
  role: 'Fits the single-cell GLM (predictors: cellular detection rate, plate, mitochondrial
    clone) and performs the two-sided quasi-likelihood F-test of clone1.MT0 vs clone1.MT1'
- name: IHW
  role: Multiple-testing correction of both gene-level p-values and gene set p-values,
    using average gene expression as the independent covariate
- name: limma (camera)
  role: Two-sided competitive gene set test on 50 MSigDB Hallmark gene sets using
    the edgeR log2-fold-changes as input
- name: MSigDB Hallmark gene sets
  role: Reference collection of 50 gene sets against which enrichment is tested
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-28845-0
    title: MQuad enables clonal substructure discovery using single cell mitochondrial
      variants
  source_content_hash: sha256:4a320119d821387e59d2aafdbb12832b9e3fed9ce990275501c02d6e70be4593
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IHW
  - MSigDB Hallmark gene sets
  - Molecular Signatures Database (MsigDB)
  - edgeR
  - limma
  - limma (camera)
  edam_operation: http://edamontology.org/operation_3565
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# differential-gene-expression-analysis

## Summary

Tests whether two mtDNA-defined subclones (e.g. clone1.MT0 vs clone1.MT1 within nuclear clone 1 of the fibroblast Smart-seq2 dataset) are transcriptionally distinct, using an edgeR quasi-likelihood GLM with IHW correction, and annotates the biology via limma camera on 50 MSigDB Hallmark gene sets.

## When to use

Use when a gene-by-cell Smart-seq2 (or comparable) scRNA-seq count matrix is paired with per-cell mtDNA clone/subclone labels and the goal is to test if two subclones are transcriptionally distinct and which Hallmark pathways separate them.

## When NOT to use

- The goal is to identify informative mtDNA variants or assign cells to clones — use the cellSNP/cellsnp-lite + MQuad + vireoSNP clonality pipeline instead; this skill consumes its clone assignments downstream.
- Input is raw FASTQ/BAM reads rather than a gene-by-cell count matrix with clone labels — counts must be produced first (e.g. STAR-aligned Smart-seq2 processing) before this skill applies.
- The two subclones share mtDNA-defined labels because the upstream variant calling failed — a null or spurious DE result would reflect clone misassignment, not biology.

## Inputs

- Gene-by-cell Smart-seq2 scRNA-seq counts matrix (fibroblast dataset) with per-cell metadata
- Per-cell mtDNA clone/subclone assignments (clone1.MT0, clone1.MT1) from the MQuad/vireoSNP clonality pipeline
- Cell-level covariates: cellular detection rate, plate/batch identifier
- 50 MSigDB Hallmark gene sets (versioned from the Molecular Signatures Database)

## Outputs

- de_gene_table.csv — per-gene edgeR quasi-likelihood F-test statistics including an FDR column, filtered to FDR < 0.1
- gene_set_enrichment_table.csv — camera test results on 50 Hallmark sets with IHW-adjusted p-values, flagged at FDR < 0.05

## How to apply

Restrict the fibroblast Smart-seq2 single-cell gene expression counts (with mtDNA clone assignments from the cellSNP-MQuad-vireoSNP pipeline) to cells within the subclones being compared, e.g. clone1.MT0 and clone1.MT1 inside nuclear clone 1. Fit an edgeR generalized linear model with cellular detection rate, plate, and the assigned mitochondrial clone as predictor variables, and test clone1.MT0 vs clone1.MT1 with the two-sided quasi-likelihood F-test — including detection rate and plate is essential because Smart-seq2 data suffer high dropout and plate-level batch effects. Apply independent hypothesis weighting (IHW) using average gene expression as the independent covariate and retain DE genes at FDR < 0.1 (the paper reports 847 such genes) in de_gene_table.csv. Then run the two-sided camera test from limma on the 50 MSigDB Hallmark gene sets, using the edgeR log2-fold-changes as input, IHW-adjusting gene set p-values and calling enrichments significant at FDR < 0.05 in gene_set_enrichment_table.csv.

## Related tools

- **edgeR** (Fits the single-cell GLM (predictors: cellular detection rate, plate, mitochondrial clone) and performs the two-sided quasi-likelihood F-test of clone1.MT0 vs clone1.MT1)
- **IHW** (Multiple-testing correction of both gene-level p-values and gene set p-values, using average gene expression as the independent covariate)
- **limma (camera)** (Two-sided competitive gene set test on 50 MSigDB Hallmark gene sets using the edgeR log2-fold-changes as input)
- **MSigDB Hallmark gene sets** (Reference collection of 50 gene sets against which enrichment is tested)

## Examples

```
Rscript -e 'library(edgeR); y <- DGEList(counts=counts_mt0mt1); fit <- glmQLFit(y, design); qlf <- glmQLFTest(fit, contrast=mt0_vs_mt1); de <- topTags(qlf, n=Inf)'
```

## Evaluation signals

- de_gene_table.csv contains a gene-level statistics column with FDR values, and the number of genes at FDR < 0.1 matches the paper's 847 DE genes.
- All four expected Hallmark sets — MYC targets, E2F targets, G2M checkpoint, and Reactive Oxygen Species (ROS) Pathway — are flagged significant at FDR < 0.05 in gene_set_enrichment_table.csv.
- The edgeR design matrix explicitly includes cellular detection rate, plate, and assigned mitochondrial clone as predictors before the subclone contrast is tested.
- IHW was applied with average gene expression as the independent covariate; adjusted-FDR columns are present in both output tables.
- Gene set p-values come from a two-sided camera test computed on edgeR log2-fold-changes (not raw counts), with IHW-adjusted values reported.

## Limitations

- Results are conditional on the upstream mtDNA clone assignments; Smart-seq2 nuclear SNV calling is hampered by high dropout and few somatic variants, so subclone labels may be noisy.
- Smart-seq2 dropout limits power for low-expression genes; cellular detection rate must be modeled to avoid confounding clone differences with overall RNA content.
- The reported counts (847 DE genes at FDR < 0.1; four Hallmark sets at FDR < 0.05) are specific to this fibroblast dataset and threshold; changing FDR cutoffs or subclone pairs changes the output.

## Evidence

- [discussion] The scRNA-seq data (Kim and fibroblast datasets; both Smart-seq2) was preprocessed: "The scRNA-seq data (Kim and fibroblast datasets; both Smart-seq2) was preprocessed"
- [discussion] Using 50 Hallmark gene sets from Molecular Signatures Database (MsigDB)43: "Using 50 Hallmark gene sets from Molecular Signatures Database (MsigDB)43"
- [intro] partly due to the relatively small number of somatic variants in the large nuclear genome11 and a high dropout rate12: "partly due to the relatively small number of somatic variants in the large nuclear genome11 and a high dropout rate12"
- [readme] Please add `--UMItag None` if you bam file does not have UMIs, e.g., smart-seq and bulk RNA-seq.: [evidence span withheld — anchor status: not_found]
