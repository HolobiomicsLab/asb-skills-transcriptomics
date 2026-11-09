---
name: cell-ontology-label-mapping
description: Use when single-cell annotations are free-text or dataset-specific terms
  needing mapping to Cell Ontology terms, or when labels must be predicted for CL
  terms absent from the training set via the ontology graph.
when_to_use_negative:
- The annotation task only concerns cell types already seen in training and labels
  are already valid CL terms—standard classifiers suffice without ontology-graph score
  extension
- The cell types of interest are topologically identical siblings (same parents in
  the Cell Ontology graph) that OnClass cannot distinguish
- The cell types or states of interest are not represented in the Cell Ontology, which
  was not developed specifically for scRNA-seq
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: OnClass
  role: 'Performs the label mapping: NLP model maps free text to CL terms, EmbedCellTypes
    embeds the ontology, and scores are extended to unseen terms via the Cell Ontology
    graph'
  repo: https://github.com/wangshenguiuc/OnClass
- name: tensorflow 2.0
  role: Deep-learning backend for OnClass 1.2 model training and prediction
- name: Scanorama
  role: Optional batch correction of expression matrices prior to ProcessTrainFeature
    feature processing
  repo: https://github.com/brianhie/scanorama
- name: Python 3.6
  role: Required runtime for OnClass 1.2
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-25725-x
    title: Leveraging the Cell Ontology to classify unseen cell types
  source_content_hash: sha256:2ebb499e22c360cfd4542c8a4d20511260a5641a6ab89ea0f426afc2147cf4c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wangshenguiuc/OnClass
  tools:
  - OnClass
  - Python
  - Python 3.6
  - Scanorama
  - TensorFlow
  - tensorflow 2.0
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# cell-ontology-label-mapping

## Summary

Maps single-cell cell-type labels—free text or dataset-specific terms—onto the Cell Ontology (CL) controlled vocabulary, and classifies cells to CL terms (including terms with no annotated cells in training) using OnClass, which embeds the ontology and transcriptomes into a shared low-dimensional space.

## When to use

Use when single-cell annotations are free-text or dataset-specific terms needing mapping to Cell Ontology terms, or when labels must be predicted for CL terms absent from the training set via the ontology graph.

## When NOT to use

- The annotation task only concerns cell types already seen in training and labels are already valid CL terms—standard classifiers suffice without ontology-graph score extension
- The cell types of interest are topologically identical siblings (same parents in the Cell Ontology graph) that OnClass cannot distinguish
- The cell types or states of interest are not represented in the Cell Ontology, which was not developed specifically for scRNA-seq

## Inputs

- cell-by-gene expression matrices (train_feature, test_feature) with matching gene lists (train_genes, test_genes)
- cell type labels per cell, either Cell Ontology term IDs (train_label) or free-text annotations
- Cell Ontology files: cl.obo (cl_obo_file), cell type network file (cell_type_network_file), and cl.ontology.nlp.emb NLP text embeddings

## Outputs

- cell labels assigned as Cell Ontology controlled-vocabulary terms, including unseen terms reached via the ontology graph
- probability/score vectors over all Cell Ontology categories per cell
- trained OnClass model file (save_model path) and low-dimensional cell type embeddings plus projected feature matrices (e.g. cell_type_embeddings.npz)

## How to apply

Load the ontology files with read_ontology_file('cell ontology', ontology_data_dir) to obtain cl_obo_file, cell_type_network_file, and the cl.ontology.nlp.emb text embeddings, then instantiate the OnClassModel with cell_type_nlp_emb_file and cell_type_network_file (OnClass 1.2, TensorFlow 2.0, Python 3.6). If the raw annotations are uncontrolled text, set nlp_mapping = True so the Char-level LSTM maps the uncontrolled vocabulary to controlled CL terms. Embed the seen cell types with OnClass_train_obj.EmbedCellTypes(train_label)—random walk with restart on the 'is_a' ontology graph (restart probabilities 0.5–0.8, clusDCA-style) followed by SVD—then project the cell-by-gene matrices into the same low-dimensional space via ProcessTrainFeature (optionally after Scanorama batch correction). Build and train the model (BuildModel with ngene = len(cor_train_genes); Train with save_model and max_iter = MAX_ITER); in its third step OnClass extends prediction scores to unseen cell types based on the Cell Ontology graph. A pre-trained model shipped with the package can annotate millions of cells in a few minutes when training from scratch is unnecessary.

