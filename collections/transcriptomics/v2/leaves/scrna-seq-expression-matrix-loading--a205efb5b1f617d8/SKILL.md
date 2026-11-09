---
name: scrna-seq-expression-matrix-loading
description: Use when the input is a .mat expression matrix from the SERM benchmark
  data (e.g. data_observed_10perc.mat with 12162 cells x 2422 genes) and it must be
  loaded into a Python array before imputation or timing benchmarks.
when_to_use_negative:
- Input is already an in-memory numpy array or AnnData/h5ad object — loadmat and the
  .mat unpacking step are unnecessary
- The expression matrix axes are genes-by-cells; axes must be transposed first since
  SERM and the ROI parameters assume rows are cells and columns are genes
- Raw unfiltered count data is the target, since the benchmark reference data were
  pre-filtered by choosing 12,162 cells and 2,422 genes with high expression
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: self-consistent expression recovery machine (SERM)
  role: 'Downstream consumer: the loaded matrix is passed directly to srm.serm() with
    ROI size [6100, 1250], percent overlap 0.5, seed 0'
  repo: https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
- name: MAGIC
  role: Baseline imputation method run on the same loaded matrix for runtime comparison
  repo: https://github.com/pkathail/magic
- name: mcImpute
  role: Baseline MATLAB matrix-completion imputation method run on the identical loaded
    matrix
  repo: https://github.com/aanchalMongia/McImpute_scRNAseq
- name: DeepImpute
  role: Baseline deep-learning imputation method run on the identical loaded matrix
  repo: https://github.com/lanagarmire/deepimpute
- name: SAUCIE
  role: Baseline TensorFlow autoencoder imputation method run on the identical loaded
    matrix
  repo: https://github.com/KrishnaswamyLab/SAUCIE
- name: SciPy
  role: Provides scipy.io.loadmat used to read the .mat expression files into Python
    arrays
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-34595-w
    title: Leveraging data-driven self-consistency for high-fidelity gene expression
      recovery
  source_content_hash: sha256:5efefb952da17c43b18578e301c9ffca48d38bc39684274ee782e13b0606e5e5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
  tools:
  - DeepImpute
  - MAGIC
  - MATLAB
  - SAUCIE
  - SciPy
  - mcImpute
  - self-consistent expression recovery machine (SERM)
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# scrna-seq-expression-matrix-loading

## Summary

Load scRNA-seq gene expression matrices from MATLAB .mat files (e.g. data_observed_10perc.mat) via scipy.io.loadmat so that rows are cells and columns are genes, as a prerequisite step for running SERM or baseline imputation methods (MAGIC, mcImpute, DeepImpute, SAUCIE).

## When to use

Use when the input is a .mat expression matrix from the SERM benchmark data (e.g. data_observed_10perc.mat with 12162 cells x 2422 genes) and it must be loaded into a Python array before imputation or timing benchmarks.

## When NOT to use

- Input is already an in-memory numpy array or AnnData/h5ad object — loadmat and the .mat unpacking step are unnecessary
- The expression matrix axes are genes-by-cells; axes must be transposed first since SERM and the ROI parameters assume rows are cells and columns are genes
- Raw unfiltered count data is the target, since the benchmark reference data were pre-filtered by choosing 12,162 cells and 2,422 genes with high expression

## Inputs

- MATLAB .mat file containing the observed dropout-affected expression matrix (e.g. data_observed_10perc.mat, variable 'data_observed', 12162 cells x 2422 genes)
- MATLAB .mat file containing the dropout-free reference expression matrix (data_reference.mat, variable 'data_reference')
- Data files downloadable from the authors' Google Drive folders

## Outputs

- In-memory gene expression matrix (rows = cells, columns = genes) suitable for srm.serm(dataObs, ROIsize, percOL, randomize=False)
- Optionally, wall-clock timing records (time.time() deltas) for downstream runtime comparison tables

## How to apply

