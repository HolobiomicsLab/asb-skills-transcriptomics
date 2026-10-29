---
name: umap-embedding-visualization
description: Use when a trained autoBioSeqpy/Keras model (tmpMod., tmpWeight.bin,
  parameters.txt) must be validated or diagnosed by visualizing its hidden-layer activations
  as a 2D UMAP scatter plot colored by class labels.
when_to_use_negative:
- The model architecture contains branches — layerUMAP cannot capture hidden outputs
  from layers in a branch of the built model; outputs must be captured manually via
  the related Jupyter Notebook instead.
- No trained autoBioSeqpy model artifacts (model file, weights, parameters.txt) exist
  yet — run autoBioSeqpy training first, as data encoding during prediction depends
  on the training parameters.
- The goal is quantitative model evaluation (accuracy, loss) rather than visual inspection
  of learned representations — running.py/predicting.py with --savePrediction is the
  appropriate path.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: layerUMAP
  role: Command-line tool that captures hidden-layer activations of an autoBioSeqpy-trained
    model and projects them into 2D UMAP space, with grid-searchable n_neighbors/min_dist,
    interactive layer selection, and saveFig output
  repo: github.com/jingry/autoBioSeqpy
- name: autoBioSeqpy
  role: Encodes biological sequences (protein/DNA/RNA FASTA) into matrices, trains
    the DL models, and saves the model (tmpMod.), weights (tmpWeight.bin), and parameters
    (parameters.txt) that layerUMAP loads
  repo: github.com/jingry/autoBioSeqpy
- name: UMAP
  role: Library used by layerUMAP to compute the two-dimensional embedding of the
    captured hidden-layer representations
provenance:
  source_task_ids:
  - task_006
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2022.105530
    title: 'layerUMAP: A tool for visualizing and understanding deep learning models
      in biological sequence classification using UMAP'
  source_content_hash: sha256:e9daf5c946c0cba0e89c949abc2839c308b8ef31f99ac80f4b28e1b27c81500d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/jingry/autoBioSeqpy
  tools:
  - UMAP
  - autoBioSeqpy
  - layerUMAP
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# umap-embedding-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

layerUMAP, a command-line tool included in autoBioSeqpy, projects hidden-layer representations of trained deep learning models (CNN, BiLSTM/BiGRU, RFCN) into two-dimensional UMAP space to visually validate classification of biological sequences and single-cell gene expression data.

## When to use

Use when a trained autoBioSeqpy/Keras model (tmpMod., tmpWeight.bin, parameters.txt) must be validated or diagnosed by visualizing its hidden-layer activations as a 2D UMAP scatter plot colored by class labels.

## When NOT to use

- The model architecture contains branches — layerUMAP cannot capture hidden outputs from layers in a branch of the built model; outputs must be captured manually via the related Jupyter Notebook instead.
- No trained autoBioSeqpy model artifacts (model file, weights, parameters.txt) exist yet — run autoBioSeqpy training first, as data encoding during prediction depends on the training parameters.
- The goal is quantitative model evaluation (accuracy, loss) rather than visual inspection of learned representations — running.py/predicting.py with --savePrediction is the appropriate path.

## Inputs

- Trained Keras model file (tmpMod.) with weights (tmpWeight.bin) and training parameters (tmpOut/parameters.txt) saved by autoBioSeqpy
- Test FASTA sequences (e.g. examples/typeIII_secreted_effectors_prediction/data/test_pos.txt and test_neg.txt; dataType protein, dataEncodingType dict, spcLen 100)
- Single-cell gene expression data (e.g. Zheng sorted dataset) with cell ontology labels from the original publication

## Outputs

- Two-dimensional UMAP scatter plot of hidden-layer representations (e.g. umap_last_hidden_layer.png) with points colored by true class labels
- Interactive layer-by-layer visualization of hidden-layer outputs along the depth of the model
- Grid of UMAP embeddings across distance metrics (chebyshev, braycurtis, canberra, correlation, cosine, euclidean, mahalanobis, manhattan, minkowski, wminkowski) for metric comparison

