---
name: biological-sequence-umap-visualization
description: Use when a trained Keras model (tmpMod./tmpWeight.bin from autoBioSeqpy,
  parameters.txt) must be inspected by embedding its hidden-layer activations for
  encoded protein/DNA/RNA test sequences in 2D UMAP space.
when_to_use_negative:
- The model contains layers inside a branch — layerUMAP cannot capture hidden outputs
  from branched layers; capture them manually via Jupyter Notebook instead
- No trained model exists yet — use autoBioSeqpy's running.py for training/prediction
  first, then apply layerUMAP
- The goal is classification accuracy rather than representation inspection — use
  the model's test metrics, not UMAP embeddings
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0080
tools:
- name: layerUMAP
  role: Command-line tool that loads the trained autoBioSeqpy model, captures hidden-layer
    activations, and projects them with UMAP
  repo: github.com/jingry/autoBioSeqpy
- name: autoBioSeqpy
  role: Loads model architecture (tmpMod.), weights (tmpWeight.bin), parameters.txt,
    and encodes protein sequences with dict encoding at spcLen 100
  repo: github.com/jingry/autoBioSeqpy
- name: UMAP
  role: Dimensionality-reduction library that embeds each hidden layer's representation
    into 2D, with tunable n_neighbors and min_dist
  repo: github.com/jingry/autoBioSeqpy
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2022.105530
    title: 'layerUMAP: A tool for visualizing and understanding deep learning models
      in biological sequence classification using UMAP'
  source_content_hash: sha256:2f1f945680dc7faae45fed912c4addf6f4c3574569ccdfb2fab8b76160d62c64
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# biological-sequence-umap-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

layerUMAP, a command-line tool in autoBioSeqpy, projects hidden-layer activations of trained deep learning models onto 2D UMAP embeddings to visualize and dissect how biological sequence representations organize across model depth.

## When to use

Use when a trained Keras model (tmpMod./tmpWeight.bin from autoBioSeqpy, parameters.txt) must be inspected by embedding its hidden-layer activations for encoded protein/DNA/RNA test sequences in 2D UMAP space.

## When NOT to use

- The model contains layers inside a branch — layerUMAP cannot capture hidden outputs from branched layers; capture them manually via Jupyter Notebook instead
- No trained model exists yet — use autoBioSeqpy's running.py for training/prediction first, then apply layerUMAP
- The goal is classification accuracy rather than representation inspection — use the model's test metrics, not UMAP embeddings

## Inputs

- Trained keras model file (e.g. tmpMod. saved by running.py)
- Model weights file (tmpWeight.bin)
- Training parameter record (parameters.txt)
- Test FASTA sequences (e.g. test_pos.txt and test_neg.txt for typeIII secreted effectors)
- Encoding settings: --dataType protein, --dataEncodingType dict, --spcLen 100

## Outputs

- Per-hidden-layer 2D UMAP point-cloud PNGs colored by true class labels (e.g. six secretory types T1SS-T7SS)
- Interactive inter-layer evolution plot showing cluster separation along model depth
- Convolution-output views reduced over 'Filtered sequence length' and 'Channel' directions (max/min/avg)
- umap_last_hidden_layer.png in the --outSaveFolderPath directory

## How to apply

Load the trained keras model architecture, its tmpWeight.bin weights, and parameters.txt through autoBioSeqpy together with test FASTA sequences encoded with dict encoding at spcLen 100. Run layerUMAP to capture the outputs of each hidden layer, apply UMAP to each layer's representation, and assemble the interactive inter-layer evolution plot showing how class clusters organize along model depth. For the convolutional layer's output tensor, separately collapse the 'Filtered sequence length' and 'Channel' directions with max, min, and avg reductions and embed each reduced view with UMAP. Grid-search the UMAP parameters n_neighbors and min_dist via the tool's command-line interface (min_dist strongly impacts cluster separation). Write all figures with --saveFig 1 into the directory given by --outSaveFolderPath and confirm every dissected layer produced a non-empty PNG.

## Related tools

- **layerUMAP** (Command-line tool that loads the trained autoBioSeqpy model, captures hidden-layer activations, and projects them with UMAP) — github.com/jingry/autoBioSeqpy
- **autoBioSeqpy** (Loads model architecture (tmpMod.), weights (tmpWeight.bin), parameters.txt, and encodes protein sequences with dict encoding at spcLen 100) — github.com/jingry/autoBioSeqpy
- **UMAP** (Dimensionality-reduction library that embeds each hidden layer's representation into 2D, with tunable n_neighbors and min_dist) — github.com/jingry/autoBioSeqpy

## Examples

```
python running.py --paraFile parameters.txt
```

## Evaluation signals

- A UMAP PNG is produced for every dissected hidden layer and each saved file in --outSaveFolderPath is non-empty
- The six Gram-negative secretory types (T1SS-T7SS) form visually distinguishable clusters in the last-hidden-layer 2D embedding, consistent with reported CNN accuracy
- The inter-layer evolution plot reproduces progressively separated class clusters along model depth (Figure 2 of the layerUMAP paper)
- Grid-searched n_neighbors/min_dist values yield clear cluster separation (min_dist notably impacts separation of cell populations on Zheng sorted)
- Convolution tensor reductions over 'Filtered sequence length' and 'Channel' directions each yield a valid UMAP embedding

## Limitations

- layerUMAP cannot capture hidden outputs from layers inside a branch of the built model; users must capture these manually in Jupyter Notebook
- The exact text of command-line commands 1-5 is only referenced via Figure 2 and not given inline, complicating exact reproduction
- Reported runtimes (approximately 30 s per command) are approximate and hardware-dependent
- The UMAP parameter min_dist has a significant impact on the separation of cell populations, so results are sensitive to this parameter choice

## Evidence

- [abstract] layerUMAP integrates autoBioSeqpy and UMAP for hidden-layer visualization: "This new tool, namely, layerUMAP, integrates autoBioSeqpy software and the UMAP library to address learned high-level representations."
- [intro] layerUMAP is a Python command-line tool inside autoBioSeqpy: "We present layerUMAP, a command-line tool included in the autoBioSeqpy software, written in Python, which makes use of UMAP to understand, diagnose and refine DL models."
- [discussion] Grid search over UMAP parameters: "we first performed a grid search in the parameter space using the command-line command 4 (Figure 2)"
- [discussion] Branch-layer limitation: "Currently, layerUMAP cannot capture the hidden outputs from the layers in a branch (if existing) of the built model"
- [readme] Dependency installation via anaconda3: "users could resolve module dependencies by installing anaconda3 (2 is not suggested) on their official website"
