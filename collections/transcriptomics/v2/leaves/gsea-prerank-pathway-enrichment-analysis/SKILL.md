---
name: gsea-prerank-pathway-enrichment-analysis
description: Use when you have a ranked gene list from a differential expression comparison
  and need per-pathway enrichment FDR values, especially to benchmark model-based
  module activity tests (e.g., VEGA Bayes Factors) against a standard enrichment method.
when_to_use_negative:
- When the goal is cell-type or condition-level interpretability of latent module
  activity rather than gene-set enrichment — VEGA's Bayesian GMV test is designed
  for that and can detect activations GSEA misses (e.g., tryptophan catabolism in
  innate immune cells)
- When the input is raw single-cell counts that have not been preprocessed into a
  differential expression ranking — prerank GSEA requires a ranked gene list as input
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3170
tools:
- name: gseapy
  role: Runs prerank GSEA on the gene-level differential expression ranking to compute
    per-pathway -log10(FDR) values for comparison against VEGA Bayes Factors
- name: scanpy
  role: Loads and subsets the Kang et al. PBMC AnnData object (GEO GSE96583) into
    stimulated and control innate immune cells
  repo: https://scanpy.readthedocs.io/en/stable/
- name: VEGA
  role: Provides the Bayesian differential GMV activity test whose |log_e(BF)| > 3
    calls are compared against GSEA FDR values
  repo: https://github.com/LucasESBS/vega
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-26017-0
    title: VEGA is an interpretable generative model for inferring biological network
      activity in single-cell transcriptomics
  source_content_hash: sha256:78130ec617e5ec10abcfa88e8c43a01873d08a52df145afc586d7a741394fecb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - VEGA
  - gseapy
  - scanpy
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# gsea-prerank-pathway-enrichment-analysis

## Summary

Run gseapy prerank GSEA on a gene-level differential expression ranking (e.g., stimulated vs control innate immune cells from the Kang et al. PBMC dataset, GEO GSE96583) to obtain per-pathway -log10(FDR) enrichment values. Used in the VEGA paper as the standard-enrichment baseline against which VEGA's Bayesian gene module variable (GMV) Bayes Factor test is compared.

## When to use

Use when you have a ranked gene list from a differential expression comparison and need per-pathway enrichment FDR values, especially to benchmark model-based module activity tests (e.g., VEGA Bayes Factors) against a standard enrichment method.

## When NOT to use

- When the goal is cell-type or condition-level interpretability of latent module activity rather than gene-set enrichment — VEGA's Bayesian GMV test is designed for that and can detect activations GSEA misses (e.g., tryptophan catabolism in innate immune cells)
- When the input is raw single-cell counts that have not been preprocessed into a differential expression ranking — prerank GSEA requires a ranked gene list as input

## Inputs

- Gene-level differential expression ranking (e.g., gene names with ranking metric) from stimulated vs control innate immune cells (Kang et al. PBMCs, GEO GSE96583, preprocessed AnnData)
- GMT file specifying pathway gene sets and gene membership (e.g., from MSigDB)

## Outputs

- Per-pathway -log10(FDR) enrichment values from gseapy prerank
- Side-by-side comparison figure of VEGA Bayes Factors and GSEA -log10(FDR) per pathway (Fig. 1f in the paper)
- Results table with gene module names, log_e(BF), and GSEA FDR values

## How to apply

First compute a gene-level differential expression ranking between the two conditions of interest (in the VEGA paper, stimulated vs control innate immune cells from the Kang et al. PBMC dataset). Load pathway gene sets from a GMT file (e.g., MSigDB, Reactome) and run gseapy's prerank mode on the ranking to obtain per-pathway -log10(FDR) values. Plot these side-by-side with VEGA's log_e(Bayes Factor) per gene module to identify concordant activations (e.g., interferon-α/β signaling) and false negatives (e.g., tryptophan catabolism activation in innate immune cells, which GSEA misses but VEGA calls significant with |log_e(BF)| > 3). Save the merged results table with gene module names, log_e(BF), and GSEA FDR values.

## Related tools

- **gseapy** (Runs prerank GSEA on the gene-level differential expression ranking to compute per-pathway -log10(FDR) values for comparison against VEGA Bayes Factors)
- **scanpy** (Loads and subsets the Kang et al. PBMC AnnData object (GEO GSE96583) into stimulated and control innate immune cells) — https://scanpy.readthedocs.io/en/stable/
- **VEGA** (Provides the Bayesian differential GMV activity test whose |log_e(BF)| > 3 calls are compared against GSEA FDR values) — https://github.com/LucasESBS/vega

## Examples

```
import gseapy; pre_res = gseapy.prerank(rnk=de_ranking, terms=msigdb_gmt, outdir='gsea_prerank_out')
```

## Evaluation signals

- Results table contains a |log_e(BF)| > 3 significance flag for tryptophan catabolism — the reported GSEA false negative — confirming the comparison captures the discordance
- GSEA detects interferon-α/β signaling activation, matching VEGA's significant call for the same comparison (concordance check)
- All three output artifacts exist: the comparison figure, the differential GMV results table, and the associated volcano plot
- GSEA returns finite -log10(FDR) values for all pathways in the GMT file, with no dropped gene sets due to missing genes in the ranking

## Limitations

- GSEA missed the tryptophan catabolism activation in innate immune cells that VEGA's Bayes Factor test detected, so prerank GSEA alone may under-report module activations
- Prior biological knowledge from databases like MSigDB can be incomplete or not context-specific, limiting enrichment analysis of unannotated genes
- Reproducibility code for the paper requires cloning the 'revision1' branch of the vega-reproducibility repository for up-to-date instructions

## Evidence

- [methods] VEGA provides a Bayesian testing procedure to find significantly activated gene modules in your dataset: "VEGA provides a Bayesian testing procedure to find significantly activated gene modules in your dataset"
- [methods] GMT file specification: "GMT file(s) with the gene module membership, such as provided by databases like `MSigDB <https://www.gsea-msigdb.org/gsea/index.jsp>`_"
- [discussion] MSigDB incompleteness caveat: "prior biological knowledge obtained from existing databases like MSigDB can be incomplete or not context-specific"
- [readme] Reproducibility instructions: [evidence span withheld — anchor status: external_doc]
