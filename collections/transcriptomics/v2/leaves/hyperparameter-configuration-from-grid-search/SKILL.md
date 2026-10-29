---
name: hyperparameter-configuration-from-grid-search
description: Use when running AutoImpute.py on a pre-processed scRNA-seq count matrix
  and dataset-specific hyperparameters must be set from the paper's grid search (e.g.,
  the ~68,000-cell PBMC dataset values in Table 2).
when_to_use_negative:
- The input matrix is raw, un-normalized counts rather than the library-size-normalized,
  log2-transformed, top-1000-gene matrix produced by the R pre-processing script
- You intend to re-run the grid search itself rather than reuse the paper's tuned
  per-dataset hyperparameters
- The goal is imputation with methods other than AutoImpute — the task card reports
  that the other tested imputation methods exhausted system memory on the PBMC dataset
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: AutoImpute
  role: The autoencoder-based imputation model whose command-line hyperparameters
    (hidden_units, lambda_val, initial_learning_rate, threshold, iterations) are configured
    by this skill
  repo: github.com/divyanshu-talwar/AutoImpute
- name: TensorFlow
  role: Backend that trains the three-layer autoencoder via gradient descent with
    RMSProp using the configured hyperparameters
- name: numpy
  role: Dependency for loading and manipulating the count matrix fed to AutoImpute.py
- name: scikit-learn
  role: Dependency listed for the AutoImpute Python environment; downstream K-means
    clustering (ARI 0.2850 on PBMC) validates the imputed output
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-018-34688-x
    title: 'AutoImpute: Autoencoder based imputation of single-cell RNA-seq data'
  source_content_hash: sha256:75dea593c3adf31a0d5699a2fb11c70e9ec50970c81c582c42134a11e7888f58
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AutoImpute
  - TensorFlow
  - numpy
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# hyperparameter-configuration-from-grid-search

## Summary

Configure the AutoImpute TensorFlow autoencoder for scRNA-seq imputation using the paper's grid-selected hyperparameters (e.g., PBMC: lambda_val=1, hidden_units=2000, initial_learning_rate=1e-5, threshold=1e-3, up to 7000 iterations with RMSProp) instead of re-tuning from scratch.

## When to use

Use when running AutoImpute.py on a pre-processed scRNA-seq count matrix and dataset-specific hyperparameters must be set from the paper's grid search (e.g., the ~68,000-cell PBMC dataset values in Table 2).

## When NOT to use

- The input matrix is raw, un-normalized counts rather than the library-size-normalized, log2-transformed, top-1000-gene matrix produced by the R pre-processing script
- You intend to re-run the grid search itself rather than reuse the paper's tuned per-dataset hyperparameters
- The goal is imputation with methods other than AutoImpute — the task card reports that the other tested imputation methods exhausted system memory on the PBMC dataset

## Inputs

- Pre-processed scRNA-seq count matrix (.csv or .mat) with rows as cells and columns as genes (e.g., PBMC: ~68,000 cells x 1000 high-dispersion genes)
- Dataset-specific hyperparameter values from the paper's grid search (Table 2): lambda_val, hidden_units, initial_learning_rate, threshold, iterations

## Outputs

- Imputed expression matrix (imputed_matrix.csv) with the same dimensions as the input matrix
- Model checkpoints (e.g., checkpoints/model1.ckpt) and training-loss logs (log.txt)

## How to apply

Locate the dataset's tuned hyperparameters from the paper's grid search (Table 2); for the 10x Genomics PBMC dataset these are lambda_val=1, hidden_units=2000, initial_learning_rate=1e-5, threshold=1e-3, with training up to 7000 iterations under the RMSProp optimizer. Pass these as command-line arguments to AutoImpute.py along with --data pointing to the pre-processed .csv/.mat count matrix (log2-transformed, library-size normalized, top 1000 high-dispersion genes). The hidden_units value sets the latent space dimensions of the three-layer autoencoder, and threshold implements early stopping when the change in consecutive loss values falls below it. Rationale: these values were selected by the authors' grid search to balance reconstruction against the L2 regularization term (controlled by lambda_val), and the PBMC configuration is the one that completed on ~68,000 cells without memory exhaustion.

## Related tools

- **AutoImpute** (The autoencoder-based imputation model whose command-line hyperparameters (hidden_units, lambda_val, initial_learning_rate, threshold, iterations) are configured by this skill) — github.com/divyanshu-talwar/AutoImpute
- **TensorFlow** (Backend that trains the three-layer autoencoder via gradient descent with RMSProp using the configured hyperparameters)
- **numpy** (Dependency for loading and manipulating the count matrix fed to AutoImpute.py)
- **scikit-learn** (Dependency listed for the AutoImpute Python environment; downstream K-means clustering (ARI 0.2850 on PBMC) validates the imputed output)

## Examples

```
python AutoImpute.py --data pbmc.csv --lambda_val 1 --hidden_units 2000 --initial_learning_rate 0.00001 --threshold 0.001 --iterations 7000 --imputed_save imputed_matrix --debug True
```

## Evaluation signals

- imputed_matrix.csv exists with the same dimensions as the input (~68,000 cells x 1000 genes for PBMC)
- Training terminates via threshold early stopping or at the 7000-iteration cap, with loss values recorded in log.txt showing monotone decrease
- K-means ARI of the imputed data approximates the reported PBMC value of 0.2850 (vs 0.2687 without imputation)
- The run completes without a memory-exhaustion error, matching the paper's scalability claim for PBMC
- Command-line arguments match the Table 2 values exactly: lambda_val=1, hidden_units=2000, initial_learning_rate=1e-5, threshold=1e-3

## Limitations

- Hyperparameters are dataset-specific: the PBMC values in Table 2 need not transfer to the other eight tested datasets; the README defaults (e.g., initial_learning_rate 0.0001, threshold 0.0001) differ from the PBMC configuration
- The README documents Python 2.7 as the runtime environment with TensorFlow, which may not run unmodified on modern TensorFlow versions
- The threshold flag only triggers convergence when the change in consecutive loss values drops below it; a too-large threshold stops training prematurely

## Evidence

- [readme] hidden_units option: "--hidden_units 2000	# Size of hidden layer or latent space dimensions."
- [readme] threshold early stopping: "--threshold 0.0001	# To stop gradient descent after the change in loss function value in consecutive iterations is less than the threshold, implying convergence."
- [readme] input format: "The input to `AutoImpute.py` is a pre-processed count single cell matrix, with columns representing genes and rows representing cells."
- [abstract] method overview: "we developed an autoencoder-based sparse gene expression matrix imputation method"
