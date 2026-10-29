---
name: model-evaluation-metric-computation
description: 'Use when a trained cell-type transfer model (e.g. TACTiCS) must be scored:
  build the per-cell-type matching score matrix between species and check reproduced
  recall values against published metrics.'
when_to_use_negative:
- Input is raw unprocessed scRNA-seq data rather than model outputs — preprocess and
  run TACTiCS classification/label propagation first
- Goal is gene similarity scoring (ProtBERT embeddings vs BLAST) rather than cell-type
  matching evaluation
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: TACTiCS
  role: Runs within-species subclass classification and cross-species transfer-learning
    label propagation whose outputs are scored; TACTiCS.save persists cell embeddings
  repo: https://github.com/kbiharie/TACTiCS
- name: PyTorch
  role: Backend required to train and evaluate the TACTiCS models being scored
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad248
    title: Cell type matching across species using protein embeddings and transfer
      learning
  source_content_hash: sha256:8e68d9889784771e95ca026605f9aa229001a11f72c230c68ab54d3c2106d8bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kbiharie/TACTiCS
  tools:
  - PyTorch
  - TACTiCS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# model-evaluation-metric-computation

## Summary

Compute and verify cell-type matching evaluation metrics for cross-species label transfer (TACTiCS): per-cell-type matching score matrices, within-species classification recall, and cross-species-resolution recall on Allen Brain M1 scRNA-seq data.

## When to use

Use when a trained cell-type transfer model (e.g. TACTiCS) must be scored: build the per-cell-type matching score matrix between species and check reproduced recall values against published metrics.

## When NOT to use

- Input is raw unprocessed scRNA-seq data rather than model outputs — preprocess and run TACTiCS classification/label propagation first
- Goal is gene similarity scoring (ProtBERT embeddings vs BLAST) rather than cell-type matching evaluation

## Inputs

- TACTiCS cell-type matching score matrix (mouse vs human subclasses) from transfer-learning label propagation
- Within-species subclass classification predictions from the restored pretrained TACTiCS models
- Expected reference metrics from the publication (subclass recall, cross-species recall, per-pair scores)

## Outputs

- Per-cell-type matching score matrix between mouse and human subclasses
- subclass_matching_results.csv table of matching scores
- Within-species recall and cross-species-resolution recall values for comparison against published metrics

## How to apply

After restoring pretrained TACTiCS models and running within-species subclass classification and cross-species transfer-learning label propagation on the Allen Brain M1 50k-subset data, compute the per-cell-type matching score matrix between mouse and human subclasses. Verify within-species classification recall (expected 1.0) and cross-species-resolution recall (expected 0.96). Inspect specific expected score pairs: mouse Peri–human VLMC 0.5, human VLMC–mouse VLMC 1.0, and species-specific pair scores between 0.04 and 0.5. At subclass resolution, confirm the correct cell-type match is returned for all 23 cell types shared between human and mouse. Persist the matching score table (e.g. subclass_matching_results.csv) so metrics are auditable against the published values.

## Related tools

- **TACTiCS** (Runs within-species subclass classification and cross-species transfer-learning label propagation whose outputs are scored; TACTiCS.save persists cell embeddings) — https://github.com/kbiharie/TACTiCS
- **PyTorch** (Backend required to train and evaluate the TACTiCS models being scored)

## Examples

```
from tactics import TACTiCS; model = TACTiCS.restore(pretrained); scores = model.match_subclasses(mouse, human); scores.to_csv('subclass_matching_results.csv'); model.save()
```

## Evaluation signals

- Within-species subclass classification recall equals 1.0
- Cross-species-resolution recall equals 0.96
- Correct subclass match returned for all 23 cell types shared between human and mouse
- Mouse Peri–human VLMC score is 0.5 and human VLMC–mouse VLMC score is 1.0
- Species-specific pair scores fall between 0.04 and 0.5

## Limitations

- Expected recall and score values are specific to the Allen Brain M1 human–mouse 50k-subset benchmark; other species pairs or resolutions may differ
- Full unprocessed dataset (Zenodo 10.5281/zenodo.6786357) differs from the 50k subsets with pretrained models (10.5281/zenodo.7582460), so metrics are not directly comparable across deposits

## Evidence

- [results] At the subclass resolution, TACTiCS correctly returns the cell type match for all 23 cell types shared between human and mouse.: [evidence span withheld — anchor status: not_found]
- [results] Compute the per-cell-type matching score matrix between mouse and human subclasses.: [evidence span withheld — anchor status: not_found]
- [methods] TACTiCS requires PyTorch.: "TACTiCS requires PyTorch. The transformers package is only necessary for embedding protein sequences."
- [readme] The preprocessed data, trained model and resulting cell embeddings can be saved using `TACTiCS.save`.: "The preprocessed data, trained model and resulting cell embeddings can be saved using `TACTiCS.save`."