Read the .mat files with scipy.io.loadmat and extract the variable inside (e.g. 'data_reference' from data_reference.mat, 'data_observed' from data_observed_10perc.mat); verify the matrix shape matches the expected cell-by-gene dimensions (12162 cells x 2422 genes for the 10% sampling-efficiency bone-marrow benchmark). Keep rows as cells and columns as genes, since SERM's ROI size is specified in these axes (e.g. [6100, 1250] for a 2x2 ROI partition). Wrap the loading step with time.time() only around the imputation call (srm.serm), not the loading, so runtime comparisons against MAGIC, mcImpute, DeepImpute, and SAUCIE are fair. Reference and observed data must be downloaded from the authors' Google Drive links, and the observed data are simulated from reference data by sampling-efficiency loss (e.g. alpha <- rgamma(ncol(data.filt), 10, 100) followed by Poisson sampling at 10% efficiency).

## Related tools

- **self-consistent expression recovery machine (SERM)** (Downstream consumer: the loaded matrix is passed directly to srm.serm() with ROI size [6100, 1250], percent overlap 0.5, seed 0) — https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
- **MAGIC** (Baseline imputation method run on the same loaded matrix for runtime comparison) — https://github.com/pkathail/magic
- **mcImpute** (Baseline MATLAB matrix-completion imputation method run on the identical loaded matrix) — https://github.com/aanchalMongia/McImpute_scRNAseq
- **DeepImpute** (Baseline deep-learning imputation method run on the identical loaded matrix) — https://github.com/lanagarmire/deepimpute
- **SAUCIE** (Baseline TensorFlow autoencoder imputation method run on the identical loaded matrix) — https://github.com/KrishnaswamyLab/SAUCIE
- **SciPy** (Provides scipy.io.loadmat used to read the .mat expression files into Python arrays)

## Examples

```
import scipy.io as sio, serm as srm, random, time; dataObs = sio.loadmat('data_observed_10perc.mat')['data_observed']; t=time.time(); random.seed(0); out_SERM = srm.serm(dataObs,[6100,1250],0.5,randomize=False); print('Required time:', str(time.time()-t),'second')
```

## Evaluation signals

- Loaded variable shape equals 12162 x 2422 (cells x genes) for the data_observed_10perc.mat benchmark
- Matrix orientation is cells-by-rows/genes-by-columns, consistent with ROI size [6100, 1250] dividing the matrix into 4 (2x2) ROIs
- Wall-clock timing (time.time()) excludes the loadmat step and surrounds only the imputation call, enabling a fair >=10x speedup comparison
- The imputed output from srm.serm has the same dimensions as the observed input matrix
- Random seed set to 0 (random.seed(0)) before running for reproducibility

## Limitations

- SERM learns the gene expression model from a small fraction of the data, which is a challenge for small genomic datasets where the estimated distribution may diverge from the actual data distribution
- Reference data were created by choosing only cells and genes with high expression and dropouts were simulated rather than observed experimentally, so loaded matrices represent simulated dropout conditions
- SERM assumes ROI coverage is sufficiently large that the captured distribution is representative of the entire matrix, which constrains acceptable matrix dimensions

## Evidence

- [readme] scipy.io.loadmat usage for .mat expression files: "The following two code lines read the reference data
a=sio.loadmat('data_reference.mat')
dataRef=a['data_reference']"
- [readme] Matrix orientation is cells x genes: "The gene expression data should be in matrix format (rows are cells and collumns are genes)."
- [readme] Benchmark dimensions and provenance of the filtered reference dataset: "Following Huang et al
(Nature Methods 15, 539–542 (2018)), we choose 12,162 cells and 2,422 genes with
high expression to create a reference dataset."
- [readme] Simulated sampling efficiency creating observed data: "The observed data were generated from the reference data by simulating efficiency loss that introduces zeros"
- [readme] Timing methodology for runtime benchmarking: "t=time.time() # For keeping record of computational time
random.seed(0) # Setting the random number generator seed to 0."
- [abstract] Order-of-magnitude efficiency claim motivating fair timing: "We show that SERM improves the accuracy of gene imputation with orders of magnitude enhancement in computational efficiency"
