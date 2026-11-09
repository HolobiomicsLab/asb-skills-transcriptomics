---
name: ads-recall-metric-computation
description: Use when you need to score and compare cross-species cell type matching
  outputs (e.g., human–mouse, human–marmoset, mouse–marmoset motor cortex) with per-cell-type
  ADS and recall metrics, as in the TACTiCS benchmark against SAMap and Seurat.
when_to_use_negative:
- The goal is gene matching itself (e.g., comparing ProtBERT-based gene similarity
  to BLAST) rather than scoring the resulting cell type matches
- Only a single method is being run and no cross-method benchmark comparison is needed
- The input is within-species cell type annotation only, with no cross-species matching
  to score
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: TACTiCS
  role: Generates cross-species cell type matches to be scored with ADS and recall;
    run via the TACTiCS class in tactics.py
  repo: https://github.com/kbiharie/TACTiCS
- name: SAMap
  role: Baseline method whose cross-species matches are scored with ADS and recall
    for benchmarking
- name: Seurat
  role: Baseline method using BLAST one-to-one gene matches whose matches are scored
    with ADS and recall
- name: BLAST
  role: Provides the one-to-one gene matches used by the Seurat baseline
- name: PyTorch
  role: Dependency required to run TACTiCS when producing matches to be scored
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad248
    title: Cell type matching across species using protein embeddings and transfer
      learning
  source_content_hash: sha256:e4bb9fe815be2b2ffddb52668f0a7607a7a1e1d721930b9b6d8853f677266cd1
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# ads-recall-metric-computation

## Summary

Computes the per-cell-type ADS (alignment/cell-type matching score) and recall used in the TACTiCS paper to quantify cross-species cell type matching quality, enabling head-to-head benchmarking of TACTiCS against SAMap and Seurat at subclass and cross-species label resolutions.

## When to use

Use when you need to score and compare cross-species cell type matching outputs (e.g., human–mouse, human–marmoset, mouse–marmoset motor cortex) with per-cell-type ADS and recall metrics, as in the TACTiCS benchmark against SAMap and Seurat.

## When NOT to use

- The goal is gene matching itself (e.g., comparing ProtBERT-based gene similarity to BLAST) rather than scoring the resulting cell type matches
- Only a single method is being run and no cross-method benchmark comparison is needed
- The input is within-species cell type annotation only, with no cross-species matching to score

## Inputs

- Preprocessed 50k-subset scRNA-seq count data for human, mouse, and marmoset primary motor cortex (Zenodo doi 10.5281/zenodo.7582460)
- Cell type matching outputs from TACTiCS, SAMap, and Seurat for each species pair at subclass and cross-species label resolutions
- Label definitions at subclass and cross-species resolutions for the three species

## Outputs

- Per-cell-type ADS and recall values for each method, species pair, and label resolution
- Benchmark results table (ads_recall_benchmark.csv) aggregating all ADS/recall values
- Per-cell-type contrasts, e.g., the L6b comparison between TACTiCS and SAMap

## How to apply

Run each matching method (TACTiCS via the `TACTiCS` class in tactics.py, SAMap, and Seurat with BLAST one-to-one gene matches) on the same species pair at both subclass and cross-species label resolutions, using the 50k-subset scRNA-seq count data from Zenodo (doi 10.5281/zenodo.7582460). For each method and resolution, compute per-cell-type ADS and recall, including contrasts such as the L6b comparison between TACTiCS and SAMap. Assemble all ADS and recall values into a single benchmark table (saved as ads_recall_benchmark.csv) so methods can be compared like-for-like on identical inputs. The expected outcome, per the paper, is that TACTiCS achieves higher ADS and recall than both SAMap and Seurat at the subclass resolution across all three pairwise species comparisons.

## Related tools

- **TACTiCS** (Generates cross-species cell type matches to be scored with ADS and recall; run via the TACTiCS class in tactics.py) — https://github.com/kbiharie/TACTiCS
- **SAMap** (Baseline method whose cross-species matches are scored with ADS and recall for benchmarking)
- **Seurat** (Baseline method using BLAST one-to-one gene matches whose matches are scored with ADS and recall)
- **BLAST** (Provides the one-to-one gene matches used by the Seurat baseline)
- **PyTorch** (Dependency required to run TACTiCS when producing matches to be scored)

## Examples

```
from tactics import TACTiCS; model = TACTiCS(scrna_human, scrna_mouse, gene_dist_matrix)
```

## Evaluation signals

- All ADS/recall values for all three methods, three species pairs, and two label resolutions are present in ads_recall_benchmark.csv with no missing cells
- Per-cell-type scores are computed at the individual cell-type level, including the L6b contrast between TACTiCS and SAMap
- Replication check: TACTiCS should show higher ADS and recall than both SAMap and Seurat at the subclass resolution across all three pairwise species comparisons
- Each method is evaluated on the same 50k-subset inputs (Zenodo doi 10.5281/zenodo.7582460) so scores are comparable across methods

## Limitations

- Benchmark scope is restricted to the primary motor cortex scRNA-seq datasets of human, mouse, and marmoset (50k subsets); scores may not generalize to other tissues or species pairs
- Comparisons are resolution-dependent: the reported superiority of TACTiCS holds at the subclass resolution, and results must be reported separately for subclass and cross-species label resolutions
- The ProtBERT-based gene matching in TACTiCS outperformed BLAST within TACTiCS, so Seurat/BLAST baseline scores partly reflect gene-matching quality rather than the matching algorithm alone

## Evidence

- [abstract] TACTiCS outperforms Seurat and SAMap: "our model outperforms Seurat and the state-of-the-art method SAMap"
- [methods] TACTiCS run interface: "To train and evaluate TACTiCS, use the `TACTiCS` class in `tactics.py`."
- [readme] Benchmark inputs: "the protein sequences, protein embeddings, raw count data and trained models for 50k subsets of the data can be found on [Zenodo]"
- [abstract] Gene matching baseline: "our gene matching method results in better cell type matches than BLAST in our model"
