---
name: single-cell-rna-seq-count-simulation
description: Use when you need controlled scRNA-seq test data with known per-cell
  circadian phases to benchmark phase-inference algorithms or study how library size
  and sampling design affect phase-estimation error.
when_to_use_negative:
- The goal is inference on real experimental droplet- or plate-based scRNA-seq data,
  not controlled benchmarking — apply Tempo directly instead
- The task concerns cell-cycle phase ordering rather than the circadian cycle; the
  paper notes existing cell-cycle methods perform poorly on circadian data and the
  generative model here targets circadian CCGs
- Non-transcriptomic single-cell modalities are the input, since Tempo's generative
  model is defined over gene-expression counts
edam_operation: http://edamontology.org/operation_3566
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Tempo
  role: Provides the Negative Binomial generative model and simulation utilities used
    to generate count matrices with known ground-truth phases; also the method under
    benchmark
  repo: github.com/bauerbach95/tempo
- name: anndata
  role: Container format for simulated count matrices passed to Tempo and baselines
- name: numpy
  role: Array math for generating simulated counts and computing phase errors
- name: pandas
  role: Tabulation of median phase errors per method and condition
- name: scanpy
  role: scRNA-seq preprocessing (scanpy>=1.6) within the Tempo environment
- name: scikit-learn
  role: Baseline PCA phase estimates on the simulated count matrices
- name: PyTorch
  role: Backend for Tempo's Bayesian variational inference when fitting simulated
    data
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-34185-w
    title: 'Tempo: an unsupervised Bayesian algorithm for circadian phase inference
      in single-cell transcriptomics'
  source_content_hash: sha256:51ac6ebec1ae7196484932285b13e657142883430dc21b9ca70b60e91ffde418
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PyTorch
  - Python
  - Tempo
  - anndata
  - numpy
  - pandas
  - scanpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3566
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# single-cell-rna-seq-count-simulation

## Summary

Simulate single-cell RNA-seq count matrices with known ground-truth circadian phases using Tempo's Negative Binomial generative model, enabling benchmarking of phase-inference methods (Tempo vs. Cyclops, Cyclum, PCA) against a theoretical optimum.

## When to use

Use when you need controlled scRNA-seq test data with known per-cell circadian phases to benchmark phase-inference algorithms or study how library size and sampling design affect phase-estimation error.

## When NOT to use

- The goal is inference on real experimental droplet- or plate-based scRNA-seq data, not controlled benchmarking — apply Tempo directly instead
- The task concerns cell-cycle phase ordering rather than the circadian cycle; the paper notes existing cell-cycle methods perform poorly on circadian data and the generative model here targets circadian CCGs
- Non-transcriptomic single-cell modalities are the input, since Tempo's generative model is defined over gene-expression counts

## Inputs

- Simulation grid specification (cell counts 500-5000, median UMI counts 3000-20000, unsynchronized or ZT0/6/12/18 conditions)
- Core clock gene list (~20 circadian clock genes)
- Ground-truth circadian phases per cell

## Outputs

- Simulated scRNA-seq count matrices (anndata/numpy/pandas format) with known per-cell circadian phase labels
- Per-method absolute phase-error distributions (eCDFs, e.g., reproducing Fig. 2a/d panels)
- Median phase-error tables per method and condition

## How to apply

Define a simulation grid over unsynchronized and ZT0/6/12/18 time-course conditions, spanning 500-5000 cells and median UMI counts of 3000-20000. Generate count matrices from Tempo's Negative Binomial generative model with known ground-truth phases using the package's simulation utilities in Python with numpy, pandas, and anndata. Run Tempo (Bayesian variational inference, pytorch backend) on each simulated dataset, restricted to core clock genes (~20 moderately expressed genes) and with all genes, alongside baseline methods (Cyclops, Cyclum, PCA). Compute absolute phase point-estimate errors against the simulated ground truth and compare error eCDFs to the maximum-likelihood optimum under the true generative model. Before benchmarking, verify the installation by running the packaged test, which must print 'SUCCESSFULLY FINISHED'.

## Related tools

- **Tempo** (Provides the Negative Binomial generative model and simulation utilities used to generate count matrices with known ground-truth phases; also the method under benchmark) — github.com/bauerbach95/tempo
- **anndata** (Container format for simulated count matrices passed to Tempo and baselines)
- **numpy** (Array math for generating simulated counts and computing phase errors)
- **pandas** (Tabulation of median phase errors per method and condition)
- **scanpy** (scRNA-seq preprocessing (scanpy>=1.6) within the Tempo environment)
- **scikit-learn** (Baseline PCA phase estimates on the simulated count matrices)
- **PyTorch** (Backend for Tempo's Bayesian variational inference when fitting simulated data)

## Examples

```
git clone https://github.com/bauerbach95/tempo; cd tempo; source install.sh; conda activate tempo; python run_test.py
```

## Evaluation signals

- The packaged validation test (python run_test.py) completes and prints 'SUCCESSFULLY FINISHED'
- Tempo's cell phase point-estimate error eCDF lies only slightly worse than the maximum-likelihood optimum computed under the true generative model
- Baseline methods (Cyclops, Cyclum, PCA) show degrading error eCDFs at small library sizes, matching the reported ordering
- Recomputed absolute phase errors against simulated ground truth reproduce Fig. 2a/d panel patterns and median-error ordering across conditions
- Simulated datasets fall within the intended grid (500-5000 cells; median UMI counts 3000-20000) for each ZT condition

## Limitations

- Simulations assume CCG expression means follow 24 h sinusoidal patterns; the authors suggest this assumption may underlie 'de novo cycler' issues and that more flexible waveform modeling is needed
- Tempo does not explicitly model technical effects, which becomes more necessary for data collected as multiple samples over time
- Subpopulation-specific gene amplitude variation is not explicitly modeled; the authors propose mapping cell-state embeddings to sinusoidal parameters as an ideal solution
- Tempo requires python >= 3.8, pytorch>=1.9.0, and mini-conda for installation; tested on macOS 10.14.5/12.2.1 and CentOS Linux 7 with >=8GB RAM

## Evidence

- [other] Tempo's cell phase point estimate error eCDFs are only slightly worse than the optimum (maximum-likelihood phases under the true generative model).: [evidence span withheld — anchor status: not_found]
- [other] Define the simulation grid: unsynchronized and ZT0/6/12/18 time-course conditions, with cell counts spanning 500-5000 and median UMI counts spanning 3000-20000.: [evidence span withheld — anchor status: not_found]
- [methods] Tempo requires python >= 3.8 and depends on the following python packages: "Tempo requires python >= 3.8 and depends on the following python packages"
- [readme] Successful completion of the test will yield a message "SUCCESSFULLY FINISHED".: "Successful completion of the test will yield a message "SUCCESSFULLY FINISHED"."
- [abstract] we introduce Tempo, a Bayesian variational inference approach that incorporates domain knowledge of the clock and quantifies phase estimation uncertainty: "we introduce Tempo, a Bayesian variational inference approach that incorporates domain knowledge of the clock and quantifies phase estimation uncertainty"
