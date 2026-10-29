---
name: single-cell-perturbation-differential-testing
description: Use when comparing perturbed vs control single-cell RNA-seq cells and
  asking which pre-annotated gene modules/pathways (GMT-defined GMVs) are significantly
  differentially active, with module-level significance and a GSEA cross-check.
when_to_use_negative:
- You have no prior gene-module annotation (GMT file) — VEGA's interpretable latent
  space and GMV testing require user-provided gene modules.
- The question is gene-level differential expression rather than module/pathway-level
  activity — use a standard DE tool instead of the VEGA BF test.
- The dataset is not count/normalized single-cell expression prepared per VEGA's preprocessing
  recommendations (article notes count-data handling as a limitation; SFig. 8).
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: VEGA
  role: Sparse-decoder VAE that computes the differential gene module activity Bayesian
    test (log_e Bayes Factor) between conditions
  repo: https://github.com/LucasESBS/vega
- name: scanpy
  role: Loads and preprocesses the single-cell dataset into the AnnData object consumed
    by VEGA
  repo: https://scanpy.readthedocs.io/en/stable/
- name: scvi-tools
  role: Ecosystem whose Bayesian testing procedure VEGA's differential GMV test follows
  repo: https://scvi-tools.org/
- name: gseapy
  role: Runs prerank GSEA on the gene-level DE ranking to obtain per-pathway FDR values
    for comparison against VEGA BFs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-26017-0
    title: VEGA is an interpretable generative model for inferring biological network
      activity in single-cell transcriptomics
  source_content_hash: sha256:da67be1a7338c0c72544d8acf5549539f890e629bc7e60c54acadb77522339a8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LucasESBS/vega
  tools:
  - VEGA
  - gseapy
  - scanpy
  - scvi-tools
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# single-cell-perturbation-differential-testing

## Summary

Bayesian differential gene module activity testing with VEGA on single-cell perturbation data: computes log_e(Bayes Factors) for each gene module variable (GMV) between conditions (e.g., stimulated vs control Kang PBMC innate immune cells) and benchmarks module calls against GSEA FDR values.

## When to use

Use when comparing perturbed vs control single-cell RNA-seq cells and asking which pre-annotated gene modules/pathways (GMT-defined GMVs) are significantly differentially active, with module-level significance and a GSEA cross-check.

## When NOT to use

- You have no prior gene-module annotation (GMT file) — VEGA's interpretable latent space and GMV testing require user-provided gene modules.
- The question is gene-level differential expression rather than module/pathway-level activity — use a standard DE tool instead of the VEGA BF test.
- The dataset is not count/normalized single-cell expression prepared per VEGA's preprocessing recommendations (article notes count-data handling as a limitation; SFig. 8).

## Inputs

- Preprocessed single-cell dataset wrapped in a scanpy AnnData object (e.g., Kang et al. PBMCs, GEO GSE96583) with condition and cell-type annotations
- GMT file specifying gene module variables (GMVs) and gene membership, e.g., from MSigDB
- Trained VEGA model (pretrained models from the vega-reproducibility repo or trained via train_all_models.sh)

## Outputs

- Table of gene module names with log_e(Bayes Factor), GSEA FDR values, and a |log_e(BF)| > 3 significance flag
- Volcano plot of differential GMV results (modules passing the |log_e(BF)| > 3 threshold)
- Side-by-side comparison figure of VEGA Bayes Factors vs GSEA -log10(FDR) per pathway

## How to apply

Load the preprocessed single-cell dataset with scanpy (e.g., Kang et al. PBMCs, GEO GSE96583) as an AnnData object and a GMT file specifying the gene module variables and gene membership (e.g., from MSigDB); VEGA's sparse decoder wiring mirrors these user-provided gene modules. Subset the cells of interest by cell type and condition (stimulated vs control). Run VEGA's differential gene module activity test, which follows a scvi-tools-inspired Bayesian testing procedure, to compute the log_e(Bayes Factor) per GMV between conditions, and flag modules with |log_e(BF)| > 3 as significantly differentially active; render the volcano plot. As a benchmark, run gseapy prerank GSEA on the gene-level differential expression ranking of the same comparison to obtain per-pathway -log10(FDR), then plot VEGA BFs side-by-side with GSEA FDRs (e.g., as Fig. 1f) to surface modules VEGA calls significant that GSEA misses (e.g., tryptophan catabolism in innate immune cells).

## Related tools

- **VEGA** (Sparse-decoder VAE that computes the differential gene module activity Bayesian test (log_e Bayes Factor) between conditions) — https://github.com/LucasESBS/vega
- **scanpy** (Loads and preprocesses the single-cell dataset into the AnnData object consumed by VEGA) — https://scanpy.readthedocs.io/en/stable/
- **scvi-tools** (Ecosystem whose Bayesian testing procedure VEGA's differential GMV test follows) — https://scvi-tools.org/
- **gseapy** (Runs prerank GSEA on the gene-level DE ranking to obtain per-pathway FDR values for comparison against VEGA BFs)

## Examples

```
pip install scvega; import vega  # train VEGA on scanpy AnnData with GMT-defined GMVs, then run the differential module-activity Bayesian test for stimulated vs control cells
```

## Evaluation signals

- Results table contains a |log_e(BF)| > 3 significance flag for tryptophan catabolism (the reported GSEA false negative in innate immune cells).
- Volcano plot displays only modules passing the |log_e(BF)| > 3 threshold (expected hits: interferon signaling, tryptophan catabolism).
- All three output artifacts exist: volcano plot, BF-vs-GSEA comparison figure, and differential GMV results table with gene module names, log_e(BF), and GSEA FDR columns.
- GSEA reproducibly detects interferon-α/β signaling activation while VEGA additionally calls tryptophan catabolism, matching the paper's reported comparison.

## Limitations

- Prior biological knowledge from databases like MSigDB can be incomplete or not context-specific, as additional unannotated genes can play an important role in certain gene modules.
- VEGA's sparse, single-layer decoder with hard-coded connections limits generalizability and leaves no room for correcting prior knowledge when context requires it.
- VEGA's limited generative capacity sacrifices predictive performance for biological interpretability of the latent space.

## Evidence

- [methods] Bayesian testing procedure on preprocessed AnnData + GMT: "VEGA provides a Bayesian testing procedure to find significantly activated gene modules in your dataset"
- [readme] Required inputs: AnnData and GMT: [evidence span withheld — anchor status: external_doc]
- [abstract] Decoder wiring mirrors gene modules for interpretability: "whose decoder wiring mirrors user-provided gene modules, providing direct interpretability to the latent variables"
- [methods] Preprocessing recommendation: "Note: We recommend to preprocess the single-cell dataset before passing it to VEGA."
- [readme] Reproducibility code and pretrained models: [evidence span withheld — anchor status: external_doc]