## How to apply

First train or load the model with autoBioSeqpy (e.g. running.py with --dataType protein, --dataEncodingType dict, --spcLen 100) so that the model file tmpMod., weights tmpWeight.bin, and the training-record file tmpOut/parameters.txt are saved — these are the files layerUMAP consumes. Invoke layerUMAP as a command-line tool, feeding the test FASTA sequences (e.g. test_pos.txt and test_neg.txt) or single-cell gene expression data (e.g. Zheng sorted dataset) through the network to capture activations of the chosen hidden layer, including a layer-by-layer dissection option that walks the depth of the model interactively. Grid-search the UMAP parameters n_neighbors and min_dist to obtain clear cluster separation; note that on the Zheng sorted dataset the 'min_dist' parameter had a significant impact on the separation of cell populations. Render the 2D embedding as a point-cloud figure colored by true class labels (e.g. the six secretory types T1SS–T7SS, or cell ontology labels), with --saveFig enabled writing to the designated output folder (e.g. umap_last_hidden_layer.png). Judge success by expert visual review: the classes the network was trained to distinguish should form visually distinct clusters consistent with the reported classification accuracy (e.g. RFCN 98.09% on Zheng sorted).

## Related tools

- **layerUMAP** (Command-line tool that captures hidden-layer activations of an autoBioSeqpy-trained model and projects them into 2D UMAP space, with grid-searchable n_neighbors/min_dist, interactive layer selection, and saveFig output) — github.com/jingry/autoBioSeqpy
- **autoBioSeqpy** (Encodes biological sequences (protein/DNA/RNA FASTA) into matrices, trains the DL models, and saves the model (tmpMod.), weights (tmpWeight.bin), and parameters (parameters.txt) that layerUMAP loads) — github.com/jingry/autoBioSeqpy
- **UMAP** (Library used by layerUMAP to compute the two-dimensional embedding of the captured hidden-layer representations)

## Examples

```
python running.py --dataType protein --dataEncodingType dict --dataTrainFilePaths examples/typeIII_secreted_effectors_prediction/data/train_pos.txt examples/typeIII_secreted_effectors_prediction/data/train_neg.txt --modelLoadFile examples/typeIII_secreted_effectors_prediction/model/protein_CNN_model.py --outSaveFolderPath tmpOut --saveFig 1 --spcLen 100 --modelSaveName tmpMod. --weightSaveName tmpWeight.bin --paraSaveName parameters.txt
```

## Evaluation signals

- Output figure (e.g. umap_last_hidden_layer.png) exists in the designated --outSaveFolderPath when --saveFig 1 is set
- Expert visual review confirms the six secretory types (T1SS–T7SS) form distinguishable clusters in the 2D projection, consistent with the high classification accuracy reported for the best CNN model
- In the ten-metric comparison, nine distance metrics yield visually distinct cell-population clusters while only wminkowski yields mixed, non-separated populations
- Grid search over n_neighbors and min_dist shows sensitivity of cluster separation to min_dist (demonstrated on the Zheng sorted dataset), so tuned values should be reported
- Layer-by-layer dissection produces progressively more interpretable embeddings as model depth increases, consistent with the interactive depth option

## Limitations

- layerUMAP cannot capture hidden outputs from layers in a branch (if existing) of the built model; users must capture branch-layer outputs manually using the related Jupyter Notebook
- The actual text of command-line commands 1–5 is only referenced via Figure 2, not given inline in the article (reproducibility gap)
- The UMAP parameters n_neighbors and min_dist must be tuned per dataset — min_dist had a significant impact on separation of cell populations on the Zheng sorted dataset
- Reported runtimes are approximate and hardware-dependent (e.g. approximately 30 s per command, excluding interaction time)
- No changelog found in the source repository content

## Evidence
