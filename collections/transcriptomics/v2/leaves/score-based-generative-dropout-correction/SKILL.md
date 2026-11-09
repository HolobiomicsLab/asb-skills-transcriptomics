---
name: score-based-generative-dropout-correction
description: Use when seq-based ST data (Slide-seq V2, Visium) has dropout or sparse
  gene expression and a paired scRNA-seq/snRNA-seq reference is available, and you
  need to recover/impute whole-transcriptome expression and benchmark the correction.
when_to_use_negative:
- When the ST data are image-based (e.g., MERFISH) already at single-cell resolution
  — the article's stated use case here is enhancing seq-based data like Slide-seq
  V2, not image-based panels
- When no paired single-cell reference atlas is available for the tissue, since the
  score model is trained on cell-type-specific expression from the reference
- When the goal is cell-type proportion estimation only (use deconvolution tools such
  as RCTD, Cell2location, or CARD) rather than transcriptome-wide expression recovery
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: SpatialScope
  role: 'Main method: learns the conditional score-based generative model from the
    snRNA-seq reference and performs gene expression decomposition via annealed Langevin
    dynamics posterior sampling to correct dropouts'
  repo: https://github.com/YangLabHKUST/SpatialScope
- name: Tangram
  role: Comparison baseline for dropout correction, evaluated at the same MAE/PCC
    metrics on the subsampled datasets
- name: RCTD (spacexr)
  role: Inspiration/source of the singlet/doublet classification replacing nucleus
    segmentation on Slide-seq spots
  repo: https://github.com/dmcable/spacexr
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-43629-w
    title: Integrating spatial and single-cell transcriptomics data using deep generative
      models with SpatialScope
  source_content_hash: sha256:808a65675a10b192279c6bfe92881b0187e5982194419894467e6219fc0fd738
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YangLabHKUST/SpatialScope
  tools:
  - RCTD
  - RCTD (spacexr)
  - SpatialScope
  - Tangram
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# score-based-generative-dropout-correction

## Summary

Corrects simulated technical dropouts in spatial transcriptomics (e.g., Slide-seq V2 cerebellum) by running SpatialScope's gene expression decomposition, which uses a conditional score-based generative model trained on a paired scRNA-seq/snRNA-seq reference and annealed Langevin dynamics posterior sampling, then benchmarks the corrected expression against Tangram using MAE and PCC.

## When to use

Use when seq-based ST data (Slide-seq V2, Visium) has dropout or sparse gene expression and a paired scRNA-seq/snRNA-seq reference is available, and you need to recover/impute whole-transcriptome expression and benchmark the correction.

## When NOT to use

- When the ST data are image-based (e.g., MERFISH) already at single-cell resolution — the article's stated use case here is enhancing seq-based data like Slide-seq V2, not image-based panels
- When no paired single-cell reference atlas is available for the tissue, since the score model is trained on cell-type-specific expression from the reference
- When the goal is cell-type proportion estimation only (use deconvolution tools such as RCTD, Cell2location, or CARD) rather than transcriptome-wide expression recovery

## Inputs

- Seq-based ST data (e.g., Slide-seq V2 mouse cerebellum with 10 μm spots; 20,141 genes × 8,952 spots after filtering)
- Paired scRNA-seq/snRNA-seq reference (e.g., mouse cerebellum snRNA-seq: 24,387 genes, 15,609 cells, 19 cell types)
- Marker gene list for dropout simulation (e.g., 22 high-capture marker genes)
- Optional paired histology image for nucleus segmentation (not needed for Slide-seq singlet/doublet classification)

## Outputs

- Posterior-mean de-noised, dropout-corrected single-cell-resolution expression matrix (10,975 cells across 8,952 spots in the cerebellum case)
- Per-spot cell type assignments (singlet/doublet classifications)
- MAE and PCC comparison table of corrected vs. ground-truth expression for SpatialScope and Tangram at each subsampling level (50, 100, 200 counts)

## How to apply

First preprocess the ST data: filter genes with zero counts across all spots, remove spots with total UMI counts less than 100 (and for Slide-seq V2 cerebellum also apply gene/spot detection thresholds), yielding e.g. 20,141 genes and 8,952 spots. Replace nucleus segmentation with RCTD-inspired singlet/doublet classification (at most two cells per 10 μm Slide-seq spot) to obtain single-cell-resolution units, then run cell type identification using total-counts normalization plus log(1+x) transformation, selecting the top 1000 highly variable genes and 50 marker genes per cell type, with the Potts-model spatial smoothness prior (ν=10). To simulate dropout, subsample UMIs of high-capture marker genes (e.g., 22 cerebellar markers) down to fixed total counts such as 50, 100, and 200. Reconstruct expression by running SpatialScope's gene expression decomposition — a conditional score-based generative model learned from the reference (e.g., mouse cerebellum snRNA-seq: 24,387 genes, 15,609 cells, 19 cell types) with annealed Langevin dynamics posterior sampling (Algorithm 1) — and run Tangram on the same subsampled data as the comparison baseline. Evaluate by computing mean absolute error (MAE) and Pearson's correlation (PCC) between corrected and ground-truth expression per method per subsampling level; the article reports SpatialScope achieves the lowest MAE and highest PCC across all settings.

## Related tools

- **SpatialScope** (Main method: learns the conditional score-based generative model from the snRNA-seq reference and performs gene expression decomposition via annealed Langevin dynamics posterior sampling to correct dropouts) — https://github.com/YangLabHKUST/SpatialScope
- **Tangram** (Comparison baseline for dropout correction, evaluated at the same MAE/PCC metrics on the subsampled datasets)
- **RCTD (spacexr)** (Inspiration/source of the singlet/doublet classification replacing nucleus segmentation on Slide-seq spots) — https://github.com/dmcable/spacexr

## Examples

```
git clone https://github.com/YangLabHKUST/SpatialScope.git && cd SpatialScope && conda env create -f environment.yml
```

## Evaluation signals

- SpatialScope achieves the lowest MAE and highest PCC versus Tangram across all subsampling settings (50, 100, 200 counts), reproducing Fig. 5b-d
- The corrected output covers 10,975 cells across 8,952 spots after singlet/doublet classification (6,929 singlet spots and 2,023 doublet spots)
- Pre-filtering yields 20,141 genes and 8,952 spots after removing zero-count genes and spots with total UMI counts < 100
- Cell type identification uses 19 cerebellar cell types from the 15,609-cell snRNA-seq reference with the Potts prior at ν=10

## Limitations

- Training the deep generative models is time-consuming, measured in hours or days
- The Potts model assumption that neighboring cells belong to the same cell type may not always be valid
- Platform effects must be appropriately corrected or cell type identification and gene expression decomposition results will be unsatisfactory
- No software version numbers or detailed computational environment are reported

## Evidence

- [methods] Gene expression decomposition via conditional score model with Langevin sampling: [evidence span withheld — anchor status: not_found]
- [methods] Score-based model learned from single-cell reference: "We first learn a score-based generative model to approximate the expression distribution of different cell types"
- [methods] Slide-seq singlet/doublet classification: "we replace the first step nucleus segmentation by singlet/doublet classification, which assumes that at most two cell types co-exist within a Slide-seq spot"
- [methods] Filtering and normalization of reference genes: [evidence span withheld — anchor status: not_found]
- [readme] README: SpatialScope is a deep generative integration method: "A unified approach for integrating spatial and single-cell transcriptomics data by leveraging deep generative models"
