---
name: imputation-benchmark-comparison
description: Use when a dropout-affected scRNA-seq matrix and its dropout-free reference
  counterpart are available, and you need to rank competing imputation techniques
  by recovery accuracy and computational efficiency.
when_to_use_negative:
- The dataset is a small genomic dataset, where SERM's distribution estimated from
  a small data fraction may diverge from the actual data distribution and yield inaccurate
  imputation (per the paper's discussion)
- Only a single noisy matrix without a dropout-free reference or simulated-dropout
  ground truth exists, making Pearson-recovery evaluation impossible
- The observed matrix contains experimentally observed dropouts rather than simulated
  sampling-efficiency loss, breaking the reference-vs-observed benchmark design used
  in this paper
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_3308
tools:
- name: self-consistent expression recovery machine (SERM)
  role: Primary method under benchmark; called as srm.serm(dataObs, ROIsize, percOL)
    with randomized seed 0, timed around the call
  repo: https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
- name: MAGIC
  role: Baseline imputation method run with default configurations on the same observed
    matrices for comparison
  repo: https://github.com/pkathail/magic
- name: mcImpute
  role: Baseline matrix-completion imputation method downloaded from its repository
    and run with default settings for timing and accuracy comparison
  repo: https://github.com/aanchalMongia/McImpute_scRNAseq
- name: DeepImpute
  role: Baseline deep-learning imputation method downloaded from its repository and
    run with default settings for comparison
  repo: https://github.com/lanagarmire/deepimpute
- name: SAUCIE
  role: Baseline autoencoder imputation method downloaded from its repository and
    run with default settings for comparison
  repo: https://github.com/KrishnaswamyLab/SAUCIE
provenance:
  source_task_ids:
  - task_006
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-34595-w
    title: Leveraging data-driven self-consistency for high-fidelity gene expression
      recovery
  source_content_hash: sha256:f805bfbba82ff407e84e56fc60f47c3162cb500e9e067b157bf4be9850014d18
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
  tools:
  - DeepImpute
  - MAGIC
  - MATLAB
  - PHATE
  - SAUCIE
  - SciPy
  - Slingshot
  - TSCAN
  - mcImpute
  - self-consistent expression recovery machine (SERM)
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 24
---

# imputation-benchmark-comparison

## Summary

Benchmark scRNA-seq imputation methods (e.g., SERM against MAGIC, mcImpute, AutoImpute, DeepImpute, SAUCIE) on the same dropout-affected matrices, comparing recovery accuracy (Pearson coefficients, cluster quality indices) and wall-clock runtime against the reference data.

## When to use

Use when a dropout-affected scRNA-seq matrix and its dropout-free reference counterpart are available, and you need to rank competing imputation techniques by recovery accuracy and computational efficiency.

## When NOT to use

- The dataset is a small genomic dataset, where SERM's distribution estimated from a small data fraction may diverge from the actual data distribution and yield inaccurate imputation (per the paper's discussion)
- Only a single noisy matrix without a dropout-free reference or simulated-dropout ground truth exists, making Pearson-recovery evaluation impossible
- The observed matrix contains experimentally observed dropouts rather than simulated sampling-efficiency loss, breaking the reference-vs-observed benchmark design used in this paper

## Inputs

- Reference (dropout-free) gene expression matrix, cells x genes, in .mat format (e.g., data_reference.mat)
- Observed dropout-affected matrix created at a controlled sampling efficiency (e.g., data_observed_10perc.mat, 12162 cells x 2422 genes)
- Cell type labels for clustering evaluation (e.g., get_numeric_labels annotations)
- Baseline method implementations: MAGIC (pkathail/magic), mcImpute, DeepImpute, SAUCIE

## Outputs

- Imputed gene expression matrix per method (SERM out_SERM, plus MAGIC/mcImpute/DeepImpute/SAUCIE outputs)
- Per-gene Pearson coefficients of imputed vs. reference data per method and sampling efficiency
- Cluster quality indices (accuracy, NMI, AR, Hubert) with mean +/- SD across n=1000 k-means initializations
- Per-method wall-clock runtime table with SERM speedup ratios

## How to apply

