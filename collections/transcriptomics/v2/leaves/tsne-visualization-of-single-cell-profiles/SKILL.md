---
name: tsne-visualization-of-single-cell-profiles
description: Use when you have translated or measured single-cell profiles (e.g.,
  predicted ADT from consecutive epigenome→transcriptome→proteome translation) and
  need a 2-D embedding to visually assess cell-type separation.
when_to_use_negative:
- Input is raw count matrices that have not yet been translated or preprocessed (RNA_data_preprocessing,
  ATAC_data_preprocessing, CLR_transform should run first)
- You need quantitative clustering scores rather than visual embeddings — use calculate_cluster.calculate_cluster_index
  (AMI, ARI, HOM, NMI) instead
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scButterfly draw_cluster
  role: Provides draw_tsne for rendering t-SNE embeddings of translated single-cell
    profiles
  repo: https://github.com/BioX-NKU/scButterfly
- name: scButterfly
  role: Generates the translated profiles (predicted ADT) that are visualized; installed
    via pip from PyPI
  repo: https://github.com/BioX-NKU/scButterfly
- name: Python
  role: Runtime environment (Python 3.9 via conda) in which scButterfly and its visualization
    API are executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-47418-x
    title: 'scButterfly: a versatile single-cell cross-modality translation method
      via dual-aligned variational autoencoders'
  source_content_hash: sha256:45f3f9cf1de0b0b66d20a378a1f6d21ff2b1adb33ee7d46616f151c0bb15a6c9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioX-NKU/scButterfly
  tools:
  - BABEL
  - Polarbear
  - Python
  - scButterfly
  - scButterfly draw_cluster
  - sciPENN
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# tsne-visualization-of-single-cell-profiles

## Summary

Render t-SNE embeddings of single-cell profiles (e.g., scButterfly-predicted ADT values) using scButterfly's draw_cluster.draw_tsne to visually inspect whether translated profiles recapitulate nuanced differences among cells and cell-type structure.

## When to use

Use when you have translated or measured single-cell profiles (e.g., predicted ADT from consecutive epigenome→transcriptome→proteome translation) and need a 2-D embedding to visually assess cell-type separation.

## When NOT to use

- Input is raw count matrices that have not yet been translated or preprocessed (RNA_data_preprocessing, ATAC_data_preprocessing, CLR_transform should run first)
- You need quantitative clustering scores rather than visual embeddings — use calculate_cluster.calculate_cluster_index (AMI, ARI, HOM, NMI) instead

## Inputs

- Predicted ADT profiles from chained scButterfly translation (epigenome-to-transcriptome then transcriptome-to-proteome)
- Measured ADT profiles from CITE_BMMC paired scRNA-seq/ADT data
- Ground-truth BMMC cell-type labels for coloring embeddings
- AnnData objects of single-cell profiles

## Outputs

- t-SNE embedding plots of predicted and measured profiles colored by cell type
- Qualitative visual comparison supporting clustering performance assessment

## How to apply

After obtaining predicted profiles from scButterfly (e.g., ADT predictions from chained Model.test calls on BMMC and CITE_BMMC data), call scButterfly's draw_cluster.draw_tsne to render t-SNE embeddings of the profiles, optionally colored by ground-truth BMMC cell-type labels. Use the embeddings alongside quantitative clustering metrics (AMI, ARI, HOM, NMI from calculate_cluster.calculate_cluster_index) to judge whether the predicted profiles give a clear elucidation of nuanced differences among cells. Visual inspection complements the metric comparison against baselines like BABEL+sciPENN and Polarbear+sciPENN, whose comparison table is written to consecutive_translation_metrics.tsv.

## Related tools

- **scButterfly draw_cluster** (Provides draw_tsne for rendering t-SNE embeddings of translated single-cell profiles) — https://github.com/BioX-NKU/scButterfly
- **scButterfly** (Generates the translated profiles (predicted ADT) that are visualized; installed via pip from PyPI) — https://github.com/BioX-NKU/scButterfly
- **Python** (Runtime environment (Python 3.9 via conda) in which scButterfly and its visualization API are executed)

## Examples

```
from scButterfly.draw_cluster import draw_tsne; draw_tsne(predicted_ADT, cell_type_labels)
```

## Evaluation signals

- t-SNE embeddings of predicted ADT profiles should show separation consistent with ground-truth BMMC cell-type labels
- Visual clustering of predicted profiles should appear at least as coherent as embeddings from BABEL+sciPENN and Polarbear+sciPENN baselines
- Embedding-based impressions should agree with quantitative AMI, ARI, HOM, and NMI scores from calculate_cluster.calculate_cluster_index recorded in consecutive_translation_metrics.tsv
- Predicted-vs-measured ADT profiles should occupy comparable embedding space, supporting differential expression of proteins (DEP) interpretation

## Limitations

- t-SNE is qualitative; the paper's claims of 'overall superior cell clustering performance' are quantified via AMI, ARI, HOM, and NMI, so embeddings alone are insufficient evidence
- scButterfly's future extensions (spatial transcriptomics, single-cell Hi-C, cross-cell-type/cross-organ translation) are not yet supported, limiting the contexts in which such visualizations are validated

## Evidence

- [full_text] The ADT profiles predicted via scButterfly's consecutive translation gave a clear elucidation of nuanced differences among cells and provided overall superior cell clustering performance than baseline methods (BABEL+sciPENN, Polarbear+sciPENN).: "gave a clear elucidation of nuanced differences among cells and provided overall superior cell clustering performance"
- [methods] draw_tsne is exposed via the scButterfly draw_cluster module for rendering embeddings.: [evidence span withheld — anchor status: not_found]
- [readme] README documents the workflow of preprocessing, training, and predicting profiles that are subsequently visualized.: [evidence span withheld — anchor status: external_doc]
