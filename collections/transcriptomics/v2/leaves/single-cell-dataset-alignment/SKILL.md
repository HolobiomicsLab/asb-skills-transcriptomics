---
name: single-cell-dataset-alignment
description: Use when multiple annotated scRNA-seq datasets must be combined to learn
  a hierarchical classification tree, and batch effects are expected between the datasets
  (e.g., different PBMC studies measured on different platforms).
when_to_use_negative:
- When only a single dataset is analyzed and no cross-dataset population matching
  is needed
- When the goal is batch correction itself rather than preprocessing for hierarchical
  classifier training — scHPL (and this alignment step as framed here) is not a batch
  correction tool
- When datasets cannot be assumed to share cell populations or a roughly consistent
  annotation terminology across studies
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Seurat
  role: Seurat v3 integration on 2000 highly variable genes to align PBMC datasets
    before building the AnnData object fed to scHPL
- name: scVI
  role: Advised alternative alignment method prior to scHPL population matching
- name: scArches
  role: Advised alternative alignment method; part of the treeArches framework around
    scHPL for building and updating reference atlases
  repo: https://github.com/theislab/scarches
- name: scHPL
  role: 'Downstream consumer: learns the hierarchical classification tree from the
    aligned, annotated datasets'
  repo: https://github.com/lcmmichielsen/scHPL
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-23196-8
    title: Hierarchical progressive learning of cell identities in single-cell data
  source_content_hash: sha256:3e549038551df1afff312a18bab11c1232a0f19cee0eb992c391534939dfc4c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scarches
  tools:
  - Seurat
  - scArches
  - scHPL
  - scVI
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# single-cell-dataset-alignment

## Summary

Align multiple annotated single-cell RNA-seq datasets into a shared feature space before hierarchical classifier training, because scHPL is not a batch correction tool and requires batch-effect-corrected inputs for reliable cell population matching across datasets.

## When to use

Use when multiple annotated scRNA-seq datasets must be combined to learn a hierarchical classification tree, and batch effects are expected between the datasets (e.g., different PBMC studies measured on different platforms).

## When NOT to use

- When only a single dataset is analyzed and no cross-dataset population matching is needed
- When the goal is batch correction itself rather than preprocessing for hierarchical classifier training — scHPL (and this alignment step as framed here) is not a batch correction tool
- When datasets cannot be assumed to share cell populations or a roughly consistent annotation terminology across studies

## Inputs

- Multiple annotated scRNA-seq datasets with cell-type labels and batch indicators (e.g., PBMC-eQTL, PBMC-Bench10Xv2, PBMC-FACS)
- Highly variable gene list (e.g., 2000 HVGs) for Seurat v3 integration
- Aligned datasets downloadable from the scHPL Zenodo record (doi:10.5281/zenodo.4557712)

## Outputs

- Batch-corrected integrated expression matrix in the '.X' slot of an AnnData object with labels and batch metadata
- Aligned dataset set suitable for input to scHPL's learn function for tree construction

## How to apply

Integrate the annotated datasets (e.g., PBMC-eQTL, PBMC-Bench10Xv2, PBMC-FACS) using Seurat v3 integration on ~2000 highly variable genes, then copy the integrated expression matrix into the '.X' slot of a new AnnData object with cell labels and batch indicators as metadata columns. Alternative alignment methods advised by the scHPL authors are scVI or scArches (via the treeArches framework). The rationale is that scHPL automatically matches cell populations across datasets and constructs a hierarchical classification tree, so misaligned datasets cause spurious or erroneous tree mismatches. Only after alignment should scHPL's learn function be run; the aligned data then feed per-node classifiers (linear SVM for Seurat-integrated data retaining many dimensions, kNN, or one-class SVM). Note that adding a new dataset later requires rerunning the alignment and consequently scHPL.

## Related tools

- **Seurat** (Seurat v3 integration on 2000 highly variable genes to align PBMC datasets before building the AnnData object fed to scHPL)
- **scVI** (Advised alternative alignment method prior to scHPL population matching)
- **scArches** (Advised alternative alignment method; part of the treeArches framework around scHPL for building and updating reference atlases) — https://github.com/theislab/scarches
- **scHPL** (Downstream consumer: learns the hierarchical classification tree from the aligned, annotated datasets) — https://github.com/lcmmichielsen/scHPL

## Examples

```
import scanpy as sc; adata_int = sc.read_h5ad('pbmc_integrated.h5ad'); from scHPL import learn; tree = learn(adata_int.X, labels=adata_int.obs['labels'], batch=adata_int.obs['batch'], classifier='linear_svm')
```

## Evaluation signals

- The integrated expression matrix sits in AnnData .X with cell labels and batch indicators present as metadata columns
- After scHPL learning, the constructed tree correctly matches known shared cell populations across the aligned datasets (e.g., matches the PBMC inter-dataset tree in Fig. 5B)
- Predictions on a held-out aligned dataset (e.g., PBMC-Bench10Xv3) match published confusion-matrix structure respecting hierarchy boundaries (Fig. 5C)
- No obviously erroneous tree mismatches; if observed, check marker gene expression to distinguish mislabeled populations from alignment errors
- Downstream classifier performance: linear SVM accuracy equal to or higher than one-class SVM on the aligned data, with the one-class SVM degrading more on small populations

## Limitations

- Adding a new dataset requires rerunning the alignment — and consequently also scHPL — with current batch correction tools
- Batch effects make it hard to determine whether mistakes in the constructed tree are caused by erroneous dataset alignment or by mismatches created by scHPL
- scHPL is not a batch correction tool; alignment must happen upstream of tree learning

## Evidence

- [discussion] Since batch effects are inevitable when combining datasets, we require datasets to be aligned before running scHPL: "Since batch effects are inevitable when combining datasets, we require datasets to be aligned before running scHPL"
- [methods] Align datasets before matching populations: "scHPL is not a batch correction tool, we advise to align the datasets before matching the cell populations."
- [readme] README alignment advice with scVI/scArches: "NOTE: scHPL is not a batch correction tool, we advise to align the datasets before matching the cell populations. We advise doing this with scVI or scArches"
- [discussion] Rerun required for new datasets: "when a new dataset is added, the alignment—and consequently also scHPL—has to be rerun"
