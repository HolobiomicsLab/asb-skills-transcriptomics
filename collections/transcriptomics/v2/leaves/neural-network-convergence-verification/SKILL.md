---
name: neural-network-convergence-verification
description: Use when running the ScribbleDom pipeline (run_other_visium.sh or run_other_st.sh)
  and you must confirm the training loop converged for each sample and that per-sample
  results were written to final_outputs/{dataset}/{samples[i]}/.
when_to_use_negative:
- When only verifying the R preprocessing step (get_genex_data_from_10x_h5.R) — convergence
  checking applies to the Python training stage that follows
- When no training run has been launched yet — this skill validates a completed or
  in-progress run, not configuration authoring
- When the goal is AutoScribbleDom output quality assessment (ARI comparison) rather
  than confirming the training loop itself ran to completion
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_3444
- http://edamontology.org/topic_3316
tools:
- name: ScribbleDom
  role: Semi-supervised convolutional network whose training loop (Inception-style
    convolution filters, Scribble Dropout) is monitored for convergence and whose
    outputs are checked for completeness
  repo: https://github.com/1alnoman/ScribbleDom
- name: Python 3.10.6
  role: Runtime in which the PyTorch training loop executes; convergence and output-file
    verification is performed on this stage
- name: Mclust
  role: Produces the mclust_backbone.csv automated scribble that the mclust-schema
    training consumes; its presence confirms the mclust run had valid input
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad594
    title: 'ScribbleDom: using scribble-annotated histology images to identify domains
      in spatial transcriptomics data'
  source_content_hash: sha256:672d05bab81ce2e2c328a7568e1c5a334ebede0d93a760a073b77875df069fce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1alnoman/ScribbleDom
  tools:
  - Mclust
  - Python
  - Python 3.10.6
  - R
  - ScribbleDom
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_3444
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# neural-network-convergence-verification

## Summary

Verify that the ScribbleDom semi-supervised convolutional network completes its full grid-search training loop (max_iter = 300 epochs, seed/alpha/beta sweeps) without divergence and that per-sample result files land in the expected output directory tree.

## When to use

Use when running the ScribbleDom pipeline (run_other_visium.sh or run_other_st.sh) and you must confirm the training loop converged for each sample and that per-sample results were written to final_outputs/{dataset}/{samples[i]}/.

## When NOT to use

- When only verifying the R preprocessing step (get_genex_data_from_10x_h5.R) — convergence checking applies to the Python training stage that follows
- When no training run has been launched yet — this skill validates a completed or in-progress run, not configuration authoring
- When the goal is AutoScribbleDom output quality assessment (ARI comparison) rather than confirming the training loop itself ran to completion

## Inputs

- config_expert.json and config_mclust.json (schema=expert/mclust, n_pcs=15, max_iter=300, beta_options=[0.25,0.3,0.35,0.4])
- Preprocessed data folder with principal components of spots, coordinates, and mclust_backbone.csv automated scribble
- manual_scribble.csv placed in preprocessed_data/{dataset}/{samples[i]}/ (optional, for expert schema)
- Space Ranger output directory (raw_gene_x/{dataset}/{samples[i]}/) with tissue_positions_list.csv, scalefactors_json.json, and {samples[i]}_filtered_feature_bc_matrix.h5

## Outputs

- Per-sample trained-model artifacts in model_outputs/{dataset}/{samples[i]}/
- Per-sample final result files in final_outputs/{dataset}/{samples[i]}/
- Confirmation that all grid-search iterations (max_iter=300, alpha 0.05–0.95, beta 0.25–0.4) completed without divergence

## How to apply

After launching training via ./run_other_visium.sh config_expert.json config_mclust.json, check that the loop runs for all max_iter = 300 epochs for every combination in the hyperparameter grid (seed_options, alpha_options spanning 0.05–0.95, beta_options spanning 0.25–0.4, lr_options [0.1]) for both the expert and mclust schemas. Confirm no divergence occurs during iterations — the loss, which integrates gene expression profile similarity and adherence to scribble input, should train stably toward convergence. Then verify that per-sample result files exist under model_outputs/ and final_outputs/{dataset}/{samples[i]}/ (e.g. final_outputs/cancers/bcdc_ffpe/ for the bcdc_ffpe Visium sample). For the mclust schema, confirm the training consumed the mclust_backbone.csv automated scribble produced during preprocessing; for the expert schema, the manual_scribble.csv placed in preprocessed_data/{dataset}/{samples[i]}/. A completed run yields both AutoScribbleDom (mclust) and expert-scribble outputs, allowing comparison against state-of-the-art baselines via adjusted Rand index.

## Related tools

- **ScribbleDom** (Semi-supervised convolutional network whose training loop (Inception-style convolution filters, Scribble Dropout) is monitored for convergence and whose outputs are checked for completeness) — https://github.com/1alnoman/ScribbleDom
- **Python 3.10.6** (Runtime in which the PyTorch training loop executes; convergence and output-file verification is performed on this stage)
- **Mclust** (Produces the mclust_backbone.csv automated scribble that the mclust-schema training consumes; its presence confirms the mclust run had valid input)

## Examples

```
chmod +x run_other_visium.sh && ./run_other_visium.sh config_expert.json config_mclust.json  # then check final_outputs/cancers/bcdc_ffpe/
```

## Evaluation signals

- Training loop completes all max_iter = 300 iterations without divergence for every alpha (0.05–0.95) and beta (0.25–0.4) grid value
- Result files appear under final_outputs/{dataset}/{samples[i]}/ (e.g. final_outputs/cancers/bcdc_ffpe/) for both config_expert.json and config_mclust.json runs
- Model artifacts are written to model_outputs/{dataset}/{samples[i]}/ during training
- The mclust run consumed mclust_backbone.csv from the preprocessed_data folder; the expert run consumed manual_scribble.csv if present
- Reproducibility holds via torch.backends.cudnn.deterministic = True and torch.backends.cudnn.benchmark = False (GPU) or CPU execution

## Limitations

- If manual_scribble.csv is not generated, only AutoScribbleDom (fully unsupervised mclust-backbone) results are produced
- Grid sweeping alpha over 0.05–0.95 and beta over 0.25–0.4 across 300 iterations per configuration is computationally expensive for large samples
- No changelog found in the repository, so behavior changes between code versions cannot be traced

## Evidence

- [methods] The scribble dropout rate is governed by a second hyperparameter, beta, which is tuned by grid search over 0.25 to 0.4: [evidence span withheld — anchor status: not_found]
- [readme] max_iter and beta_options in the config: ""max_iter": 300,
    "nConv": 1,
    "seed_options": [4],
    "alpha_options": [0.05,0.1,0.15,0.2,0.25,0.3,0.35,0.4,0.45,0.5,0.55,0.6,0.65,0.7,0.75,0.8,0.85,0.9,0.95],"
- [readme] final output location: "Get the final output in final_outputs folder in"
- [readme] deterministic reproducibility settings: "torch.backends.cudnn.deterministic = True
torch.backends.cudnn.benchmark = False"
- [abstract] loss function components: "ScribbleDom incorporates a loss function that integrates two crucial components: similarity in gene expression profiles and adherence to the valuable input of a human annotator through scribbles"
