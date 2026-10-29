---
name: spatial-domain-detection-semi-supervised-training
description: Use when running ScribbleDom training on Visium or ST spatial transcriptomics
  samples (e.g. bcdc_ffpe, melanoma, human DLPFC) with either expert manual_scribble.csv
  or Mclust-generated automated scribbles, and you need to tune the scribble-dropout
  rate beta.
when_to_use_negative:
- Input is not spatial transcriptomics spot data (Visium or ST platform) — the pipeline
  expects tissue_positions_list.csv, scalefactors_json.json, and filtered_feature_bc_matrix.h5
  or sample .rds
- No scribble source is available and a fully supervised expert workflow is required
  — without manual_scribble.csv you get only AutoScribbleDom results
- The goal is domain detection on single-cell (non-spatial) data without histology
  images or spot coordinates
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3678
tools:
- name: ScribbleDom
  role: Semi-supervised convolutional network trained with Scribble Dropout (rate
    beta) to identify spatial domains
  repo: https://github.com/1alnoman/ScribbleDom
- name: Mclust
  role: Generates the automated scribble (mclust_backbone.csv) used as noisy training
    signal that Scribble Dropout regularizes
- name: Loupe browser
  role: Optional source of expert manual_scribble.csv used under the expert schema
    instead of the automated scribble
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad594
    title: 'ScribbleDom: using scribble-annotated histology images to identify domains
      in spatial transcriptomics data'
  source_content_hash: sha256:6a9ae24286cfb1b02d2a64861d898c8067036ed82631fecf45b9923a51735a50
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1alnoman/ScribbleDom
  tools:
  - Loupe browser
  - Mclust
  - Python
  - R
  - ScribbleDom
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3678
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# spatial-domain-detection-semi-supervised-training

## Summary

Train the ScribbleDom semi-supervised convolutional network to identify spatial domains in spatial transcriptomics data, using Scribble Dropout (rate beta, swept 0.25–0.4) to counter residual noise in automatically generated scribbles, with alpha swept 0.05–0.95 via grid search.

## When to use

Use when running ScribbleDom training on Visium or ST spatial transcriptomics samples (e.g. bcdc_ffpe, melanoma, human DLPFC) with either expert manual_scribble.csv or Mclust-generated automated scribbles, and you need to tune the scribble-dropout rate beta.

## When NOT to use

- Input is not spatial transcriptomics spot data (Visium or ST platform) — the pipeline expects tissue_positions_list.csv, scalefactors_json.json, and filtered_feature_bc_matrix.h5 or sample .rds
- No scribble source is available and a fully supervised expert workflow is required — without manual_scribble.csv you get only AutoScribbleDom results
- The goal is domain detection on single-cell (non-spatial) data without histology images or spot coordinates

## Inputs

- Preprocessed data folder with principal components of spots, coordinates, and mclust_backbone.csv (automated scribble)
- Optional manual_scribble.csv from Loupe browser placed in preprocessed_data/{dataset}/{samples[i]}/
- config_expert.json and config_mclust.json (schema=expert/mclust, n_pcs=15, max_iter=300, beta_options=[0.25,0.3,0.35,0.4])
- Space Ranger output count matrix (.h5 for Visium) or .rds file for ST data

## Outputs

- Trained ScribbleDom model checkpoints under model_outputs/
- Per-sample spatial domain results in final_outputs/{dataset}/{samples[i]}/

## How to apply

Prepare config_mclust.json and config_expert.json (differing only in the schema field: 'mclust' vs 'expert') with n_pcs = 15, max_iter = 300, alpha_options spanning 0.05–0.95 and beta_options = [0.25, 0.3, 0.35, 0.4]. After R preprocessing (top 2000 highly variable genes, principal components, mclust_backbone.csv automated scribble), launch training via './run_other_visium.sh config_expert.json config_mclust.json' for Visium or run_other_st.sh for ST data. During each training iteration, Scribble Dropout randomly removes scribbles from a fraction of scribbled spots: beta governs this dropout rate and is grid-searched jointly with alpha (the scribble-loss weighting) to mitigate residual noise in automated scribbles. The network uses Inception-style convolution filters of varying sizes to capture spot microenvironment/spatial continuity, and combines gene-expression-profile similarity with scribble adherence in its loss. Judge success by convergence over up to max_iter = 300 epochs without divergence and by appearance of per-sample results in final_outputs/{dataset}/{samples[i]}/.

## Related tools

- **ScribbleDom** (Semi-supervised convolutional network trained with Scribble Dropout (rate beta) to identify spatial domains) — https://github.com/1alnoman/ScribbleDom
- **Mclust** (Generates the automated scribble (mclust_backbone.csv) used as noisy training signal that Scribble Dropout regularizes)
- **Loupe browser** (Optional source of expert manual_scribble.csv used under the expert schema instead of the automated scribble)

## Examples

```
chmod +x run_other_visium.sh; ./run_other_visium.sh config_expert.json config_mclust.json
```

## Evaluation signals

- Training completes all iterations up to max_iter = 300 without divergence
- Per-sample result files appear under final_outputs/{dataset}/{samples[i]}/ directory structure
- Grid search over beta_options = [0.25, 0.3, 0.35, 0.4] and alpha_options = 0.05–0.95 finishes for both expert and mclust schema configs
- Resulting ARI improvements should be in the reported range (1.82%–169.38% on 9/12 human DLPFC samples; 15.54% on the melanoma dataset) when benchmarking
- Deterministic GPU behavior confirmed via torch.backends.cudnn.deterministic = True and torch.backends.cudnn.benchmark = False

## Limitations

- Scribble Dropout's beta and alpha must be re-tuned by grid search per dataset; the recommended beta sweep is limited to 0.25–0.4 (interval 0.05)
- Without manual_scribble.csv, only AutoScribbleDom results are produced (fully unsupervised mode)
- Reproducibility requires the cudnn deterministic settings on GPU; they are unnecessary on CPU
- Pipeline targets Linux with specific versions (Python 3.10.6, R 4.3.1, conda 4.12.0) and needs libcairo2-dev/libxt-dev installed

## Evidence

- [results] beta grid search rationale: [evidence span withheld — anchor status: not_found]
- [abstract] Inception blocks: "The spatial continuity of the tissue domains is taken into account by extracting information on the spot microenvironment through convolution filters of varying sizes"
- [readme] config contents: [evidence span withheld — anchor status: not_found]
- [readme] launch command: "chmod +x run_other_visium.sh
./run_other_visium.sh config_expert.json config_mclust.json"
- [readme] determinism: "torch.backends.cudnn.deterministic = True
torch.backends.cudnn.benchmark = False"
- [readme] auto scribble output: "This step will produce preprocessed data with principal components of spots, coordinates and mclust_backbone.csv as Automated Scribble"
