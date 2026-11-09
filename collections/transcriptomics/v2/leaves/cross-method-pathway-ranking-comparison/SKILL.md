---
name: cross-method-pathway-ranking-comparison
description: Use when a single-cell differential comparison (e.g., stimulated vs control
  PBMCs) has been scored for pathway/gene-module activity by both a Bayes Factor test
  (VEGA) and a ranked-list enrichment method (GSEA) and the goal is a side-by-side
  significance comparison.
when_to_use_negative:
- The goal is pure gene-level differential expression ranking without any gene-module
  or pathway activity interpretation
- No prior-knowledge gene set (GMT, e.g., MSigDB) covers the input genes, so VEGA's
  sparse annotation-informed decoder and GSEA gene sets cannot both be defined on
  the same comparison
- The comparison is across datasets with different gene module definitions, making
  cross-method per-pathway FDR/BF pairing meaningless
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: VEGA
  role: Runs the scvi-tools-inspired Bayesian differential gene module activity test
    producing log_e(Bayes Factor) per GMV
  repo: https://github.com/LucasESBS/vega
- name: scanpy
  role: Loads and preprocesses the Kang PBMC dataset and subsets stimulated vs control
    cells in the AnnData object
- name: scvi-tools
  role: Ecosystem underpinning VEGA's Bayesian testing procedure for differential
    module activity
- name: gseapy
  role: Runs prerank GSEA on the gene-level differential expression ranking to obtain
    per-pathway -log10(FDR) values
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-26017-0
    title: VEGA is an interpretable generative model for inferring biological network
      activity in single-cell transcriptomics
  source_content_hash: sha256:0365bc4fbd40d5ba1808fa366a133f651f8ed442bdd2b4e9093351bfd92dfe28
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LucasESBS/vega
  tools:
  - VEGA
  - gseapy
  - scanpy
  - scvi-tools
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# cross-method-pathway-ranking-comparison

## Summary

Benchmark VEGA's differential gene module variable (GMV) Bayes Factor test against standard gseapy prerank GSEA FDR on the same single-cell comparison, exposing pathways (e.g., tryptophan catabolism in Kang PBMC innate immune cells) that one method detects and the other misses.

## When to use

Use when a single-cell differential comparison (e.g., stimulated vs control PBMCs) has been scored for pathway/gene-module activity by both a Bayes Factor test (VEGA) and a ranked-list enrichment method (GSEA) and the goal is a side-by-side significance comparison.

## When NOT to use

- The goal is pure gene-level differential expression ranking without any gene-module or pathway activity interpretation
- No prior-knowledge gene set (GMT, e.g., MSigDB) covers the input genes, so VEGA's sparse annotation-informed decoder and GSEA gene sets cannot both be defined on the same comparison
- The comparison is across datasets with different gene module definitions, making cross-method per-pathway FDR/BF pairing meaningless

## Inputs

- Preprocessed AnnData object of the Kang et al. PBMC dataset (GEO GSE96583) split into stimulated and control condition subsets (e.g., innate immune cells)
- GMT file specifying gene module variables (GMVs) and gene membership, e.g., from MSigDB
- Gene-level differential expression ranking between stimulated and control cells

## Outputs

- Volcano plot of differential GMV log_e(Bayes Factor) values showing only modules passing the |log_e(BF)| > 3 threshold (Fig. 1e-style)
- Side-by-side comparison figure of VEGA Bayes Factors vs GSEA -log10(FDR) per pathway, highlighting GSEA-missed pathways such as tryptophan catabolism (Fig. 1f-style)
- Differential GMV results table with gene module names, log_e(BF), GSEA FDR values, and a |log_e(BF)| > 3 significance flag

## How to apply

Run VEGA's scvi-tools-inspired differential GMV Bayesian test on the preprocessed AnnData subset (e.g., stimulated vs control innate immune cells from Kang et al. GSE96583) to obtain log_e(Bayes Factor) per gene module, and flag modules with |log_e(BF)| > 3 as significantly active. Independently run gseapy prerank GSEA on the gene-level differential expression ranking of the same comparison to obtain per-pathway -log10(FDR). Plot VEGA Bayes Factors side-by-side with GSEA -log10(FDR) per pathway, highlighting discordant calls (in the paper, GSEA detects interferon-alpha/beta signaling but misses tryptophan catabolism activation that VEGA calls significant). The rationale is that VEGA's module-activity test and GSEA's gene-set ranking are complementary, so cross-method concordance/difference is the validation signal.

## Related tools

- **VEGA** (Runs the scvi-tools-inspired Bayesian differential gene module activity test producing log_e(Bayes Factor) per GMV) — https://github.com/LucasESBS/vega
- **scanpy** (Loads and preprocesses the Kang PBMC dataset and subsets stimulated vs control cells in the AnnData object)
- **scvi-tools** (Ecosystem underpinning VEGA's Bayesian testing procedure for differential module activity)
- **gseapy** (Runs prerank GSEA on the gene-level differential expression ranking to obtain per-pathway -log10(FDR) values)

## Examples

```
bash train_all_models.sh  # train VEGA models from the vega-reproducibility repo, then reproduce figures in the jupyter notebooks
```

## Evaluation signals

- The results table contains a |log_e(BF)| > 3 significance flag for tryptophan catabolism, the reported GSEA false negative
- GSEA detects interferon-alpha/beta signaling activation while failing to call tryptophan catabolism significant, reproducing the paper's discordance
- The volcano plot displays only modules passing the |log_e(BF)| > 3 threshold
- All three output artifacts (volcano plot, comparison figure, results table) exist and the table includes gene module names, log_e(BF), and GSEA FDR columns

## Limitations

- VEGA's sparse, single-layer decoder limits generalizability and robustness, and its hard-coded connections leave no room for correcting prior knowledge about gene modules
- Prior biological knowledge from databases like MSigDB can be incomplete or not context-specific, so unannotated genes important to a module are missed
- VEGA's limited generative capacity sacrifices predictive performance for biological interpretability of the latent space

## Evidence

- [methods] VEGA Bayesian testing procedure: "VEGA provides a Bayesian testing procedure to find significantly activated gene modules in your dataset"
- [intro] VEGA architecture: "Here we propose VEGA (VAE enhanced by gene annotations), a VAE with a sparse linear decoder informed by biological networks"
- [readme] Reproducibility code: [evidence span withheld — anchor status: external_doc]
- [readme] Gene module input format: [evidence span withheld — anchor status: external_doc]
