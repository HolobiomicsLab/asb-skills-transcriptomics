---
name: scrna-seq-count-matrix-downsampling
description: Use when you must quantify a method's scalability on large scRNA-seq
  data by regressing runtime against number of cells on a series of downsampled count
  matrices.
when_to_use_negative:
- The dataset is already small (e.g. thousands of cells like the 68K PBMC or Paul
  et al. 2,730-cell set) — subsampling then adds no scalability information.
- The goal is quality/accuracy benchmarking of denoising rather than runtime scaling
  — the article notes that evaluating whether denoising improves data is difficult
  without ground truth.
- The method under test cannot run at the target scale at all (e.g. SAVER/scImpute
  on 100,000 cells take hours and may be intractable), making the runtime comparison
  uninformative.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Scanpy
  role: Provides filter_genes_dispersion to select the top 1,000 highly variable genes
    in each downsampled subset
  repo: https://github.com/theislab/dca
- name: DCA
  role: Method under test whose CPU runtime is measured across downsample sizes using
    its negative binomial noise model and default 64-32-64 architecture
  repo: https://github.com/theislab/dca
- name: MAGIC
  role: Competing denoising method timed on downsampled subsets where tractable
  repo: https://github.com/pkathail/magic
- name: SAVER
  role: Competing denoising method timed on downsampled subsets where tractable
  repo: https://github.com/mohuangx/SAVER
- name: scImpute
  role: Competing imputation method timed on downsampled subsets where tractable
  repo: https://github.com/Vivianstats/scImpute
- name: anndata
  role: h5ad-formatted data container for the count matrices
  repo: https://github.com/theislab/anndata
- name: 10XGenomics/single-cell-3prime-paper
  role: Source repository for the 1.3M-cell count matrix
  repo: http://www.github.com/10XGenomics/single-cell-3prime-paper
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-07931-2
    title: Single-cell RNA-seq denoising using a deep count autoencoder
  source_content_hash: sha256:8458414eeceaf80c76bf0322056317b486735afd902ea414b1696fc511c5abc8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/dca
  tools:
  - 10XGenomics/single-cell-3prime-paper
  - DCA
  - Keras
  - MAGIC
  - SAVER
  - Scanpy
  - TensorFlow
  - anndata
  - scImpute
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# scrna-seq-count-matrix-downsampling

## Summary

Systematically downsample a very large scRNA-seq count matrix (e.g. the 1.3M-cell 10X Genomics mouse brain dataset) to a ladder of cell counts, restrict to top highly variable genes, and measure how a denoising method's wall-clock runtime scales with dataset size.

## When to use

Use when you must quantify a method's scalability on large scRNA-seq data by regressing runtime against number of cells on a series of downsampled count matrices.

## When NOT to use

- The dataset is already small (e.g. thousands of cells like the 68K PBMC or Paul et al. 2,730-cell set) — subsampling then adds no scalability information.
- The goal is quality/accuracy benchmarking of denoising rather than runtime scaling — the article notes that evaluating whether denoising improves data is difficult without ground truth.
- The method under test cannot run at the target scale at all (e.g. SAVER/scImpute on 100,000 cells take hours and may be intractable), making the runtime comparison uninformative.

## Inputs

- 1.3 million cell raw count matrix from the 10X Genomics single-cell-3prime-paper repository
- List of downsample sizes (100, 1,000, 2,000, 5,000, 10,000, 100,000 cells)
- Gene filter parameter (top 1,000 highly variable genes via Scanpy filter_genes_dispersion)

## Outputs

- Downsampled count matrices at each cell-count level (zero-expression cells/genes removed, 1,000 HVGs retained)
- Runtimes table mapping each method and subset size to wall-clock CPU runtime
- Fitted log-log scaling slope of runtime versus number of cells (runtime_scaling_results.csv)

## How to apply

Obtain the 1.3 million cell count matrix from the 10X Genomics single-cell-3prime-paper repository, then downsample it to 100, 1,000, 2,000, 5,000, 10,000 and 100,000 cells. Restrict each subset to the top 1,000 highly variable genes selected with Scanpy's filter_genes_dispersion function, and remove cells and genes with zero expression from each downsampled count matrix. Run the method under test (e.g. DCA with its default 64-32-64 three-hidden-layer architecture, NB noise model, RMSProp optimizer at learning rate 0.001 on CPU) and competitors where tractable (MAGIC, SAVER, scImpute), recording wall-clock runtime per method and subset size. The scalability analysis in the paper was performed on a server with two Intel Xeon E5-2620 2.40 GHz CPUs. Finally, fit a log-log linear regression of runtime versus number of cells and extract the slope as the scaling exponent.

## Related tools

- **Scanpy** (Provides filter_genes_dispersion to select the top 1,000 highly variable genes in each downsampled subset) — https://github.com/theislab/dca
- **DCA** (Method under test whose CPU runtime is measured across downsample sizes using its negative binomial noise model and default 64-32-64 architecture) — https://github.com/theislab/dca
- **MAGIC** (Competing denoising method timed on downsampled subsets where tractable) — https://github.com/pkathail/magic
- **SAVER** (Competing denoising method timed on downsampled subsets where tractable) — https://github.com/mohuangx/SAVER
- **scImpute** (Competing imputation method timed on downsampled subsets where tractable) — https://github.com/Vivianstats/scImpute
- **anndata** (h5ad-formatted data container for the count matrices) — https://github.com/theislab/anndata
- **10XGenomics/single-cell-3prime-paper** (Source repository for the 1.3M-cell count matrix) — http://www.github.com/10XGenomics/single-cell-3prime-paper

## Examples

```
import scanpy.api as sc; sub = sc.pp.subsample(adata, n_obs=10000, copy=True); sc.pp.filter_genes_dispersion(sub, n_top_genes=1000)
```

## Evaluation signals

- The fitted log-log slope of DCA runtime versus cell count should be close to 1 (paper reported 0.66), consistent with linear scaling.
- All six subset sizes (100 through 100,000 cells) should yield recorded runtimes with runtime monotonically increasing in cell count.
- Each downsampled matrix should be restricted to 1,000 genes (filter_genes_dispersion) and contain no all-zero cells or genes.
- DCA should complete on 100,000 cells in minutes while MAGIC, SAVER and scImpute require hours where they are tractable.
- runtime_scaling_results.csv should contain both the per-method/subset runtime table and the fitted slope.

## Limitations

- Hardware specifics are only partially specified: the scalability analysis used a server with two Intel Xeon E5-2620 2.40 GHz CPUs, so absolute runtimes depend on the machine and results may not transfer.
- Hyperparameters were not fine-tuned for any analysis in the manuscript, so the measured runtimes reflect default settings (64-32-64 layers, RMSProp, lr 0.001) rather than optimized ones.
- Competing methods may be intractable at larger subset sizes, so the runtime comparison is asymmetric across the size ladder.

## Evidence

- [discussion] 1.3M downsample ladder: "The 1.3 million cell data matrix was downsampled to 100, 1000, 2000, 5000, 10,000 and 100,000 cells"
- [discussion] HVG selection: "the top 1000 highly variable genes are selected using “filter_genes_dispersion” function of Scanpy"
- [discussion] zero-expression filter: "First, cells and genes with zero expression are removed from the count matrix."
- [abstract] linear scaling finding: "Our method scales linearly with the number of cells and can, therefore, be applied to datasets of millions of cells."
- [discussion] hardware: "Scalability analysis was performed on a server with two Intel Xeon E5-2620 2.40 GHz CPUs."
