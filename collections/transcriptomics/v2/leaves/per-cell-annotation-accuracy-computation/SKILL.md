---
name: per-cell-annotation-accuracy-computation
description: Use when benchmarking scRNA-seq cell-type annotation tools (e.g. EasyCellType
  vs Garnett, scPred, SingleR) on simulated datasets with known ground-truth labels
  and you need per-cell accuracy tables.
when_to_use_negative:
- The dataset is real tissue data without simulated ground-truth cell-type labels
  — per-cell accuracy against truth cannot be computed and only plausibility or expert
  review applies
- The goal is annotation itself (e.g. querying marker lists against CellMarker via
  EasyCellType) rather than evaluating annotation accuracy
- Accuracy has already been aggregated at the cluster level and per-cell resolution
  is not required
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: EasyCellType
  role: Marker-based annotation method under evaluation; its GSEA and modified Fisher's
    exact test soft/hard classifier accuracies are computed per cell against simulated
    ground truth
  repo: https://github.com/rx-li/EasyCellType
- name: Seurat
  role: Standard preprocessing pipeline (normalization, variable feature selection,
    scaling, PCA, clustering, FindAllMarkers) applied to every simulated PBMC dataset
    before annotation
- name: Garnett
  role: Supervised baseline run with a pre-trained PBMC classifier in cell-level and
    cluster-level extension modes; per-cell accuracy computed against ground truth
- name: scPred
  role: Supervised baseline trained on 3500 sampled cells per dataset, generating
    predictions in both prediction and no-rejection variants for accuracy computation
- name: SingleR
  role: Supervised baseline annotating cells with the HumanPrimaryCellAtlasData reference;
    per-cell accuracy computed against ground truth
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad029
    title: 'EasyCellType: marker-based cell-type annotation by automatically querying
      multiple databases'
  source_content_hash: sha256:b94f96e63c8c04cc3651b0ac1ae340797eb11f4804d602604add854eff82facc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rx-li/EasyCellType
  tools:
  - EasyCellType
  - Garnett
  - Seurat
  - SingleR
  - scPred
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# per-cell-annotation-accuracy-computation

## Summary

Computes per-cell annotation accuracy for single-cell RNA-seq cell-type annotation tools by comparing each predicted cell label against simulated ground-truth labels, as done in the EasyCellType paper's PBMC simulation benchmark for Supplementary Figure S5. This enables head-to-head accuracy comparison across tools, tool variants, and dataset sizes.

## When to use

Use when benchmarking scRNA-seq cell-type annotation tools (e.g. EasyCellType vs Garnett, scPred, SingleR) on simulated datasets with known ground-truth labels and you need per-cell accuracy tables.

## When NOT to use

- The dataset is real tissue data without simulated ground-truth cell-type labels — per-cell accuracy against truth cannot be computed and only plausibility or expert review applies
- The goal is annotation itself (e.g. querying marker lists against CellMarker via EasyCellType) rather than evaluating annotation accuracy
- Accuracy has already been aggregated at the cluster level and per-cell resolution is not required

## Inputs

- Simulated PBMC scRNA-seq count matrices at several cell counts with known ground-truth cell-type labels
- Seurat-processed objects (normalized, scaled, PCA, clustered) per simulated dataset
- Pre-trained PBMC classifier for Garnett (cell-level and cluster-level extension modes)
- scPred training subsample of 3500 cells per dataset
- SingleR reference: HumanPrimaryCellAtlasData

## Outputs

- Per-cell annotation accuracy values for every tool/variant and dataset size
- Combined accuracy table written to figure_s5_accuracy_table.csv
- Qualitative comparison against the pattern shown in Supplementary Figure S5

## How to apply

