---
name: scrna-seq-denoising-benchmark-evaluation
description: Use when a CITE-seq or multimodal dataset provides protein-level sub-population
  labels (e.g., CD56bright/CD56dim NK cells from CD16/CD56 ADTs) and you must quantify
  whether denoised RNA better matches that protein structure than raw counts.
when_to_use_negative:
- The dataset lacks a paired protein/multimodal measurement, so no independent ground-truth
  labels exist against which to compute silhouette agreement
- The question is about imputation of specific missing values rather than whether
  denoising recovers sub-population structure; the article notes the non-trivial distinction
  between true (celltype-specific) and false (dropout) zeros makes classical imputation
  evaluation unsuitable
- The RNA matrix has already been heavily filtered or transformed (e.g., normalized,
  log-transformed) in ways incompatible with DCA's raw-count negative binomial input
  assumption
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: DCA
  role: Deep count autoencoder (Keras/TensorFlow, NB or ZINB loss) producing the denoised
    RNA matrix evaluated in the benchmark
  repo: https://github.com/theislab/dca
- name: Scanpy
  role: Python API exposing DCA via the 'dca' method in the preprocessing package,
    and filter_genes_dispersion for selecting highly variable genes
- name: mclust
  role: R package fitting the two-component Gaussian mixture on CD16/CD56 protein
    levels to derive benchmark cluster labels
- name: anndata
  role: h5ad-formatted HDF5 storage for the count matrices used in the pipeline
  repo: https://github.com/theislab/anndata
- name: kopt
  role: Hyperparameter optimization (with hyperopt, TPE, 1000 configurations) if DCA
    parameters need tuning by lowest reconstruction error
  repo: https://github.com/Avsecz/kopt
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-07931-2
    title: Single-cell RNA-seq denoising using a deep count autoencoder
  source_content_hash: sha256:683876ec6f25a7300866634676dd8214ba12848d8a9627b5b4096de2059317c8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/dca
  tools:
  - DCA
  - Scanpy
  - anndata
  - kopt
  - mclust
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# scrna-seq-denoising-benchmark-evaluation

## Summary

A benchmark skill for judging whether scRNA-seq denoising (e.g., DCA deep count autoencoder with NB/ZINB loss) recovers biological structure, using CITE-seq antibody-derived tag (ADT) protein labels as ground truth and quantifying agreement via silhouette coefficients on marker genes such as NCAM1/FCGR3A.

## When to use

Use when a CITE-seq or multimodal dataset provides protein-level sub-population labels (e.g., CD56bright/CD56dim NK cells from CD16/CD56 ADTs) and you must quantify whether denoised RNA better matches that protein structure than raw counts.

## When NOT to use

- The dataset lacks a paired protein/multimodal measurement, so no independent ground-truth labels exist against which to compute silhouette agreement
- The question is about imputation of specific missing values rather than whether denoising recovers sub-population structure; the article notes the non-trivial distinction between true (celltype-specific) and false (dropout) zeros makes classical imputation evaluation unsuitable
- The RNA matrix has already been heavily filtered or transformed (e.g., normalized, log-transformed) in ways incompatible with DCA's raw-count negative binomial input assumption

## Inputs

- CITE-seq count matrix (GSE100866 Stoeckius et al. cord blood mononuclear cells, 8,005 human cells) with RNA counts and ADT protein measurements
- Cell annotation / cluster labels identifying the NK cell population (906 cells)
- DCA-denoised RNA expression matrix (denoised counts from the deep count autoencoder)
- Protein-derived sub-population labels from a 2-component mclust Gaussian mixture on FCGR3A/NCAM1 ADT levels

## Outputs

- Average silhouette width of protein labels in protein ADT space (~0.47)
- Average silhouette width of protein labels in original NCAM1/FCGR3A RNA space (~0.17)
- Average silhouette width of protein labels in DCA-denoised NCAM1/FCGR3A RNA space (~0.58)
- Comparison table written to silhouette_scores.csv

## How to apply

