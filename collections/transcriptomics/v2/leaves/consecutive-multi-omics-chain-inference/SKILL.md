---
name: consecutive-multi-omics-chain-inference
description: Use when you have scATAC-seq/scRNA-seq batch data plus a separate paired
  scRNA-seq/ADT (CITE-seq) dataset and need to predict protein (ADT) abundance for
  cells measured only by scATAC-seq, chaining epigenome→transcriptome→proteome inference.
when_to_use_negative:
- When directly paired multi-omics data covering all target modalities is available
  — single paired translation between two modalities suffices.
- When the target modality is spatial transcriptomics or single-cell Hi-C — the article
  names these only as future extensions not yet supported.
- When only transcriptome-level predictions are needed — the epigenome-to-transcriptome
  translator alone is sufficient without chaining to the CITE model.
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scButterfly
  role: Framework providing dual-aligned VAE translators for cross-modality prediction
    and the chained inference workflow
  repo: https://github.com/BioX-NKU/scButterfly
- name: scButterfly train_model
  role: Trains the epigenome-to-transcriptome translator on BMMC scATAC-seq/scRNA-seq
    batches
  repo: https://github.com/BioX-NKU/scButterfly
- name: scButterfly train_model_cite
  role: Trains the transcriptome-to-proteome translator on paired CITE_BMMC scRNA-seq/ADT
    data
  repo: https://github.com/BioX-NKU/scButterfly
- name: scButterfly calculate_cluster
  role: Computes AMI, ARI, HOM, NMI clustering indices of predicted ADT profiles against
    ground-truth cell-type labels
  repo: https://github.com/BioX-NKU/scButterfly
- name: scButterfly draw_cluster
  role: Renders t-SNE embeddings (draw_tsne) of predicted profiles for visual assessment
  repo: https://github.com/BioX-NKU/scButterfly
- name: scButterfly data_processing
  role: Provides RNA_data_preprocessing, ATAC_data_preprocessing, and CLR_transform
    for modality-specific preprocessing
  repo: https://github.com/BioX-NKU/scButterfly
- name: BABEL
  role: Baseline method combined with sciPENN for comparison in consecutive translation
- name: Polarbear
  role: Baseline method combined with sciPENN for comparison in consecutive translation
- name: sciPENN
  role: Baseline RNA-to-ADT predictor used in the BABEL+sciPENN and Polarbear+sciPENN
    baseline pipelines
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-47418-x
    title: 'scButterfly: a versatile single-cell cross-modality translation method
      via dual-aligned variational autoencoders'
  source_content_hash: sha256:cc99d5baaa090022a329c22a196e94171c22b93ef14ea5a437b50ee76df31dc7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioX-NKU/scButterfly
  tools:
  - BABEL
  - Polarbear
  - Python
  - scButterfly
  - scButterfly calculate_cluster
  - scButterfly data_processing
  - scButterfly draw_cluster
  - scButterfly train_model
  - scButterfly train_model_cite
  - sciPENN
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# consecutive-multi-omics-chain-inference

## Summary

Chains two trained scButterfly translators so single-modality scATAC-seq cells can be mapped first to predicted scRNA-seq profiles and then to predicted ADT (protein) profiles, enabling proteome-level analysis from epigenome-only measurements via consecutive translation.

## When to use

Use when you have scATAC-seq/scRNA-seq batch data plus a separate paired scRNA-seq/ADT (CITE-seq) dataset and need to predict protein (ADT) abundance for cells measured only by scATAC-seq, chaining epigenome→transcriptome→proteome inference.

## When NOT to use

- When directly paired multi-omics data covering all target modalities is available — single paired translation between two modalities suffices.
- When the target modality is spatial transcriptomics or single-cell Hi-C — the article names these only as future extensions not yet supported.
- When only transcriptome-level predictions are needed — the epigenome-to-transcriptome translator alone is sufficient without chaining to the CITE model.

## Inputs

- BMMC batch scATAC-seq/scRNA-seq data (AnnData objects of raw count matrices)
- CITE_BMMC paired scRNA-seq/ADT data (AnnData objects of raw count matrices)
- Ground-truth BMMC cell-type labels for clustering evaluation

## Outputs

- Predicted ADT profiles for scATAC-seq-only cells (consecutive translation output)
- AMI, ARI, HOM, and NMI clustering comparison table (consecutive_translation_metrics.tsv)
- Differential expression of proteins (DEP) analysis of predicted vs. measured ADT values
- t-SNE embeddings of predicted proteome profiles (draw_cluster.draw_tsne)