Load the reference and observed (dropout-affected) expression matrices via scipy.io.loadmat (e.g., data_observed_10perc.mat with 12162 cells x 2422 genes at 10% sampling efficiency). Run each imputation method on the identical observed matrix: SERM with ROI size [6100,1250] (2x2 ROI partition), 50% overlap, random seed 0, and the baselines MAGIC, mcImpute, DeepImpute, and SAUCIE from their official repositories with default configurations. Wrap each method call in time.time() to record wall-clock runtime. Evaluate accuracy by computing mean Pearson coefficients between imputed and reference data per gene, and cluster quality indices (accuracy, NMI, AR, Hubert) over k-means on UMAP/t-SNE embeddings (n=1000 k-means initializations). Aggregate per-method runtimes into a comparison table and compute SERM's speedup ratio; validation is confirming >=10x speedup versus baselines and >=20% improvement in mean Pearson coefficient, or that a stated sampling efficiency reproduces reported values (e.g., 0.15-0.65 Pearson at 0.1% downsampling).

## Related tools

- **self-consistent expression recovery machine (SERM)** (Primary method under benchmark; called as srm.serm(dataObs, ROIsize, percOL) with randomized seed 0, timed around the call) — https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
- **MAGIC** (Baseline imputation method run with default configurations on the same observed matrices for comparison) — https://github.com/pkathail/magic
- **mcImpute** (Baseline matrix-completion imputation method downloaded from its repository and run with default settings for timing and accuracy comparison) — https://github.com/aanchalMongia/McImpute_scRNAseq
- **DeepImpute** (Baseline deep-learning imputation method downloaded from its repository and run with default settings for comparison) — https://github.com/lanagarmire/deepimpute
- **SAUCIE** (Baseline autoencoder imputation method downloaded from its repository and run with default settings for comparison) — https://github.com/KrishnaswamyLab/SAUCIE

## Examples

```
import scipy.io as sio, serm as srm, time, random; dataObs=sio.loadmat('data_observed_10perc.mat')['data_observed']; t=time.time(); random.seed(0); out_SERM=srm.serm(dataObs,[6100,1250],0.5,randomize=False); print('Required time:',time.time()-t,'second')
```

## Evaluation signals

- Recorded runtime for SERM on the 0.6M-cell x 30,000-gene benchmark completes under 18 minutes while MAGIC/mcImpute/AutoImpute/DeepImpute fail to finish within 3 days (>=10x speedup)
- Per-gene Pearson coefficients of SERM-imputed vs. reference data exceed 0.9 for most genes at 5% sampling efficiency, versus <=0.4 for most other methods
- SERM achieves the best cluster quality indices (accuracy, NMI, AR, Hubert) across benchmark datasets; scVI is the runner-up
- Zero-count similarity: the number of zeros in SERM-imputed data should closely match the reference data's zero count
- Percent improvement in mean Pearson coefficient of imputed vs. observed data for SERM should exceed the eight competing techniques at each sampling efficiency (0.1%-10%)

## Limitations

- SERM learns the gene expression model from a small fraction of the data, so for small genomic datasets the estimated ROI distribution may diverge from the actual distribution and produce inaccurate imputation values
- Batch effects are not considered explicitly in the SERM formulation, although z-score integration is used to combine imputed batches
- The benchmark reference is itself created by filtering to cells/genes with high expression and dropouts are simulated (e.g., Poisson sampling following alpha <- rgamma(ncol(data.filt), 10, 100)), not experimentally observed
- Accuracy of ROI-based imputation assumes ROI coverage is sufficiently large that the captured distribution is representative of the entire matrix

## Evidence

- [results] SERM analyzed a dataset of 0.6 million cells and 30,000 genes in under 18 minutes on an Intel Core i9 PC with 64GB RAM: [evidence span withheld — anchor status: not_found]
- [readme] Timing protocol: "t=time.time() # For keeping record of computational time
random.seed(0) # Setting the random number generator seed to 0."
- [readme] Pearson improvement metric: "Percent improvement in mean Pearson coefficient of the imputed data (in comparison to the observed data) by eight different techniques for cellular taxonomy  dataset."
- [readme] Sampling efficiency range: "The sampling efficiencies (0.1%-10%) to create the observed data are shown in x-axis."
- [results] Benchmark source data: "Following Huang et al (Nature Methods 15, 539–542 (2018)), we choose 12,162 cells and 2,422 genes with high expression to create a reference dataset."
