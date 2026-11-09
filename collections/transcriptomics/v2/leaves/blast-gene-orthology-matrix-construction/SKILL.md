---
name: blast-gene-orthology-matrix-construction
description: 'Use when running a gene-matching ablation in cross-species scRNA-seq
  analysis: you need BLAST-derived gene similarity distance matrices to replace ProtBERT
  matches as input to TACTiCS on datasets such as human-mouse motor cortex.'
when_to_use_negative:
- When the goal is TACTiCS's best performance rather than an ablation — ProtBERT-based
  gene matching yields better cell type matches than BLAST, so BLAST matrices are
  the comparison baseline, not the default
- When running SAMap's baseline pipeline, which uses ProtBERT-derived gene matches
  rather than BLAST distance matrices
- When the datasets are not cross-species scRNA-seq pairs (TACTiCS requires two scRNA-seq
  datasets plus a gene similarity distance matrix)
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3170
tools:
- name: BLAST
  role: Produce the many-to-many and one-to-one gene matches that are converted into
    gene similarity distance matrices for the ablation conditions
- name: TACTiCS
  role: Consume the two scRNA-seq datasets plus the BLAST distance matrix; train/evaluate
    via the TACTiCS class in tactics.py and save with TACTiCS.save
  repo: https://github.com/kbiharie/TACTiCS
- name: PyTorch
  role: Framework required by TACTiCS for retraining and evaluation under each BLAST
    condition
- name: SAMap
  role: Comparison method re-run with ProtBERT-derived gene matches to benchmark against
    the BLAST-based TACTiCS conditions
- name: transformers
  role: Used to embed protein sequences for ProtBERT-derived gene matches in the ablation
    contrast
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad248
    title: Cell type matching across species using protein embeddings and transfer
      learning
  source_content_hash: sha256:cc1cc694f7d69ab1c62f00b768a54450d3663d30b28e436c3180842e3a115b38
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kbiharie/TACTiCS
  tools:
  - BLAST
  - PyTorch
  - SAMap
  - TACTiCS
  - transformers
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# blast-gene-orthology-matrix-construction

## Summary

Construct gene similarity distance matrices from BLAST-based gene matches (many-to-many or one-to-one orthology conditions) so they can serve as TACTiCS inputs for cross-species cell type matching, enabling ablation against ProtBERT-based gene matching.

## When to use

Use when running a gene-matching ablation in cross-species scRNA-seq analysis: you need BLAST-derived gene similarity distance matrices to replace ProtBERT matches as input to TACTiCS on datasets such as human-mouse motor cortex.

## When NOT to use

- When the goal is TACTiCS's best performance rather than an ablation — ProtBERT-based gene matching yields better cell type matches than BLAST, so BLAST matrices are the comparison baseline, not the default
- When running SAMap's baseline pipeline, which uses ProtBERT-derived gene matches rather than BLAST distance matrices
- When the datasets are not cross-species scRNA-seq pairs (TACTiCS requires two scRNA-seq datasets plus a gene similarity distance matrix)

## Inputs

- Raw scRNA-seq count data for human and mouse primary motor cortex (50k subsets)
- Protein sequences for the two species' genes
- BLAST alignment results (many-to-many and one-to-one gene-match conditions)
- Trained TACTiCS models from the Zenodo deposit 10.5281/zenodo.7582460

## Outputs

- Gene similarity distance matrix for the BLAST many-to-many condition
- Gene similarity distance matrix for the BLAST one-to-one condition
- Retrained TACTiCS models and cell embeddings per condition (via TACTiCS.save)
- gene_matching_ablation_metrics.csv with ADS and recall per gene-matching condition

## How to apply

Download the human and mouse motor cortex raw count data, protein embeddings and trained 50k-subset models from the Zenodo deposit 10.5281/zenodo.7582460. Run BLAST on protein sequences to obtain gene matches under two conditions — many-to-many and one-to-one — and encode each as a gene similarity distance matrix, the format TACTiCS requires alongside the two scRNA-seq datasets. Feed each BLAST matrix into the TACTiCS class in tactics.py under PyTorch, retrain, and save models/cell embeddings with TACTiCS.save. For the ablation contrast, re-run SAMap with ProtBERT-derived gene matches generated with the transformers package, then compute ADS and recall per condition. Expected result: BLAST many-to-many matches substantially decrease TACTiCS ADS and recall versus ProtBERT matches, confirming the protein-embedding gene matching is the stronger choice.

## Related tools

- **BLAST** (Produce the many-to-many and one-to-one gene matches that are converted into gene similarity distance matrices for the ablation conditions)
- **TACTiCS** (Consume the two scRNA-seq datasets plus the BLAST distance matrix; train/evaluate via the TACTiCS class in tactics.py and save with TACTiCS.save) — https://github.com/kbiharie/TACTiCS
- **PyTorch** (Framework required by TACTiCS for retraining and evaluation under each BLAST condition)
- **SAMap** (Comparison method re-run with ProtBERT-derived gene matches to benchmark against the BLAST-based TACTiCS conditions)
- **transformers** (Used to embed protein sequences for ProtBERT-derived gene matches in the ablation contrast)

## Examples

```
from tactics import TACTiCS; model = TACTiCS(human_counts, mouse_counts, blast_dist_matrix); model.train(); model.save()
```

## Evaluation signals

- TACTiCS ADS and recall decrease substantially under BLAST many-to-many matches relative to the ProtBERT baseline across cross-species comparisons
- BLAST one-to-one condition shows mixed effects across species pairs, not uniform degradation
- SAMap performance is similar to baseline except recall dropping from 0.8 to 0.73 with ProtBERT matches
- Each condition produces a complete gene similarity distance matrix suitable as the third required TACTiCS input
- All per-condition ADS and recall values are written to gene_matching_ablation_metrics.csv

## Limitations

- BLAST-based gene matching measurably worsens TACTiCS cell type matching, so it should not be used as the production gene-matching method
- TACTiCS requires PyTorch, and the transformers package is only needed for embedding protein sequences — omitting it blocks ProtBERT baseline generation
- The workflow was validated on human-mouse cortical data from specific Zenodo deposits; other species pairs are not covered by the reported validation

## Evidence

- [methods] As input, `TACTiCS` requires two scRNA-seq datasets and a distance matrix describing the gene similarities.: "As input, `TACTiCS` requires two scRNA-seq datasets and a distance matrix describing the gene similarities."
- [methods] calc_dist description: "`calc_dist` calculates the latter for embeddings created with `embed_proteins` in `genes.py`."
- [abstract] BLAST vs ProtBERT finding: "our gene matching method results in better cell type matches than BLAST in our model"
- [readme] Zenodo 50k subsets: "the protein sequences, protein embeddings, raw count data and trained models for 50k subsets of the data can be found on [Zenodo]"
- [readme] PyTorch requirement: "TACTiCS requires PyTorch. The transformers package is only necessary for embedding protein sequences."