Load the Stoeckius et al. CITE-seq cord blood dataset (GSE100866, subset to 8,005 human cells by removing cells with <90% human UMI counts), restrict to the annotated NK population of 906 cells, and fit a two-component Gaussian mixture (R mclust) on the CD16 (FCGR3A) and CD56 (NCAM1) ADT levels to derive protein labels. Denoise the RNA with DCA, which replaces original counts with the mean of the negative binomial component and by default uses three hidden layers (64, 32, 64) with RMSProp (learning rate 0.001, x0.1 on plateau, early stopping after 25 epochs). Compute average silhouette widths of the protein labels against three spaces with cluster::silhouette: protein space (expected ~0.47), original RNA of NCAM1/FCGR3A (expected ~0.17), and DCA-denoised NCAM1/FCGR3A RNA (expected ~0.58). Assemble the three values into silhouette_scores.csv and accept the run if the denoised ordering (DCA > protein > original RNA) is reproduced.

## Related tools

- **DCA** (Deep count autoencoder (Keras/TensorFlow, NB or ZINB loss) producing the denoised RNA matrix evaluated in the benchmark) — https://github.com/theislab/dca
- **Scanpy** (Python API exposing DCA via the 'dca' method in the preprocessing package, and filter_genes_dispersion for selecting highly variable genes)
- **mclust** (R package fitting the two-component Gaussian mixture on CD16/CD56 protein levels to derive benchmark cluster labels)
- **anndata** (h5ad-formatted HDF5 storage for the count matrices used in the pipeline) — https://github.com/theislab/anndata
- **kopt** (Hyperparameter optimization (with hyperopt, TPE, 1000 configurations) if DCA parameters need tuning by lowest reconstruction error) — https://github.com/Avsecz/kopt

## Examples

```
import scanpy as sc; sc.pp.dca(adata)  # then cluster::silhouette on protein labels vs. raw vs. denoised NCAM1/FCGR3A
```

## Evaluation signals

- Ordering invariant: DCA-denoised RNA silhouette (~0.58) > protein ADT space (~0.47) > original RNA (~0.17)
- Exact expected values: protein-space silhouette ~0.47, original RNA ~0.17, denoised RNA ~0.58 for the 906-cell NK sub-population
- Protein labels reproduced: 2-component mclust on FCGR3A/NCAM1 ADTs yields the known CD56bright/CD56dim NK split
- silhouette_scores.csv contains a 3-row comparison table (protein, original RNA, DCA-denoised RNA) with finite silhouette widths
- Denoised matrix was generated by replacing original counts with the negative binomial mean, not by naive smoothing of raw counts

## Limitations

- It may be difficult to determine when denoising improves scRNA-seq data, and defining ground truth for real data is challenging; this silhouette-vs-protein design is one partial answer, not a universal benchmark
- Overimputation can occur with inadequate hyperparameter choices (e.g., too low-dimensional bottleneck layer); hyperparameters were not fine-tuned for the manuscript's analyses
- The published repo provides no DOI, accession IDs, or preserved archive (e.g., Zenodo) for datasets or code, and hardware specifics for main analyses are only partially specified

## Evidence

- [abstract] DCA takes the count distribution, overdispersion and sparsity of the data into account using a negative binomial noise model with or without zero-inflation: "DCA takes the count distribution, overdispersion and sparsity of the data into account using a negative binomial noise model with or without zero-inflation"
- [discussion] denoised matrix generated by NB mean: "The denoised matrix is generated by replacing the original count values with the mean of the negative binomial component"
- [discussion] evaluation difficulty limitation: "The evaluation of denoising is difficult because the definition of a ground truth can be challenging for real data."
- [discussion] dataset subsetting filter: "data were subset to 8,005 human cells by removing cells with less than 90% human UMI counts."
- [intro] Scanpy API for DCA: [evidence span withheld — anchor status: not_found]
- [readme] README description of DCA: "A deep count autoencoder network to denoise scRNA-seq data and remove the dropout effect by taking the count structure, overdispersed nature and sparsity of the data into account"