## How to apply

Load the BMMC batch scATAC-seq/scRNA-seq data and the CITE_BMMC paired scRNA-seq/ADT data with butterfly.Butterfly.load_data, preprocessing with data_processing.RNA_data_preprocessing, ATAC_data_preprocessing, and CLR_transform. Train the epigenome-to-transcriptome translator with scButterfly.train_model.Model.train using dual-aligned VAEs on the BMMC batches, then train the transcriptome-to-proteome translator with scButterfly.train_model_cite.Model.train on CITE_BMMC. At inference, run the first translator's Model.test to map scATAC-seq cells to predicted scRNA profiles, then feed those predictions into the CITE translator's Model.test to obtain predicted ADT profiles. Judge the chain by computing AMI, ARI, HOM, and NMI clustering scores via calculate_cluster.calculate_cluster_index against ground-truth BMMC cell-type labels and against baselines (BABEL+sciPENN, Polarbear+sciPENN), and by differential expression of proteins (DEP) analysis of predicted vs. measured ADT values with t-SNE embeddings from draw_cluster.draw_tsne.

## Related tools

- **scButterfly** (Framework providing dual-aligned VAE translators for cross-modality prediction and the chained inference workflow) — https://github.com/BioX-NKU/scButterfly
- **scButterfly train_model** (Trains the epigenome-to-transcriptome translator on BMMC scATAC-seq/scRNA-seq batches) — https://github.com/BioX-NKU/scButterfly
- **scButterfly train_model_cite** (Trains the transcriptome-to-proteome translator on paired CITE_BMMC scRNA-seq/ADT data) — https://github.com/BioX-NKU/scButterfly
- **scButterfly calculate_cluster** (Computes AMI, ARI, HOM, NMI clustering indices of predicted ADT profiles against ground-truth cell-type labels) — https://github.com/BioX-NKU/scButterfly
- **scButterfly draw_cluster** (Renders t-SNE embeddings (draw_tsne) of predicted profiles for visual assessment) — https://github.com/BioX-NKU/scButterfly
- **scButterfly data_processing** (Provides RNA_data_preprocessing, ATAC_data_preprocessing, and CLR_transform for modality-specific preprocessing) — https://github.com/BioX-NKU/scButterfly
- **BABEL** (Baseline method combined with sciPENN for comparison in consecutive translation)
- **Polarbear** (Baseline method combined with sciPENN for comparison in consecutive translation)
- **sciPENN** (Baseline RNA-to-ADT predictor used in the BABEL+sciPENN and Polarbear+sciPENN baseline pipelines)

## Examples

```
from scButterfly.butterfly import Butterfly; butterfly = Butterfly(); butterfly.load_data(RNA_data, ATAC_data, train_id, test_id, validation_id); butterfly.data_preprocessing(); butterfly.train_model(); A2R_predict, R2A_predict = butterfly.test_model()
```

## Evaluation signals

- AMI, ARI, HOM, and NMI scores from calculate_cluster_index should exceed those of BABEL+sciPENN and Polarbear+sciPENN baselines, written to consecutive_translation_metrics.tsv.
- DEP analysis should show predicted ADT values consistent with measured ADT, with t-SNE (draw_tsne) revealing clear cell-type separation.
- Predicted scRNA profiles from the first translator (Model.test) should feed forward into the CITE translator without dimension mismatches.
- Predicted ADT profiles should give clear elucidation of nuanced differences among cells at the proteome level.

## Limitations

- Performance for cross-cell-type and cross-organ translation is flagged as needing improvement via advanced machine learning techniques with adaptive and interpretable model parameters.
- Chained inference compounds translator errors: scRNA profiles used for CITE training come from a different dataset (CITE_BMMC) than the one being translated (BMMC), relying on cross-dataset generalization.
- Incorporation of cell heterogeneity information from public bulk omics data or annotated cell atlases, and prior knowledge of gene regulatory mechanisms, are proposed future improvements not currently implemented.

## Evidence

- [abstract] scButterfly can be generalized to unpaired data training, perturbation-response analysis, and consecutive translation.: "scButterfly can be generalized to unpaired data training, perturbation-response analysis, and consecutive translation."
- [methods] Chain preprocessing APIs: "butterfly.Butterfly.load_data"
- [methods] Clustering index evaluation API: "calculate_cluster.calculate_cluster_index"
- [readme] Load raw count matrices before preprocessing: [evidence span withheld — anchor status: external_doc]
- [readme] Augmentation improves prediction quality: [evidence span withheld — anchor status: external_doc]
