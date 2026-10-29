---
name: cross-study-generalization-evaluation
description: Use when an interpretable latent-space model (e.g., VEGA) has been trained
  on one single-cell study and you must quantify generalization to held-out cell types/conditions
  or to a completely unseen second study.
when_to_use_negative:
- The dataset was seen fully during training — this skill only applies to out-of-sample
  (held-out or unseen-study) inference
- 'You need maximal generative/predictive accuracy rather than interpretable module
  activity: VEGA''s sparse single-layer decoder sacrifices generative capacity for
  interpretability'
- The gene module annotation (e.g., MSigDB) is unavailable or your genes are largely
  unannotated, since hard-coded decoder connections cannot correct incomplete prior
  knowledge
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0089
tools:
- name: VEGA
  role: Trains the annotation-masked VAE (vega.VEGA / vega.VegaSCVI on scvi-tools
    backbone) and infers GMV activities out-of-sample for held-out cells and the unseen
    study
  repo: https://github.com/LucasESBS/vega
- name: scanpy
  role: Loads and preprocesses the Kang and Zheng PBMC count matrices (filtering and
    normalization) before VEGA
- name: scvi-tools
  role: Backbone of VegaSCVI and source of the Bayesian differential gene module activity
    test used to compare trained vs. held-out cells
- name: vega-reproducibility
  role: Provides preprocessed data archive, train_all_models.sh, and jupyter notebooks
    to reproduce the paper generalization figures
  repo: https://github.com/LucasESBS/vega-reproducibility
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-26017-0
    title: VEGA is an interpretable generative model for inferring biological network
      activity in single-cell transcriptomics
  source_content_hash: sha256:444fed10a98ab71c52ab1b69619d67ab0a7e9c04efc2698d476a6966a45855bb
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
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# cross-study-generalization-evaluation

## Summary

Evaluates whether a VEGA-trained interpretable VAE generalizes out-of-sample by inferring gene module variable (GMV) activities for held-out cell type/condition pairs and for an entirely unseen second PBMC study (Kang vs. Zheng), quantified via top-50 differential GMV overlap and reconstruction R2.

## When to use

Use when an interpretable latent-space model (e.g., VEGA) has been trained on one single-cell study and you must quantify generalization to held-out cell types/conditions or to a completely unseen second study.

## When NOT to use

- The dataset was seen fully during training — this skill only applies to out-of-sample (held-out or unseen-study) inference
- You need maximal generative/predictive accuracy rather than interpretable module activity: VEGA's sparse single-layer decoder sacrifices generative capacity for interpretability
- The gene module annotation (e.g., MSigDB) is unavailable or your genes are largely unannotated, since hard-coded decoder connections cannot correct incomplete prior knowledge

## Inputs

- Preprocessed Kang et al. PBMC single-cell count matrix (GSE96583) in scanpy AnnData (.h5ad) form
- Zheng et al. PBMC single-cell count matrix as scanpy AnnData
- MSigDB GMT gene module files defining the decoder mask
- Trained VEGA / VegaSCVI model checkpoints

## Outputs

- Top-50 differentially activated GMV overlap percentages for the biological (~72%) and cross-study (~67%) generalization settings
- Reconstruction R2 values for out-of-sample and baseline comparisons
- generalization_results.csv tidy table aggregating Fig. 4a-f metrics

## How to apply

Preprocess the Kang et al. (GSE96583) and Zheng et al. PBMC count matrices with scanpy (filtering and normalization) as recommended before passing data to VEGA, and parse MSigDB GMT files to build the mask wiring the sparse linear decoder (vega.VEGA / vega.VegaSCVI on the scvi-tools backbone). For biological generalization, train VEGA on the Kang study with one cell type/condition pair held out, infer single-cell GMVs for the held-out pair, run the scvi-tools-inspired Bayesian differential gene module activity test on trained vs. held-out cells, and compute the overlap of the top 50 differentially activated GMVs (target ~72%). For technical cross-study generalization, train on study A (Kang), embed and reconstruct study B (Zheng) cells out-of-sample, compute top-50 GMV overlap (~67% target) and reconstruction R2 against the raw study A-vs-B baseline. Aggregate overlap percentages and R2 values (Fig. 4a-f) into a tidy table (generalization_results.csv).

## Related tools

- **VEGA** (Trains the annotation-masked VAE (vega.VEGA / vega.VegaSCVI on scvi-tools backbone) and infers GMV activities out-of-sample for held-out cells and the unseen study) — https://github.com/LucasESBS/vega
- **scanpy** (Loads and preprocesses the Kang and Zheng PBMC count matrices (filtering and normalization) before VEGA)
- **scvi-tools** (Backbone of VegaSCVI and source of the Bayesian differential gene module activity test used to compare trained vs. held-out cells)
- **vega-reproducibility** (Provides preprocessed data archive, train_all_models.sh, and jupyter notebooks to reproduce the paper generalization figures) — https://github.com/LucasESBS/vega-reproducibility

## Examples

```
import vega; vega.VEGA(my_scanpy_data, my_gmt_file, train=True)
```

## Evaluation signals

- Top-50 differential GMV overlap for cross-study (Kang→Zheng) setting lands near the reported ~67% average
- Top-50 differential GMV overlap for held-out cell type/condition pair lands near the reported ~72% average
- R2 shows only a marginal decrease in the out-of-sample biological generalization setting
- Cross-study R2 improves over the raw study A-vs-B reconstruction baseline
- generalization_results.csv contains overlap and R2 entries for both experiments (Fig. 4a-f coverage)

## Limitations

- VEGA's sparse, single-layer decoder prevents further improvement of generalizability and robustness
- VEGA's limited generative capacity sacrifices predictive performance for biological interpretability of the latent space
- Prior biological knowledge from databases like MSigDB can be incomplete or not context-specific, and hard-coded decoder connections leave no room for correcting prior knowledge

## Evidence

- [other] Cross-study top-50 GMV overlap finding: [evidence span withheld — anchor status: not_found]
- [intro] Two-fold scope including out-of-sample inference: "the scope of VEGA is twofold, (1) encoding data over an interpretable latent space and (2) inferring gene module activities for out-of-sample data"
- [methods] Preprocessing recommendation: "Note: We recommend to preprocess the single-cell dataset before passing it to VEGA."
- [methods] Bayesian differential module testing: "VEGA provides a Bayesian testing procedure to find significantly activated gene modules in your dataset"
- [readme] GMT input requirement: [evidence span withheld — anchor status: external_doc]
- [discussion] Decoder limitation on generalization: "The clear limitations of the current architecture resides in the sparse, single-layer decoder of the model."
