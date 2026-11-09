---
name: hyperparameter-sweep-configuration
description: Use when benchmarking unsupervised manifold-alignment methods on single-cell
  multi-modal datasets and you need fair, article-specified hyperparameter sweeps
  (Methods 2.7) plus scTopoGAN model selection over 20 candidate GANs.
when_to_use_negative:
- When a single method configuration is already validated and published for your data
  — a full sweep wastes compute and is unnecessary
- When the input datasets have overlapping features or cells, since supervised/semi-supervised
  integration methods (not these unsupervised alignment baselines) are the appropriate
  choice per the article's scope
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_3517
tools:
- name: scTopoGAN
  role: Topology-guided GAN whose run_scTopoGAN selects from 20 candidate models by
    topological loss and refines the best for 1000 epochs — an internal model-selection
    sweep
  repo: https://github.com/AkashCiel/scTopoGAN
- name: UnionCom
  role: Baseline alignment method applied with Methods-2.7 hyperparameter sweeps via
    UnionCom.py
  repo: https://github.com/AkashCiel/scTopoGAN
- name: MMD-MA
  role: Baseline alignment method applied with Methods-2.7 hyperparameter sweeps via
    manifold_align_mmd_pytorch.py
  repo: https://github.com/AkashCiel/scTopoGAN
- name: topoAE
  role: Produces the per-modality latent manifolds that feed the GAN candidate-selection
    stage
  repo: https://github.com/AkashCiel/scTopoGAN
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbad171
    title: 'scTopoGAN: unsupervised manifold alignment of single-cell data'
  source_content_hash: sha256:9e0c1025673812d043749eeec721daa2c299941188150746323c010355aa4fc0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AkashCiel/scTopoGAN
  tools:
  - MMD-MA
  - SCOTv2
  - UnionCom
  - scTopoGAN
  - topoAE
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# hyperparameter-sweep-configuration

## Summary

Configure and execute hyperparameter sweeps for manifold-alignment baselines (UnionCom, MMD-MA, SCOTv2) and select/refine scTopoGAN GAN models so benchmark results on Full PBMC, Partial PBMC, BM, and scGEM datasets are comparable and reproducible against Table 2.

## When to use

Use when benchmarking unsupervised manifold-alignment methods on single-cell multi-modal datasets and you need fair, article-specified hyperparameter sweeps (Methods 2.7) plus scTopoGAN model selection over 20 candidate GANs.

## When NOT to use

- When a single method configuration is already validated and published for your data — a full sweep wastes compute and is unnecessary
- When the input datasets have overlapping features or cells, since supervised/semi-supervised integration methods (not these unsupervised alignment baselines) are the appropriate choice per the article's scope

## Inputs

- Pre-processed Full PBMC, Partial PBMC, BM, and scGEM datasets from the Data folder (shuffled per benchmark protocol)
- Per-modality topoAE low-dimensional embeddings (.npy) from get_TopoAE_Embeddings
- Methods-2.7 hyperparameter sweep definitions for UnionCom, MMD-MA, and SCOTv2

## Outputs

- Aligned source-to-target embeddings (.npy) per method and dataset
- Candidate GAN model selection logs (20 models ranked by topological loss; best refined for 1000 epochs)
- Per method-dataset metrics: Celltype matching, Subcelltype matching, batch LISI aggregated into a Table-2-equivalent CSV

## How to apply

For each method-dataset pair, apply the hyperparameter sweeps specified in Methods 2.7: run UnionCom via UnionCom.py and MMD-MA via manifold_align_mmd_pytorch.py with these sweeps, and run the SCOTv2 baseline with its Methods-2.7 settings. For scTopoGAN itself, first compute topoAE embeddings per modality with get_TopoAE_Embeddings, then run run_scTopoGAN which selects from 20 candidate GAN models by topological loss and refines the best model for 1000 epochs. Keep sweeps identical across datasets (Full PBMC, Partial PBMC, BM, scGEM) so downstream Celltype matching, Subcelltype matching, and batch LISI scores are directly comparable. The rationale is that scTopoGAN's topology-guided selection replaces a hand-tuned sweep: the topological loss over candidates acts as the model-selection criterion, while baselines require explicit sweeps for a fair unsupervised comparison.

## Related tools

- **scTopoGAN** (Topology-guided GAN whose run_scTopoGAN selects from 20 candidate models by topological loss and refines the best for 1000 epochs — an internal model-selection sweep) — https://github.com/AkashCiel/scTopoGAN
- **UnionCom** (Baseline alignment method applied with Methods-2.7 hyperparameter sweeps via UnionCom.py) — https://github.com/AkashCiel/scTopoGAN
- **MMD-MA** (Baseline alignment method applied with Methods-2.7 hyperparameter sweeps via manifold_align_mmd_pytorch.py) — https://github.com/AkashCiel/scTopoGAN
- **topoAE** (Produces the per-modality latent manifolds that feed the GAN candidate-selection stage) — https://github.com/AkashCiel/scTopoGAN

## Examples

```
python run_scTopoGAN.py  # then: python UnionCom.py; python manifold_align_mmd_pytorch.py  # with Methods-2.7 sweep settings per dataset
```

## Evaluation signals

- The output CSV contains exactly one Celltype matching, Subcelltype matching, and batch LISI entry per method-dataset combination
- Each scTopoGAN run reports 20 candidate GAN models evaluated by topological loss, with the selected model refined for 1000 epochs
- Baseline runs used the identical Methods-2.7 sweep settings across Full PBMC, Partial PBMC, BM, and scGEM
- Resulting batch LISI values approach the optimal value of 2 for the best-integrating method, matching the Table 2 pattern
- Metric values reproduce the corresponding Table 2 results reported in the scTopoGAN preprint

## Limitations

- Hyperparameter sweep settings are dataset-specific to the benchmark (Full PBMC, Partial PBMC, BM, scGEM); the README provides no changelog or documented defaults for new data
- The GAN model selection depends on topological loss as the sole criterion on unsupervised data with no ground-truth correspondence, so a poorly matching candidate may still be selected
- Candidate count (20) and refinement length (1000 epochs) are fixed by run_scTopoGAN; deviating from them invalidates comparison with the published Table 2

## Evidence

- [methods] run_scTopoGAN selecting from 20 candidates: "```run_scTopoGAN``` which aligns the source manifold to the target manifold using the topology-guided GAN model."
- [methods] baselines via dedicated scripts: "The ```UnionCom.py``` and ```manifold_align_mmd_pytorch.py``` scripts are used to apply UnionCom and MMD-MA, respectively."
- [methods] pre-processed Data folder input: "this script takes as input the pre-processed data provided in the ```Data``` folder"
- [full_text] benchmark workflow from source card: [evidence span withheld — anchor status: not_found]
- [discussion] no changelog found: "_No changelog found._"