First simulate PBMC scRNA-seq count matrices at several cell counts with known ground-truth cell-type labels, then preprocess each with the standard Seurat pipeline (NormalizeData, FindVariableFeatures with vst/nfeatures=2000, ScaleData, RunPCA, FindNeighbors/FindClusters at resolution 0.5, FindAllMarkers with only.pos=TRUE, min.pct=0.25, logfc.threshold=0.25). Annotate each dataset with every tool and variant: Garnett with a pre-trained PBMC classifier in both cell-level and cluster-level extension modes, scPred trained on 3500 sampled cells per dataset in both prediction and no-rejection variants, and SingleR using the HumanPrimaryCellAtlasData reference, alongside the EasyCellType GSEA and Fisher-exact-test soft/hard classifiers. For each tool/variant and dataset size, count matches between the predicted label and the simulated ground-truth label for every single cell and divide by the number of cells. Write the combined accuracy table across all combinations to figure_s5_accuracy_table.csv, and check that values are internally consistent and qualitatively reproduce the Supplementary Figure S5 pattern (EasyCellType soft classifiers comparable to or better than supervised methods; supervised methods except Garnett and SingleR generally beat EasyCellType's hard classifiers).

## Related tools

- **EasyCellType** (Marker-based annotation method under evaluation; its GSEA and modified Fisher's exact test soft/hard classifier accuracies are computed per cell against simulated ground truth) — https://github.com/rx-li/EasyCellType
- **Seurat** (Standard preprocessing pipeline (normalization, variable feature selection, scaling, PCA, clustering, FindAllMarkers) applied to every simulated PBMC dataset before annotation)
- **Garnett** (Supervised baseline run with a pre-trained PBMC classifier in cell-level and cluster-level extension modes; per-cell accuracy computed against ground truth)
- **scPred** (Supervised baseline trained on 3500 sampled cells per dataset, generating predictions in both prediction and no-rejection variants for accuracy computation)
- **SingleR** (Supervised baseline annotating cells with the HumanPrimaryCellAtlasData reference; per-cell accuracy computed against ground truth)

## Examples

```
pred <- SingleR(test = seurat_obj, ref = HumanPrimaryCellAtlasData(), labels = ref_labels); acc <- mean(pred$labels == ground_truth)
```

## Evaluation signals

- The combined accuracy table figure_s5_accuracy_table.csv contains one accuracy value for every tool/variant and dataset-size combination
- All per-cell accuracy values are internally consistent across datasets and tool variants (e.g. bounded in [0,1], reproducible on re-run)
- The qualitative pattern reproduces Supplementary Figure S5: supervised methods except Garnett and SingleR generally outperform EasyCellType's hard classifiers, while EasyCellType's soft classifiers are comparable to or better than the supervised methods
- Each simulated dataset carries known ground-truth labels so that per-cell match counts against predictions are well-defined
- Seurat preprocessing parameters match the paper's workflow (FindVariableFeatures nfeatures=2000, FindClusters resolution=0.5, FindAllMarkers min.pct=0.25, logfc.threshold=0.25)

## Limitations

- Ground truth is simulated PBMC data, so the accuracy pattern is qualitative and may not transfer directly to real PBMC tissue
- scPred is trained on only 3500 sampled cells per dataset, introducing sampling variability in the computed accuracies
- The article does not document software version numbers for the annotation tools, complicating exact reproduction

## Evidence

- [supplementary] Simulate PBMC scRNA-seq count matrices at several cell counts with known ground-truth labels to serve as benchmark datasets: [evidence span withheld — anchor status: not_found]
- [supplementary] Compute per-cell annotation accuracy and write the combined table: [evidence span withheld — anchor status: not_found]
- [supplementary] scPred training and variant settings: [evidence span withheld — anchor status: not_found]
- [methods] Seurat marker identification parameters used before annotation: "markers <- FindAllMarkers(pbmc, only.pos = TRUE, min.pct = 0.25, logfc.threshold = 0.25)"
- [abstract] Simulation study supporting the comparison context: "The simulation study and real-data applications demonstrate favorable results by the proposed method."
- [readme] Seurat object construction feeding the annotation workflow: [evidence span withheld — anchor status: external_doc]
