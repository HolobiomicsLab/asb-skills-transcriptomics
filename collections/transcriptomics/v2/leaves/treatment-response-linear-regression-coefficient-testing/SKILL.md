---
name: treatment-response-linear-regression-coefficient-testing
description: Use when a scRNA-seq time-series has treated vs untreated conditions
  across timepoints (e.g., cisplatin X1–X4 in TNBC PDX) and GeneVector metagenes must
  be classified as treatment-specific using regression coefficients on module scores.
when_to_use_negative:
- The input lacks a treatment/condition contrast (e.g., a single-sample dataset or
  the PBMC interferon-stimulation cell-annotation workflow) — metagene module scores
  cannot be regressed on treatment status.
- Metagenes have not yet been extracted from the GeneVector embedding — this skill
  assumes GeneEmbedding.get_adata(resolution=40) and get_metagenes have already been
  run.
- The question is cell type annotation or batch correction rather than identifying
  treatment-dependent transcriptional programs — use CellEmbedding phenotype_probability
  or batch_correct instead.
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0092
tools:
- name: GeneVector
  role: Produces the gene embedding, co-expression clustering, and metagenes whose
    module scores are the regression response variable; also supplies the time-series
    (FITNESS) workflow this testing step completes
  repo: https://github.com/nceglia/genevector
- name: SciPy
  role: Linear regression and p-value computation for per-metagene coefficient testing
- name: NumPy
  role: Numerical arrays for module scores and design matrices in the regression
- name: Scanpy
  role: Loads the TNBC PDX cisplatin time-series AnnData used to compute metagene
    module scores per cell
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-39985-2
    title: Identification of transcriptional programs using dense vector representations
      defined by mutual information with GeneVector
  source_content_hash: sha256:ebdce826f5eb6c9afdef62a248ff60c4350004a6a7243dbda12cf44cd4a64a91
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nceglia/genevector
  tools:
  - AnnData
  - GeneVector
  - NumPy
  - PyTorch
  - Python
  - Scanpy
  - SciPy
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# treatment-response linear regression coefficient testing

## Summary

Statistical skill for calling treatment-dependent metagenes: per-metagene module scores are regressed on treatment status across timepoints, and coefficients/p-values are thresholded (beta_treated > 0.1, Bonferroni-corrected p = 0.001, beta_treated > beta_untreated) to nominate drug-resistance programs such as the TGF-beta-signaling metagene 24 in cisplatin-treated TNBC PDXs.

## When to use

Use when a scRNA-seq time-series has treated vs untreated conditions across timepoints (e.g., cisplatin X1–X4 in TNBC PDX) and GeneVector metagenes must be classified as treatment-specific using regression coefficients on module scores.

## When NOT to use

- The input lacks a treatment/condition contrast (e.g., a single-sample dataset or the PBMC interferon-stimulation cell-annotation workflow) — metagene module scores cannot be regressed on treatment status.
- Metagenes have not yet been extracted from the GeneVector embedding — this skill assumes GeneEmbedding.get_adata(resolution=40) and get_metagenes have already been run.
- The question is cell type annotation or batch correction rather than identifying treatment-dependent transcriptional programs — use CellEmbedding phenotype_probability or batch_correct instead.

## Inputs

- GeneVector metagene-to-gene assignments (output of GeneEmbedding.get_metagenes from a clustered gene adata, resolution=40)
- Per-cell metagene module scores computed on the TNBC PDX cisplatin time-series AnnData (timepoints X1–X4)
- Per-cell treatment status labels (treated vs untreated) and timepoint annotations

## Outputs

- Table of per-metagene regression coefficients (beta_treated, beta_untreated) and Bonferroni-corrected p-values (e.g., tnbc_metagene_coefficients.csv)
- Binary treatment-specificity calls per metagene (criteria: beta_treated > 0.1, Bonferroni p = 0.001, beta_treated > beta_untreated)
- Nominated treatment-dependent metagenes (e.g., metagene 24, TGF-beta signaling enriched with ID1-4, EPCAM, FOXP1)

## How to apply

After GeneVector clustering yields metagenes (GeneEmbedding get_adata with resolution=40, then get_metagenes), compute each metagene's module score per cell. For each metagene, fit a linear regression of the module score on treatment status across the timepoints X1–X4, yielding beta_treated and beta_untreated coefficients. Call a metagene treatment-specific when beta_treated > 0.1, the p-value passes Bonferroni correction at 0.001, and beta_treated > beta_untreated. In the source task this procedure recovered metagene 24 as TGF-beta-signaling enriched (ID1–ID4, EPCAM, FOXP1 overexpressed in chemotherapy-resistant samples), and per-metagene coefficients, Bonferroni p-values, and treatment-specificity calls were written to tnbc_metagene_coefficients.csv.

## Related tools

- **GeneVector** (Produces the gene embedding, co-expression clustering, and metagenes whose module scores are the regression response variable; also supplies the time-series (FITNESS) workflow this testing step completes) — https://github.com/nceglia/genevector
- **SciPy** (Linear regression and p-value computation for per-metagene coefficient testing)
- **NumPy** (Numerical arrays for module scores and design matrices in the regression)
- **Scanpy** (Loads the TNBC PDX cisplatin time-series AnnData used to compute metagene module scores per cell)

## Examples

```
gene_adata = gene_embed.get_adata(resolution=40); metagenes = gene_embed.get_metagenes(gene_adata); sc.tl.score_genes(adata, gene_list=metagenes['24']); fit OLS of module score on treatment status across X1-X4 and test beta_treated > 0.1, Bonferroni p = 0.001
```

## Evaluation signals

- Treatment-specific metagenes are recovered under all three criteria: beta_treated > 0.1, Bonferroni-corrected p = 0.001, and beta_treated > beta_untreated.
- Metagene 24 is identified as TGF-beta-signaling enriched with ID1, ID2, ID3, ID4, EPCAM, and FOXP1 among its top genes, all overexpressed in chemotherapy-resistant samples.
- The output CSV contains one row per metagene with coefficient, Bonferroni p-value, and treatment-specificity call columns (schema check against tnbc_metagene_coefficients.csv).
- Regression is fit separately per metagene across timepoints X1–X4, so the number of fits equals the number of extracted metagenes (invariant).

## Limitations

- GeneVector's underlying MI-based embedding cannot discern repression from activation, so 'increasing with treatment' calls reflect statistical dependency direction signed only by Pearson correlation, not biological causality.
- Empirical MI (and hence the metagenes being tested) is limited by the available number of cells, fidelity of counts, and discretization strategy (10 quantile bins, zero expression in bin 0).
- In the article the benchmarking metrics for related analyses are referenced only as Supplementary Fig. 6A–L, which is not included in the main text — a reproducibility gap for validating downstream calls.

## Evidence

- [intro] Identify cisplatin treatment-dependent transcriptional programs related to TGF-beta in TNBC PDXs: "we show GeneVector can identify cisplatin treatment dependent transcriptional programs related to TGF-beta in TNBC PDXs"
- [readme] Time-series metagene identification workflow: [evidence span withheld — anchor status: external_doc]
- [discussion] Metagenes clustered in unsupervised fashion: "The resulting vectors can be used to generate a gene co-expression graph, and can be clustered to predict transcriptional programs, or metagenes, in an unsupervised fashion"
- [results] Drug resistance signatures in TNBC PDXs: "In cisplatin treated TNBC PDXs, GeneVector uncovers transcriptional signatures active in drug resistance"
