---
name: binding-affinity-mlp-training
description: Use when labeled oligo(dT) priming windows (paired-end alignments indicating
  primer sites) from scRNA-seq data are available and a sequence-based binding-affinity
  predictor must be trained or reproduced (e.g. the pa6with1mm model).
when_to_use_negative:
- Goal is predicting splicing status of full reads — that requires the fragment length
  distribution model in addition to the binding-affinity model (Forseti combines both)
- Primed sequencing protocol is not oligo(dT)-based (e.g. random priming), since the
  model learns oligo(dT) primer affinity specifically
- No labeled priming windows with a valid aSNR (length >= 6) can be extracted, as
  the training labels cannot be constructed
edam_operation: http://edamontology.org/operation_0253
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3308
tools:
- name: Forseti
  role: Python package whose MLP binding-affinity network is trained in mini-batches
    on priming windows; checkpoint saved as mlp_pa6with1mm.pkl
  repo: https://github.com/COMBINE-lab/forseti
- name: forseti-experiments
  role: Nextflow pipeline whose fit_models mode performs the MLP training and evaluation;
    includes trained model and evaluation outputs in workflow_output
  repo: https://github.com/COMBINE-lab/forseti-experiments
- name: Python
  role: Implementation language of the Forseti package used to train the binding-affinity
    model
  repo: https://github.com/COMBINE-lab/forseti
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae207
    title: '<tt>Forseti</tt> : a mechanistic and predictive model of the splicing
      status of scRNA-seq reads'
  source_content_hash: sha256:c958674edbac7faa50883dd1564f5de8f4dcb1f096cefb8beb0be56cd3ffb884
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/COMBINE-lab/forseti
  tools:
  - Forseti
  - Python
  - forseti-experiments
  edam_operation: http://edamontology.org/operation_0253
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# binding-affinity-mlp-training

## Summary

Trains a multi-layer perceptron (MLP) binding-affinity model that predicts the probability an oligo(dT) primer primed a given transcriptomic site, using labeled priming windows extracted from scRNA-seq datasets — one of the two trained components Forseti combines to score splicing status of reads.

## When to use

Use when labeled oligo(dT) priming windows (paired-end alignments indicating primer sites) from scRNA-seq data are available and a sequence-based binding-affinity predictor must be trained or reproduced (e.g. the pa6with1mm model).

## When NOT to use

- Goal is predicting splicing status of full reads — that requires the fragment length distribution model in addition to the binding-affinity model (Forseti combines both)
- Primed sequencing protocol is not oligo(dT)-based (e.g. random priming), since the model learns oligo(dT) primer affinity specifically
- No labeled priming windows with a valid aSNR (length >= 6) can be extracted, as the training labels cannot be constructed

## Inputs

- Labeled oligo(dT) priming windows extracted from paired-end alignments of eight training scRNA-seq datasets (positive windows containing aSNR >= 6, plus random background negatives)
- Per-dataset configuration with snr_min_size = 6 and model variant 'pa6with1mm' (single mismatch tolerance)
- forseti-experiments Nextflow pipeline state after the prepare_data and process_data modes

## Outputs

- Trained binding-affinity MLP checkpoint saved as mlp_pa6with1mm.pkl in the pre_built_models directory
- Mean classification accuracies per evaluation split: ~0.83 (training windows), ~0.84 (per-dataset 1000-example hold-out sets), ~0.89 (two held-out test datasets)

## How to apply

Extract labeled priming windows around candidate transcriptomic priming sites from the eight training scRNA-seq datasets, retaining only windows containing a valid adenine single-nucleotide repeat (aSNR) of length >= 6, consistent with the snr_min_size = 6 default and the 'pa6with1mm' model configuration. Reserve a 1000-example hold-out set from each training dataset and pool the remaining windows into mini-batched training data. Train the MLP binding-affinity network using the Forseti Python package to predict the probability that a given transcriptomic site was used in fragment generation. Evaluate classification accuracy on the full training windows, the per-dataset 1000-example hold-out sets, and the two held-out test datasets, and save the checkpoint as mlp_pa6with1mm.pkl in the pre_built_models directory. In the forseti-experiments pipeline this training corresponds to the 'fit_models' Nextflow mode, run after the 'process_data' step that computes fragment lengths and priming windows.

## Related tools

- **Forseti** (Python package whose MLP binding-affinity network is trained in mini-batches on priming windows; checkpoint saved as mlp_pa6with1mm.pkl) — https://github.com/COMBINE-lab/forseti
- **forseti-experiments** (Nextflow pipeline whose fit_models mode performs the MLP training and evaluation; includes trained model and evaluation outputs in workflow_output) — https://github.com/COMBINE-lab/forseti-experiments
- **Python** (Implementation language of the Forseti package used to train the binding-affinity model) — https://github.com/COMBINE-lab/forseti

## Examples

```
nextflow main.nf -resume --mode fit_models
```

## Evaluation signals

- Reproduced mean accuracies approximately 0.83 on training windows, 0.84 on hold-out sets, and 0.89 on the two test datasets (per Sections 2.5 and 3.1)
- Model checkpoint mlp_pa6with1mm.pkl exists in the pre_built_models directory
- Each training dataset contributed exactly a 1000-example hold-out set excluded from mini-batch training
- All retained training windows contain a valid aSNR of length >= 6 matching the pa6with1mm configuration

## Limitations

- The MLP predicts binding affinity only; a separate robust fragment length distribution model (fitted via cubic spline) is required to score putative fragments for splicing-status prediction
- Forseti cannot distinguish fundamentally ambiguous reads (polyA-tail priming with read2 in the terminal exon vs. fragment arising from an exon)
- Reproducing training requires ~3 TB of disk space for the forseti-experiments pipeline, and corrupted FASTQ fetches can fail the prepare_data step

## Evidence

- [intro] First, we train a binding affinity model to assign a probability that a given transcriptomic site is used in fragment generation: "First, we train a binding affinity model to assign a probability that a given transcriptomic site is used in fragment generation"
- [readme] MLP trained on priming windows (README): [evidence span withheld — anchor status: external_doc]
- [abstract] Forseti combines two trained models to score fragments: "Forseti combines these two trained models to predict the splicing status of the molecule of origin of reads by scoring putative fragments"
- [readme] Pre-built model checkpoint shipped with experiments repo (README): [evidence span withheld — anchor status: external_doc]
