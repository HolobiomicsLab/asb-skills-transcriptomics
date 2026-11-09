---
name: transfer-learning-classifier-training
description: 'Use when matching/aligning cell types across species from scRNA-seq
  data: you have two scRNA-seq datasets (e.g. human/mouse/marmoset motor cortex) and
  want to train a within-species classifier then propagate labels cross-species.'
when_to_use_negative:
- Gene matching only, without cell type label transfer — use BLAST or the protein-embedding
  gene-matching step alone instead of the full classifier/transfer-learning pipeline
- When you need to compare against a state-of-the-art baseline at matched dataset
  sizes, run SAMap on the same subsets rather than relying on TACTiCS training alone
- Input data is not scRNA-seq cell x gene count data (TACTiCS requires two scRNA-seq
  datasets plus a gene distance matrix)
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: TACTiCS
  role: Train and evaluate the within-species classifier and cross-species label transfer
    using the TACTiCS class in tactics.py; save model and cell embeddings with TACTiCS.save
  repo: https://github.com/kbiharie/TACTiCS
- name: PyTorch
  role: Required dependency for running TACTiCS neural network training/transfer learning
- name: transformers
  role: Used only to embed protein sequences (embed_proteins) that feed into calc_dist
    for the gene similarity matrix
- name: SAMap
  role: State-of-the-art baseline method; run on the same species pairs/subsets to
    compare cross-species matching performance
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad248
    title: Cell type matching across species using protein embeddings and transfer
      learning
  source_content_hash: sha256:21aa7d9851358b06bfa4d7e06bc16a07a10fb780059efaaaac215d4de3e30011
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kbiharie/TACTiCS
  tools:
  - PyTorch
  - SAMap
  - TACTiCS
  - transformers
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# transfer-learning-classifier-training

## Summary

Train a neural network classifier on cell types within one species (TACTiCS) and use transfer learning to propagate those labels to a second species, with gene similarity between species derived from ProtBERT-style protein-sequence embeddings rather than BLAST. The resulting trained model and cell embeddings can be saved with TACTiCS.save.

## When to use

Use when matching/aligning cell types across species from scRNA-seq data: you have two scRNA-seq datasets (e.g. human/mouse/marmoset motor cortex) and want to train a within-species classifier then propagate labels cross-species.

## When NOT to use

- Gene matching only, without cell type label transfer — use BLAST or the protein-embedding gene-matching step alone instead of the full classifier/transfer-learning pipeline
- When you need to compare against a state-of-the-art baseline at matched dataset sizes, run SAMap on the same subsets rather than relying on TACTiCS training alone
- Input data is not scRNA-seq cell x gene count data (TACTiCS requires two scRNA-seq datasets plus a gene distance matrix)

## Inputs

- Two scRNA-seq datasets (raw count data; e.g. 50k-cell subsets per species or full datasets of primary motor cortex from human, mouse, and marmoset)
- Gene similarity distance matrix computed by calc_dist from protein embeddings created with embed_proteins (genes.py); deposited embeddings available on Zenodo (10.5281/zenodo.7582460)

## Outputs

- Trained TACTiCS model (trainable/evaluable via the TACTiCS class in tactics.py)
- Preprocessed data and resulting cell embeddings, saved via TACTiCS.save
- Cross-species cell type matches/alignments (e.g. per-comparison scores such as tactics_vs_samap_50k_results.csv when benchmarked against SAMap)

## How to apply

First, build gene similarity from protein sequences: create protein-sequence embeddings with embed_proteins in genes.py (requires the transformers package), then compute the gene similarity distance matrix with calc_dist. Train and evaluate TACTiCS using the TACTiCS class in tactics.py, supplying the two scRNA-seq datasets and the gene distance matrix as input — TACTiCS requires PyTorch. The pipeline proceeds in three stages: match genes via the NLP protein-embedding model, classify cell types within a species with the neural network, then use transfer learning to propagate cell type labels between species. Save the preprocessed data, trained model and cell embeddings using TACTiCS.save. When benchmarking (e.g. against SAMap), score cell type matching at the higher (cross-species) resolution; note that on equal-sized 50k-cells-per-species subsets TACTiCS performance drops relative to full datasets but still outperforms SAMap at cross-species resolution.

## Related tools

- **TACTiCS** (Train and evaluate the within-species classifier and cross-species label transfer using the TACTiCS class in tactics.py; save model and cell embeddings with TACTiCS.save) — https://github.com/kbiharie/TACTiCS
- **PyTorch** (Required dependency for running TACTiCS neural network training/transfer learning)
- **transformers** (Used only to embed protein sequences (embed_proteins) that feed into calc_dist for the gene similarity matrix)
- **SAMap** (State-of-the-art baseline method; run on the same species pairs/subsets to compare cross-species matching performance)

## Examples

```
from tactics import TACTiCS; from genes import calc_dist; D = calc_dist(protein_embeddings); model = TACTiCS(dataset_a, dataset_b, D); model.save()
```

## Evaluation signals

- Within-species neural network classifies known cell types accurately before transfer; cross-species propagation then aligns cell types across human, mouse, and marmoset motor cortex datasets
- Benchmark at the higher (cross-species) resolution: TACTiCS should outperform SAMap in all three pairwise species comparisons, both on full data and on equal-sized 50k-cells-per-species subsets
- On 50k subsets, expect a quantifiable performance drop relative to full-dataset reference performance — record per-comparison scores alongside full-data references (tactics_vs_samap_50k_results.csv)
- Verify input requirements: TACTiCS must receive two scRNA-seq datasets plus a gene similarity distance matrix computed by calc_dist from protein embeddings
- Verify persisted artifacts: TACTiCS.save must yield preprocessed data, the trained model, and the resulting cell embeddings

## Limitations

- Performance degrades on reduced-size datasets: on equal-sized 50k-cells-per-species subsets, TACTiCS performance drops relative to full datasets (though it still beats SAMap at cross-species resolution)
- The transformers package is required only when regenerating protein-sequence embeddings; deposited embeddings on Zenodo (10.5281/zenodo.7582460) can be reused instead

## Evidence

- [abstract] transfer learning stage: "Afterward, TACTiCS uses transfer learning to propagate cell type labels between species"
- [methods] input requirements: "As input, `TACTiCS` requires two scRNA-seq datasets and a distance matrix describing the gene similarities."
- [methods] gene similarity computation: "`calc_dist` calculates the latter for embeddings created with `embed_proteins` in `genes.py`."
- [readme] persistence of outputs: "The preprocessed data, trained model and resulting cell embeddings can be saved using `TACTiCS.save`."
- [results] benchmark outcome on equal-sized subsets: [evidence span withheld — anchor status: not_found]
