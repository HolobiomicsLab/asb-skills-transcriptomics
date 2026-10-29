---
name: hold-out-classification-accuracy-evaluation
description: Use when training or validating the Forseti oligo(dT) binding-affinity
  MLP on labeled priming windows and you need held-out generalization accuracies on
  training, hold-out, and test scRNA-seq datasets.
when_to_use_negative:
- The goal is predicting splicing status of scRNA-seq reads on a new dataset rather
  than training/evaluating the binding-affinity model — use the full Forseti prediction
  workflow (spliceu index, STAR alignment, forseti_predictor) instead
- Primed transcripts lack labeled priming windows with a valid adenine single-nucleotide
  repeat of length >= 6, so no valid training or hold-out windows can be extracted
- You need splicing-status prediction quality metrics (e.g., AUC on simulated or experimental
  reads) rather than binding-affinity classification accuracy
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Forseti
  role: Python package used to train the oligo(dT) binding-affinity MLP in mini-batches
    and evaluate classification accuracy on training, hold-out, and test windows
  repo: github.com/COMBINE-lab/forseti
- name: Python
  role: Implementation language of the Forseti package; the forseti package is imported
    to train and score the binding-affinity model
  repo: github.com/COMBINE-lab/forseti
- name: Nextflow
  role: Runs the forseti-experiments pipeline (mode fit_models) that fits and evaluates
    the binding-affinity model and fragment length distribution
  repo: https://github.com/COMBINE-lab/forseti-experiments
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae207
    title: '<tt>Forseti</tt> : a mechanistic and predictive model of the splicing
      status of scRNA-seq reads'
  source_content_hash: sha256:75dab525092f5f1a26c1e0c09e0aac5ec7a430ba8bb08b3e9a18eec9d3d02a0c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Forseti
  - Nextflow
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# hold-out-classification-accuracy-evaluation

## Summary

Trains the Forseti oligo(dT) binding-affinity MLP on labeled priming windows from scRNA-seq training datasets and evaluates generalization via per-dataset 1000-example hold-out sets and two held-out test datasets, reporting mean classification accuracies (~0.83 train, ~0.84 hold-out, ~0.89 test).

## When to use

Use when training or validating the Forseti oligo(dT) binding-affinity MLP on labeled priming windows and you need held-out generalization accuracies on training, hold-out, and test scRNA-seq datasets.

## When NOT to use

- The goal is predicting splicing status of scRNA-seq reads on a new dataset rather than training/evaluating the binding-affinity model — use the full Forseti prediction workflow (spliceu index, STAR alignment, forseti_predictor) instead
- Primed transcripts lack labeled priming windows with a valid adenine single-nucleotide repeat of length >= 6, so no valid training or hold-out windows can be extracted
- You need splicing-status prediction quality metrics (e.g., AUC on simulated or experimental reads) rather than binding-affinity classification accuracy

## Inputs

- Labeled priming windows around candidate transcriptomic priming sites from the eight training paired-end scRNA-seq datasets (windows containing an aSNR of length >= 6)
- Per-dataset 1000-example hold-out splits of priming windows
- Two held-out test scRNA-seq datasets randomly selected from the 10 processed 10x Genomics datasets
- Trained binding-affinity model checkpoint (mlp_pa6with1mm.pkl) if re-evaluating rather than retraining

## Outputs

- Trained MLP binding-affinity model checkpoint saved as mlp_pa6with1mm.pkl in the pre_built_models directory
- Mean classification accuracies per evaluation set (train, hold-out, test) as reported in Sections 2.5 and 3.1
- Evaluation script outputs stored in the workflow_output directory of forseti-experiments

## How to apply

Extract labeled priming windows around candidate transcriptomic priming sites from the eight training scRNA-seq datasets, retaining only windows containing a valid adenine single-nucleotide repeat (aSNR) of length >= 6, matching the snr_min_size = 6 default and the 'pa6with1mm' model configuration. Reserve a 1000-example hold-out set from each training dataset and pool the remaining windows into batched training data. Train the MLP binding-affinity network with the Forseti Python package in mini-batches to predict the probability that a transcriptomic site is used in fragment generation, then evaluate classification accuracy on the full training windows, each per-dataset 1000-example hold-out set, and the two held-out test datasets. Save the checkpoint as mlp_pa6with1mm.pkl in the pre_built_models directory; the reproducible pipeline lives in the forseti-experiments Nextflow repo (mode fit_models), which also ships pre-computed outputs in workflow_output for validation.

## Related tools

- **Forseti** (Python package used to train the oligo(dT) binding-affinity MLP in mini-batches and evaluate classification accuracy on training, hold-out, and test windows) — github.com/COMBINE-lab/forseti
- **Python** (Implementation language of the Forseti package; the forseti package is imported to train and score the binding-affinity model) — github.com/COMBINE-lab/forseti
- **Nextflow** (Runs the forseti-experiments pipeline (mode fit_models) that fits and evaluates the binding-affinity model and fragment length distribution) — https://github.com/COMBINE-lab/forseti-experiments

## Examples

```
nextflow main.nf -resume --mode fit_models
```

## Evaluation signals

- Trained model should reproduce the mean accuracies from Sections 2.5 and 3.1: approximately 0.83 on training data, 0.84 on hold-out sets, and 0.89 on the two test datasets
- The trained model checkpoint must be saved as mlp_pa6with1mm.pkl in the pre_built_models directory
- Each training dataset must contribute exactly a 1000-example hold-out set before pooling the remaining windows for batched training
- Only priming windows containing a valid aSNR of length >= 6 (snr_min_size = 6, 'pa6with1mm' configuration) should enter training and evaluation
- forseti-experiments ships the trained binding-affinity model and evaluation outputs in workflow_output, so results can be diffed against those pre-computed artifacts

## Limitations

- The forseti-experiments pipeline requires ~3 TB of disk space and is designed for a SLURM cluster; SLURM settings in nextflow.config must be adapted (clusterOptions, memory, threads)
- Nextflow cannot verify that fetched FASTQ files are uncorrupted; if prepare_data fails, successfully processed SRR runs must be removed from input_files/sample_url_sheet.csv and the pipeline re-run
- Accuracies are dataset-dependent: evaluation uses two experimental test datasets randomly selected from the 10 processed 10x Genomics datasets, so re-drawing the split can shift reported means

## Evidence

- [other] Reserve a 1000-example hold-out set from each training dataset and pool the remaining windows into batched training data.: [evidence span withheld — anchor status: not_found]
- [other] Save the trained model checkpoint as mlp_pa6with1mm.pkl in the pre_built_models directory and record the per-set mean accuracies.: [evidence span withheld — anchor status: not_found]
- [readme] we use the emperical priming windows and fragment lenths obtained from the previous step to (1) train a multi-layer perceptron (MLP) using the priming windows: [evidence span withheld — anchor status: external_doc]
- [readme] we have included the trained binding affinity model, and the output of the evaluation scripts in the `workflow_output` directory: [evidence span withheld — anchor status: external_doc]
- [abstract] we train a binding affinity model to assign a probability that a given transcriptomic site is used in fragment generation: "we train a binding affinity model to assign a probability that a given transcriptomic site is used in fragment generation"
