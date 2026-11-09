---
name: reconstruction-r2-benchmarking
description: Use when testing whether a trained single-cell generative model (VEGA)
  generalizes to unseen cell type/condition pairs or to an entirely unseen second
  study, via GMV overlap and R2 metrics.
when_to_use_negative:
- When the goal is maximum reconstruction accuracy rather than an interpretable latent
  space — the sparse, single-layer decoder limits VEGA's generative capacity and sacrifices
  predictive performance for interpretability
- When gene modules must be corrected or extended beyond prior annotations — hard-coded
  decoder connections leave no room for correcting incomplete or non-context-specific
  MSigDB knowledge
- When input is raw unnormalized counts without the recommended scanpy filtering/normalization
  preprocessing
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: VEGA
  role: Trains the annotation-wired sparse VAE and infers single-cell GMVs for held-out
    or out-of-sample cells; provides the Bayesian differential GMV activity test
  repo: https://github.com/LucasESBS/vega
- name: scanpy
  role: Loads, filters, and normalizes the Kang and Zheng PBMC count matrices before
    passing data to VEGA
- name: scvi-tools
  role: Backbone for vega.VegaSCVI and source of the Bayesian differential gene module
    activity testing procedure
- name: vega-reproducibility
  role: Provides train_all_models.sh, preprocessed data archive, and jupyter notebooks
    to reproduce the paper's figures including generalization results
  repo: https://github.com/LucasESBS/vega-reproducibility
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-26017-0
    title: VEGA is an interpretable generative model for inferring biological network
      activity in single-cell transcriptomics
  source_content_hash: sha256:462166f285a76aa73c43d9a222c641c0e83726cd513703599e4168dfe03abbbe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LucasESBS/vega
  tools:
  - VEGA
  - scanpy
  - scvi-tools
  - vega-reproducibility
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# reconstruction-r2-benchmarking

## Summary

Benchmark out-of-sample generalization of the VEGA interpretable VAE by computing top-50 differential gene module value (GMV) overlaps between studies (Kang vs Zheng PBMC datasets) and reconstruction R2, showing ~67% GMV overlap and only marginal R2 decrease.

## When to use

Use when testing whether a trained single-cell generative model (VEGA) generalizes to unseen cell type/condition pairs or to an entirely unseen second study, via GMV overlap and R2 metrics.

## When NOT to use

- When the goal is maximum reconstruction accuracy rather than an interpretable latent space — the sparse, single-layer decoder limits VEGA's generative capacity and sacrifices predictive performance for interpretability
- When gene modules must be corrected or extended beyond prior annotations — hard-coded decoder connections leave no room for correcting incomplete or non-context-specific MSigDB knowledge
- When input is raw unnormalized counts without the recommended scanpy filtering/normalization preprocessing

## Inputs

- Kang et al. (GSE96583) PBMC single-cell count matrix
- Zheng et al. PBMC single-cell count matrix
- MSigDB GMT file(s) with gene module membership for the VEGA decoder mask
- Trained VEGA model (./trained_models/) from the vega-reproducibility pipeline

## Outputs

- generalization_results.csv: tidy table with top-50 differential GMV overlap percentages and reconstruction R2 values for biological and cross-study generalization (Fig. 4a-f)

## How to apply

Load and preprocess the Kang et al. (GSE96583) and Zheng et al. PBMC count matrices with scanpy (filtering and normalization) before passing data to VEGA. Parse MSigDB GMT gene module files to build the mask wiring the VEGA sparse linear decoder (vega.VEGA / vega.VegaSCVI on the scvi-tools backbone). For biological generalization, train VEGA on the Kang study with one cell type/condition pair held out, infer single-cell GMVs for the held-out pair, and run the scvi-tools-inspired Bayesian differential GMV activity test, computing overlap of the top 50 differentially activated GMVs (target ~72% average overlap with marginal R2 decrease). For technical generalization, train on Kang (study A), embed and reconstruct Zheng (study B) cells out-of-sample, compute top-50 GMV overlap (~67% target) and reconstruction R2, comparing against the raw study A-vs-B baseline. Aggregate overlap percentages and R2 values into a tidy table (generalization_results.csv) corresponding to Fig. 4a-f.

## Related tools

- **VEGA** (Trains the annotation-wired sparse VAE and infers single-cell GMVs for held-out or out-of-sample cells; provides the Bayesian differential GMV activity test) — https://github.com/LucasESBS/vega
- **scanpy** (Loads, filters, and normalizes the Kang and Zheng PBMC count matrices before passing data to VEGA)
- **scvi-tools** (Backbone for vega.VegaSCVI and source of the Bayesian differential gene module activity testing procedure)
- **vega-reproducibility** (Provides train_all_models.sh, preprocessed data archive, and jupyter notebooks to reproduce the paper's figures including generalization results) — https://github.com/LucasESBS/vega-reproducibility

## Examples

```
bash train_all_models.sh  # then use jupyter notebooks in LucasESBS/vega-reproducibility to compute top-50 GMV overlaps and R2 for Fig. 4 and write generalization_results.csv
```

## Evaluation signals

- Top-50 differential GMV overlap for cross-study generalization is close to the reported ~67% (Kang study A vs Zheng study B)
- Top-50 differential GMV overlap for held-out cell type/condition biological generalization is close to the reported ~72% average
- R2 decreases only marginally in the out-of-sample biological generalization setting
- Cross-study reconstruction R2 improves over the raw study A-vs-B baseline
- generalization_results.csv contains overlap and R2 entries for both experiments and matches Fig. 4a-f

## Limitations

- The sparse, single-layer decoder architecture prevents further improvement of generalizability and robustness
- VEGA's generative capacity is limited, sacrificing predictive performance for biological interpretability of the latent space
- Prior biological knowledge from MSigDB can be incomplete or not context-specific; hard-coded decoder connections cannot correct gene module definitions

## Evidence

- [results] cross-study GMV overlap finding: [evidence span withheld — anchor status: not_found]
- [methods] GMT input requirement: "GMT file(s) with the gene module membership, such as provided by databases like `MSigDB <https://www.gsea-msigdb.org/gsea/index.jsp>`_"
- [methods] preprocessing recommendation: "Note: We recommend to preprocess the single-cell dataset before passing it to VEGA."
- [methods] Bayesian testing procedure: "VEGA provides a Bayesian testing procedure to find significantly activated gene modules in your dataset"
- [discussion] architecture limitation: "The clear limitations of the current architecture resides in the sparse, single-layer decoder of the model. In fact, such an architectural design prevents the further improvement of generalizability"
- [readme] reproducibility repo usage: "Run the bash script to train all models:
``nbash train_all_models.sh
``nOnce done training, models are in ./trained_models/"
