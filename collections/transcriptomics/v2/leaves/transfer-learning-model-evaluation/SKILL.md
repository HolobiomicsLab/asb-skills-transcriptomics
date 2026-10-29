---
name: transfer-learning-model-evaluation
description: Use when you have two cross-species scRNA-seq datasets (e.g. human, mouse,
  marmoset motor cortex), a gene similarity distance matrix, and trained TACTiCS models,
  and need to quantify cell type matching performance via ADS and recall.
when_to_use_negative:
- Input is a single-species dataset with no cross-species comparison goal — TACTiCS
  transfer learning propagates labels between species and is not a within-species
  annotation tool
- You lack a gene similarity distance matrix (or protein sequences/embeddings to build
  one with embed_proteins and calc_dist), since TACTiCS requires it as mandatory input
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: TACTiCS
  role: 'Transfer-learning model: classify cell types within a species with a neural
    network and propagate labels across species; train/evaluate via the TACTiCS class
    in tactics.py and score with ADS and recall'
  repo: https://github.com/kbiharie/TACTiCS
- name: PyTorch
  role: Deep learning framework required to train and run the TACTiCS model
- name: SAMap
  role: State-of-the-art baseline for cross-species cell type matching benchmarked
    at both label resolutions
- name: Seurat
  role: Baseline method using BLAST one-to-one gene matches for cross-species alignment
    in the benchmark
- name: BLAST
  role: Provides one-to-one and many-to-many gene match conditions for the gene-matching
    ablation distance matrices
- name: transformers
  role: Hugging Face package used to embed protein sequences with ProtBERT for gene
    matching
provenance:
  source_task_ids:
  - task_004
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad248
    title: Cell type matching across species using protein embeddings and transfer
      learning
  source_content_hash: sha256:0ed48eb4ef9373e78c95f06d46017d581323486be95206b44c640fbaa9cb1ffd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kbiharie/TACTiCS
  tools:
  - BLAST
  - PyTorch
  - SAMap
  - Seurat
  - TACTiCS
  - transformers
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# Transfer learning model evaluation

## Summary

Evaluating the TACTiCS transfer-learning cell type matching model by running the TACTiCS class (tactics.py) under PyTorch on cross-species scRNA-seq datasets and scoring predictions with average dimensional score (ADS) and recall, including benchmarking against SAMap and Seurat and gene-matching ablations (ProtBERT vs BLAST).

## When to use

Use when you have two cross-species scRNA-seq datasets (e.g. human, mouse, marmoset motor cortex), a gene similarity distance matrix, and trained TACTiCS models, and need to quantify cell type matching performance via ADS and recall.

## When NOT to use

- Input is a single-species dataset with no cross-species comparison goal — TACTiCS transfer learning propagates labels between species and is not a within-species annotation tool
- You lack a gene similarity distance matrix (or protein sequences/embeddings to build one with embed_proteins and calc_dist), since TACTiCS requires it as mandatory input

## Inputs

- Two cross-species scRNA-seq count matrices (50k-subset primary motor cortex data for human, mouse, marmoset)
- Gene similarity distance matrix (calc_dist output from ProtBERT or BLAST-derived protein embeddings)
- Trained TACTiCS models and protein embeddings from Zenodo (doi 10.5281/zenodo.7582460)
- BLAST one-to-one / many-to-many gene match matrices for ablation conditions

## Outputs

- ADS and recall values per cell type per species pair at subclass and cross-species resolutions
- Benchmark table (ads_recall_benchmark.csv) comparing TACTiCS, SAMap and Seurat
- Gene-matching ablation metrics (gene_matching_ablation_metrics.csv)
- Saved trained model and cell embeddings via TACTiCS.save

## How to apply

Download the preprocessed 50k-subset count data, protein embeddings and trained models from Zenodo (doi 10.5281/zenodo.7582460). Train and evaluate TACTiCS using the TACTiCS class in tactics.py (requires PyTorch), providing two scRNA-seq datasets and a gene similarity distance matrix computed with calc_dist from embeddings created with embed_proteins; save models and cell embeddings with TACTiCS.save. Run TACTiCS per species pair at both subclass and cross-species label resolutions, record ADS and recall per cell type, and benchmark against SAMap and Seurat (BLAST one-to-one gene matches). For the gene-matching ablation, rebuild the distance matrix under BLAST many-to-many and BLAST one-to-one conditions and retrain, checking that ADS/recall decrease relative to ProtBERT-based matching. Assemble per-cell-type scores (e.g. the L6b contrast with SAMap) into a benchmark table such as ads_recall_benchmark.csv or gene_matching_ablation_metrics.csv.

## Related tools

- **TACTiCS** (Transfer-learning model: classify cell types within a species with a neural network and propagate labels across species; train/evaluate via the TACTiCS class in tactics.py and score with ADS and recall) — https://github.com/kbiharie/TACTiCS
- **PyTorch** (Deep learning framework required to train and run the TACTiCS model)
- **SAMap** (State-of-the-art baseline for cross-species cell type matching benchmarked at both label resolutions)
- **Seurat** (Baseline method using BLAST one-to-one gene matches for cross-species alignment in the benchmark)
- **BLAST** (Provides one-to-one and many-to-many gene match conditions for the gene-matching ablation distance matrices)
- **transformers** (Hugging Face package used to embed protein sequences with ProtBERT for gene matching)

## Examples

```
from tactics import TACTiCS; model = TACTiCS(counts_a, counts_b, gene_sim_dist_matrix); model.save('trained_model')
```

## Evaluation signals

- TACTiCS should achieve higher ADS and recall than SAMap and Seurat at subclass resolution across all three species pairs (human–mouse, human–marmoset, mouse–marmoset)
- Replacing ProtBERT matches with BLAST matches should substantially decrease TACTiCS ADS and recall at the cross-species resolution
- In the SAMap ProtBERT condition, recall is expected to drop from 0.8 to 0.73 with other metrics similar to baseline
- Saved outputs (model, cell embeddings) must round-trip via TACTiCS.save and be reloadable for evaluation
- Per-cell-type scores should reproduce known contrasts, e.g. the L6b difference between TACTiCS and SAMap

## Limitations

- Evaluation is grounded on 50k subsets of the primary motor cortex datasets for human, mouse and marmoset; results may not generalize to other tissues or species pairs
- Mixed one-to-one BLAST effects across species pairs complicate interpretation of gene-matching ablations
- TACTiCS requires PyTorch, and the transformers package is needed only for embedding protein sequences — environments without these cannot run the pipeline
- No changelog is available for the kbiharie/TACTiCS repository, so code-version drift is untracked

## Evidence

- [methods] TACTiCS class usage: "To train and evaluate TACTiCS, use the `TACTiCS` class in `tactics.py`."
- [methods] Input requirements: "As input, `TACTiCS` requires two scRNA-seq datasets and a distance matrix describing the gene similarities."
- [abstract] Benchmark outcome: "our model outperforms Seurat and the state-of-the-art method SAMap"
- [abstract] Gene matching finding: "our gene matching method results in better cell type matches than BLAST in our model"
- [readme] Data availability: "the protein sequences, protein embeddings, raw count data and trained models for 50k subsets of the data can be found on [Zenodo](https://zenodo.org/doi/10.5281/zenodo.7582460)"
- [readme] Saving outputs: "The preprocessed data, trained model and resulting cell embeddings can be saved using `TACTiCS.save`."
