---
name: gene-set-enrichment-analysis
description: Use when you have ranked gene-level differential expression statistics
  (e.g., edgeR log2-fold-changes between two cell clones or conditions) and want to
  know which MSigDB Hallmark pathways are enriched at FDR < 0.05.
when_to_use_negative:
- Input is raw counts without a fitted contrast — camera needs ranked gene-level statistics
  (e.g., edgeR log2-fold-changes) between defined groups, not an unlabeled expression
  matrix.
- The goal is calling or genotyping clonal mtDNA variants (MQuad/cellSNP/vireoSNP
  pipeline), not interpreting downstream pathway biology of an existing DE result.
- You need genome-wide pathway discovery beyond the 50 Hallmark sets — this skill
  is scoped to the fixed MSigDB Hallmark collection used in the paper.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
tools:
- name: edgeR
  role: Fits the GLM and two-sided quasi-likelihood F-test producing the gene-level
    log2-fold-changes that feed the camera test
- name: limma
  role: Provides the two-sided camera test for competitive gene-set enrichment accounting
    for inter-gene correlation
- name: IHW
  role: Multiple-testing correction of both gene-level and gene-set p-values, using
    average gene expression as the independent covariate
- name: Molecular Signatures Database (MsigDB)
  role: Source of the 50 Hallmark gene sets tested by camera
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-28845-0
    title: MQuad enables clonal substructure discovery using single cell mitochondrial
      variants
  source_content_hash: sha256:666e073518c927232c39fd7c523ce647257f3eea6c59a95cc201f54cbbb34e2c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IHW
  - Molecular Signatures Database (MsigDB)
  - edgeR
  - limma
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# gene-set-enrichment-analysis

## Summary

Test curated gene sets for enrichment among differential-expression statistics using the limma camera method, with IHW-based multiple-testing correction. In the MQuad fibroblast case this identifies Hallmark pathways (MYC targets, E2F targets, G2M checkpoint, ROS) distinguishing mtDNA-defined subclones MT0 vs MT1.

## When to use

Use when you have ranked gene-level differential expression statistics (e.g., edgeR log2-fold-changes between two cell clones or conditions) and want to know which MSigDB Hallmark pathways are enriched at FDR < 0.05.

## When NOT to use

- Input is raw counts without a fitted contrast — camera needs ranked gene-level statistics (e.g., edgeR log2-fold-changes) between defined groups, not an unlabeled expression matrix.
- The goal is calling or genotyping clonal mtDNA variants (MQuad/cellSNP/vireoSNP pipeline), not interpreting downstream pathway biology of an existing DE result.
- You need genome-wide pathway discovery beyond the 50 Hallmark sets — this skill is scoped to the fixed MSigDB Hallmark collection used in the paper.

## Inputs

- Single-cell gene expression counts matrix (Smart-seq2 fibroblast dataset) with per-cell clone assignments (e.g., clone1.MT0 vs clone1.MT1 from task_003 mtDNA clone calls)
- Cell-level covariates: cellular detection rate, plate/batch, and clone label
- 50 MSigDB Hallmark gene sets (gene-to-set membership index)

## Outputs

- de_gene_table.csv: per-gene edgeR GLM statistics with IHW-adjusted FDR values (DE genes at FDR < 0.1; expected ~847 genes)
- gene_set_enrichment_table.csv: per-gene-set camera statistics with IHW-adjusted FDR, significant sets flagged at FDR < 0.05

## How to apply

First fit an edgeR GLM on the single-cell (Smart-seq2) gene expression counts with cellular detection rate, plate, and clone assignment as covariates, testing the two subclones (e.g., clone1.MT0 vs clone1.MT1) with the two-sided quasi-likelihood F-test. Correct gene-level p-values with independent hypothesis weighting (IHW) using average gene expression as the independent covariate, and retain DE genes at FDR < 0.1 (the paper reports 847 such genes for the fibroblast MT0/MT1 comparison). Then run the two-sided camera test from limma on the 50 MSigDB Hallmark gene sets, supplying the edgeR log2-fold-changes as input statistics and accounting for inter-gene correlation. Adjust gene-set p-values with IHW as above and report enrichments at FDR < 0.05 in gene_set_enrichment_table.csv.

## Related tools

- **edgeR** (Fits the GLM and two-sided quasi-likelihood F-test producing the gene-level log2-fold-changes that feed the camera test)
- **limma** (Provides the two-sided camera test for competitive gene-set enrichment accounting for inter-gene correlation)
- **IHW** (Multiple-testing correction of both gene-level and gene-set p-values, using average gene expression as the independent covariate)
- **Molecular Signatures Database (MsigDB)** (Source of the 50 Hallmark gene sets tested by camera)

## Examples

```
R: gs <- camera(y=logFC_matrix, index=hallmarks_ids, design=design, contrast=mt0_vs_mt1); gs$FDR <- p.adjust(ihw(gs$PValue, ngenes, avg_expr)$qvalues, method='BH')
```

## Evaluation signals

- de_gene_table.csv contains a gene-level statistics column with FDR values, and the count of DE genes at FDR < 0.1 is consistent with the paper's reported 847 genes.
- gene_set_enrichment_table.csv flags exactly the expected sets as significant at FDR < 0.05: MYC targets, E2F targets, G2M checkpoint, and Reactive Oxygen Species (ROS) Pathway.
- Gene-set p-values are adjusted with IHW using average expression as the covariate, mirroring the gene-level correction scheme.
- The camera test was run two-sided on all 50 MSigDB Hallmark gene sets, not a subset.

## Limitations

- Only 50 MSigDB Hallmark gene sets are tested, so rare or non-Hallmark pathways will never be detected.
- Gene-level significance uses a lenient FDR < 0.1 threshold, which the paper chose for single-cell DE; results are sensitive to this cutoff and to the IHW covariate choice.
- Enrichment depends on the upstream edgeR model specification (detection rate, plate, clone as predictors); mis-specified covariates will propagate into the gene-set rankings.

## Evidence

- [methods] Run the two-sided camera test from limma on 50 MSigDB Hallmark gene sets using the edgeR log2-fold-changes as input.: [evidence span withheld — anchor status: not_found]
- [discussion] Using 50 Hallmark gene sets from Molecular Signatures Database (MsigDB): "Using 50 Hallmark gene sets from Molecular Signatures Database (MsigDB)43"
- [methods] Apply multiple-testing correction with independent hypothesis weighting (IHW) using average gene expression as the independent covariate: [evidence span withheld — anchor status: not_found]
