---
name: volcano-plot-visualization-of-differential-activity
description: Use when VEGA's Bayesian differential module activity test yields per-module
  log_e(Bayes Factor) values for a two-condition single-cell comparison and you need
  to visualize and flag significantly activated gene modules.
when_to_use_negative:
- Input is gene-level differential expression output only, with no per-module Bayes
  Factors from VEGA's Bayesian testing procedure
- The dataset has not been preprocessed as VEGA expects, since the authors recommend
  preprocessing the single-cell dataset before passing it to VEGA
- A pathway-level enrichment p-value alone (without per-module BF magnitudes) is sufficient,
  in which case a simple bar plot of -log10(FDR) suffices
tools:
- name: VEGA
  role: Produces the per-gene-module log_e(Bayes Factor) values that populate the
    volcano plot via its differential gene module activity test
  repo: https://github.com/LucasESBS/vega
- name: scanpy
  role: Loads and subsets the Kang PBMC dataset into stimulated vs control innate
    immune cell AnnData objects feeding the differential test
- name: scvi-tools
  role: Ecosystem whose Bayesian testing procedure inspires VEGA's differential module
    activity computation
- name: gseapy
  role: Runs prerank GSEA to obtain per-pathway -log10(FDR) for the side-by-side comparison
    figure
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-26017-0
    title: VEGA is an interpretable generative model for inferring biological network
      activity in single-cell transcriptomics
  source_content_hash: sha256:1c9fb46efdbbd4c3bf6094bdd2defdf182f3ac69ef659509c9381cb791f1cc3c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LucasESBS/vega
  tools:
  - VEGA
  - gseapy
  - scanpy
  - scvi-tools
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# volcano-plot-visualization-of-differential-activity

## Summary

Render VEGA differential gene module variable (GMV) test results as a volcano plot (Fig. 1e), plotting log_e(Bayes Factor) per pathway with significance flagged at |log_e(BF)| > 3, optionally alongside GSEA -log10(FDR) comparison (Fig. 1f) to expose false negatives such as tryptophan catabolism in Kang PBMC innate immune cells.

## When to use

Use when VEGA's Bayesian differential module activity test yields per-module log_e(Bayes Factor) values for a two-condition single-cell comparison and you need to visualize and flag significantly activated gene modules.

## When NOT to use

- Input is gene-level differential expression output only, with no per-module Bayes Factors from VEGA's Bayesian testing procedure
- The dataset has not been preprocessed as VEGA expects, since the authors recommend preprocessing the single-cell dataset before passing it to VEGA
- A pathway-level enrichment p-value alone (without per-module BF magnitudes) is sufficient, in which case a simple bar plot of -log10(FDR) suffices

## Inputs

- Preprocessed AnnData object of the Kang et al. PBMC dataset (GEO GSE96583), subset into stimulated and control innate immune cells
- VEGA differential test results table with log_e(Bayes Factor) per gene module
- GMT file specifying gene module (GMV) membership, e.g., from MSigDB
- gseapy prerank GSEA per-pathway FDR values on the gene-level differential expression ranking

## Outputs

- Volcano plot of differential GMV activity (Fig. 1e), displaying only modules passing |log_e(BF)| > 3
- Side-by-side comparison figure of VEGA Bayes Factors vs GSEA -log10(FDR) per pathway (Fig. 1f)
- Results table with gene module names, log_e(BF), and GSEA FDR values including a significance flag

## How to apply

Run VEGA's differential gene module activity test (the scvi-tools-inspired Bayesian testing procedure) on the two cell subsets of a preprocessed AnnData object (e.g., Kang et al. PBMC GSE96583 stimulated vs control innate immune cells) to obtain log_e(BF) for every GMV. Filter the modules table at |log_e(BF)| > 3 and render the volcano plot showing only modules passing this threshold, reproducing Fig. 1e. To support the comparison figure (Fig. 1f), also run gseapy prerank GSEA on the gene-level differential expression ranking to get per-pathway -log10(FDR), then plot VEGA BFs side-by-side with GSEA FDR per pathway, highlighting modules VEGA calls significant that GSEA misses (tryptophan catabolism). Save the volcano plot, the comparison figure, and a results table with gene module names, log_e(BF), and GSEA FDR values; per the reproducibility repo, trained models feed jupyter notebooks used to reproduce figures.

## Related tools

- **VEGA** (Produces the per-gene-module log_e(Bayes Factor) values that populate the volcano plot via its differential gene module activity test) — https://github.com/LucasESBS/vega
- **scanpy** (Loads and subsets the Kang PBMC dataset into stimulated vs control innate immune cell AnnData objects feeding the differential test)
- **scvi-tools** (Ecosystem whose Bayesian testing procedure inspires VEGA's differential module activity computation)
- **gseapy** (Runs prerank GSEA to obtain per-pathway -log10(FDR) for the side-by-side comparison figure)

## Examples

```
modules[np.abs(modules['log_e_BF']) > 3]
```

## Evaluation signals

- All three output files exist: volcano plot, BF-vs-GSEA comparison figure, and the results table
- The results table contains a |log_e(BF)| > 3 significance flag for tryptophan catabolism, the reported GSEA false negative in innate immune cells
- The volcano plot displays only gene modules passing the |log_e(BF)| > 3 threshold (interferon signaling and tryptophan catabolism in the Kang PBMC case)
- Interferon-α/β signaling is detected as activated in both the VEGA BF analysis and GSEA, consistent with the paper's finding

## Limitations

- Significance depends entirely on the |log_e(BF)| > 3 cutoff; modules below threshold are not shown on the volcano plot
- GSEA on the gene-level ranking can miss activations VEGA detects (e.g., tryptophan catabolism), so agreement between the two views should not be assumed
- Prior biological knowledge (e.g., MSigDB GMTs) defining the GMVs can be incomplete or not context-specific, affecting which modules can ever appear as significant

## Evidence

- [methods] Bayesian testing threshold: "VEGA provides a Bayesian testing procedure to find significantly activated gene modules in your dataset"
- [other] volcano plot construction from source card workflow: [evidence span withheld — anchor status: not_found]
- [other] GSEA comparison rationale: [evidence span withheld — anchor status: not_found]
- [readme] figure reproduction workflow: [evidence span withheld — anchor status: external_doc]
