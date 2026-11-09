---
name: gmv-overlap-quantification
description: 'Use when reproducing VEGA''s Fig. 4 generalization results: a trained
  VEGA model on PBMC scRNA-seq (Kang et al. GSE96583) must be evaluated on unseen
  cells (held-out cell type/condition or the Zheng et al. study) by measuring top-50
  differential GMV overlap and reconstruction R2.'
when_to_use_negative:
- The analysis goal is differential gene expression at single-gene resolution rather
  than gene module (GMV) activity comparisons between studies or conditions
- The target data were seen during VEGA training (in-sample), since the overlap metric
  is defined for out-of-sample generalization
- The prior knowledge GMT does not cover the genes/modules of the new dataset, as
  hard-coded decoder connections cannot correct incomplete or non-context-specific
  MSigDB annotations
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: VEGA
  role: Trains the annotation-wired VAE and infers single-cell gene module activities
    (GMVs) for out-of-sample data; provides the Bayesian differential GMV activity
    test
  repo: https://github.com/LucasESBS/vega
- name: scanpy
  role: Loads, filters and normalizes the Kang et al. and Zheng et al. PBMC count
    matrices before passing them to VEGA
  repo: https://scanpy.readthedocs.io/en/stable/
- name: scvi-tools
  role: Backbone on which VegaSCVI is implemented, including the Bayesian differential
    module activity testing procedure
  repo: https://scvi-tools.org/
- name: vega-reproducibility
  role: Provides preprocessed data, train_all_models.sh, and jupyter notebooks used
    to reproduce the figure results including overlap statistics
  repo: https://github.com/LucasESBS/vega-reproducibility
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-26017-0
    title: VEGA is an interpretable generative model for inferring biological network
      activity in single-cell transcriptomics
  source_content_hash: sha256:e8e5760ada9f960c98dc71a73fde82b1efd46091c62a49f99262efdc68e6a1a5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LucasESBS/vega
  tools:
  - VEGA
  - scanpy
  - scvi-tools
  - vega-reproducibility
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# gmv-overlap-quantification

## Summary

Quantifies generalization of VEGA's gene module variable (GMV) activations by computing the overlap of the top 50 differentially activated GMVs between trained/study-A models and out-of-sample (held-out or cross-study) cells, targeting the reported ~72% (biological) and ~67% (cross-study Kang-vs-Zheng) overlaps.

## When to use

Use when reproducing VEGA's Fig. 4 generalization results: a trained VEGA model on PBMC scRNA-seq (Kang et al. GSE96583) must be evaluated on unseen cells (held-out cell type/condition or the Zheng et al. study) by measuring top-50 differential GMV overlap and reconstruction R2.

## When NOT to use

- The analysis goal is differential gene expression at single-gene resolution rather than gene module (GMV) activity comparisons between studies or conditions
- The target data were seen during VEGA training (in-sample), since the overlap metric is defined for out-of-sample generalization
- The prior knowledge GMT does not cover the genes/modules of the new dataset, as hard-coded decoder connections cannot correct incomplete or non-context-specific MSigDB annotations

## Inputs

- Preprocessed Kang et al. (GSE96583) PBMC single-cell count matrix (scanpy AnnData)
- Zheng et al. PBMC single-cell count matrix (held-out study B)
- MSigDB GMT file(s) specifying gene module variables (GMVs) and gene membership
- Trained VEGA model checkpoint(s) from ./trained_models/ (vega-reproducibility pipeline)

## Outputs

- Top-50 differential GMV overlap percentages for the biological (held-out pair, ~72%) and cross-study (~67%) experiments
- Reconstruction R2 values for out-of-sample settings and the raw baseline comparison
- Tidy results table saved as generalization_results.csv (Fig. 4a-f aggregation)

## How to apply

Train a VEGA (vega.VEGA / vega.VegaSCVI on the scvi-tools backbone) model with a sparse decoder wired by an MSigDB GMT file on the Kang et al. PBMC counts preprocessed with scanpy. For biological generalization, hold out one cell type/condition pair, infer single-cell GMV activities for the held-out cells, run the scvi-tools-inspired Bayesian differential gene module activity test on trained vs held-out cells, and compute the overlap of the top 50 differentially activated GMVs (target ~72%). For technical generalization, embed and reconstruct Zheng et al. cells out-of-sample from the Kang-trained model, compute top-50 GMV overlap (~67% target) and reconstruction R2, comparing to a raw study-A-vs-B baseline. Aggregate overlap percentages and R2 values into a tidy table (generalization_results.csv); expect only a marginal R2 decrease out-of-sample and R2 improvement over the baseline in the cross-study setting.

## Related tools

- **VEGA** (Trains the annotation-wired VAE and infers single-cell gene module activities (GMVs) for out-of-sample data; provides the Bayesian differential GMV activity test) — https://github.com/LucasESBS/vega
- **scanpy** (Loads, filters and normalizes the Kang et al. and Zheng et al. PBMC count matrices before passing them to VEGA) — https://scanpy.readthedocs.io/en/stable/
- **scvi-tools** (Backbone on which VegaSCVI is implemented, including the Bayesian differential module activity testing procedure) — https://scvi-tools.org/
- **vega-reproducibility** (Provides preprocessed data, train_all_models.sh, and jupyter notebooks used to reproduce the figure results including overlap statistics) — https://github.com/LucasESBS/vega-reproducibility

## Examples

```
bash train_all_models.sh
```

## Evaluation signals

- Top-50 differential GMV overlap for the biological generalization experiment (held-out cell type/condition) is close to the reported ~72% average
- Top-50 GMV overlap for the cross-study experiment (Kang-trained model on Zheng cells) is close to the reported ~67% average
- Reconstruction R2 shows only a marginal decrease in the biological out-of-sample setting
- Cross-study R2 improves over the raw study-A-vs-B baseline
- generalization_results.csv contains overlap entries for both experiments and R2 entries satisfying the above expectations

## Limitations

- The sparse, single-layer decoder architecture limits generalizability and robustness, so the overlap metric is bounded by this design choice
- VEGA's generative capacity is limited, sacrificing predictive (reconstruction) performance for biological interpretability of the latent space
- Prior biological knowledge from MSigDB can be incomplete or not context-specific, and hard-coded decoder connections leave no room for correcting gene module memberships

## Evidence

- [results] differential GMV overlap cross-study: [evidence span withheld — anchor status: not_found]
- [methods] Bayesian module testing: "VEGA provides a Bayesian testing procedure to find significantly activated gene modules in your dataset"
- [abstract] decoder wiring: "we introduce a novel sparse Variational Autoencoder architecture, VEGA (VAE Enhanced by Gene Annotations), whose decoder wiring mirrors user-provided gene modules"
- [readme] reproducibility repo: [evidence span withheld — anchor status: external_doc]