## Related tools

- **OnClass** (Performs the label mapping: NLP model maps free text to CL terms, EmbedCellTypes embeds the ontology, and scores are extended to unseen terms via the Cell Ontology graph) — https://github.com/wangshenguiuc/OnClass
- **tensorflow 2.0** (Deep-learning backend for OnClass 1.2 model training and prediction)
- **Scanorama** (Optional batch correction of expression matrices prior to ProcessTrainFeature feature processing) — https://github.com/brianhie/scanorama
- **Python 3.6** (Required runtime for OnClass 1.2)

## Examples

```
onclass_train_obj = OnClassModel(cell_type_nlp_emb_file, cell_type_network_file); onclass_train_obj.EmbedCellTypes(train_label)
```

## Evaluation signals

- Every Cell Ontology ID present in train_label receives a finite, nonzero embedding row from EmbedCellTypes
- Projected feature matrices from ProcessTrainFeature/ProcessTestFeature share the same embedding dimensionality as the cell type embedding matrix
- All output labels are valid terms of the Cell Ontology controlled vocabulary; with nlp_mapping = True, uncontrolled free-text labels are converted to controlled CL terms
- Unseen CL terms (no annotated cells in training) receive nonzero scores after the ontology-graph score-extension step
- When using the shipped pre-trained model, annotation of millions of cells completes in minutes on a modern server

## Limitations

- OnClass cannot distinguish topologically identical unseen cell types, such as siblings with the same parents in the Cell Ontology graph
- The Cell Ontology was not developed specifically for scRNA-seq and therefore likely misses cell types and cell states
- Methods to correct batch effects in the presence of many unseen cell types are not yet developed

## Evidence

- [abstract] Here, we present OnClass, an algorithm and accompanying software for automatically classifying cells into cell types that are part of the controlled vocabulary that forms the Cell Ontology.: "Here, we present OnClass, an algorithm and accompanying software for automatically classifying cells into cell types that are part of the controlled vocabulary that forms the Cell Ontology."
- [abstract] A key advantage of OnClass is its capability to classify cells into cell types not present in the training data because it uses the Cell Ontology graph to infer cell type relationships.: "A key advantage of OnClass is its capability to classify cells into cell types not present in the training data because it uses the Cell Ontology graph to infer cell type relationships."
- [methods] Set nlp_mapping = True to use the Char-level LSTM that maps uncontrolled vocabulary to controlled vocabulary.: "Read training and test data. Set nlp_mapping = True to use the Char-level LSTM that maps uncontrolled vocabulary to controlled vocabulary."
- [discussion] Here we already offer an NLP model that is able to map free text annotations to Cell Ontology terms: "Here we already offer an NLP model that is able to map free text annotations to Cell Ontology terms"
- [discussion] OnClass will then perform the third step to extend these scores to unseen cell types based on the Cell Ontology graph.: "OnClass will then perform the third step to extend these scores to unseen cell types based on the Cell Ontology graph."
- [discussion] Another limitation of OnClass is that it cannot distinguish topologically identical unseen cell types, such as siblings with the same parents in the Cell Ontology graph.: "Another limitation of OnClass is that it cannot distinguish topologically identical unseen cell types, such as siblings with the same parents in the Cell Ontology graph."
- [readme] Update on July 4th, 2021. OnClass is now upgraded to 1.2 version!: "Update on July 4th, 2021. OnClass is now upgraded to 1.2 version!"
