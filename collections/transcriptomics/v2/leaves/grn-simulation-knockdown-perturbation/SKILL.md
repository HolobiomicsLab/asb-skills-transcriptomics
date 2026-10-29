---
name: grn-simulation-knockdown-perturbation
description: Use when a new differential network inference method must be benchmarked
  against existing methods and no real dataset with a known ground-truth GRN is available;
  simulated knockdown perturbation data provides verifiable true edges.
when_to_use_negative:
- The analysis goal is biological discovery on real disease data (e.g. COVID-19, Crohn's
  disease) where pathway enrichment, not precision/recall against a synthetic ground
  truth, is the evaluation criterion.
- Only one network complexity setting is of interest and the per-size simulation burden
  (500 simulations, or 250 for the 500-node setting) is unnecessary.
- The method under test cannot accept the two-condition expression matrix format required
  (tab-separated, rows as genes, columns as samples, identical feature sets with a
  'Gene' first column).
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0203
tools:
- name: BoostDiff
  role: Method under test — infers differential networks from each simulated disease/control
    pair using an AdaBoost ensemble of differential trees with differential variance
    improvement splitting criterion; run with n_estimators=50
  repo: https://github.com/scibiome/boostdiff_inference
- name: dcanr
  role: Provides the SimulatorGRN one-gene-knockdown scheme used to generate ground-truth
    GRNs and paired disease/control expression data for benchmarking
- name: conda
  role: Creates the bdenv environment from environment.yml to install and run BoostDiff
    on the simulated datasets
- name: pip
  role: Installs the BoostDiff package inside the conda environment (pip install .)
- name: Python
  role: Runs the BoostDiff pipeline and benchmark evaluation scripts over the simulated
    data
- name: R
  role: Runs the comparator differential network methods implemented in R (e.g. minet/ARACNE,
    GENIE3 variants)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbae034
    title: Inference of differential gene regulatory networks using boosted differential
      trees
  source_content_hash: sha256:d27d35ea6361e4f3529edaba8b6b82a8fae445bd306d1757a9849aeacdd504d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scibiome/boostdiff_inference
  tools:
  - BoostDiff
  - Python
  - R
  - conda
  - dcanr
  - pip
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# grn-simulation-knockdown-perturbation

## Summary

Simulate ground-truth gene regulatory networks and paired disease/control expression data using the dcanr SimulatorGRN one-gene-knockdown scheme, enabling precision/recall/F1 benchmarking of differential network inference methods such as BoostDiff against comparators.

## When to use

Use when a new differential network inference method must be benchmarked against existing methods and no real dataset with a known ground-truth GRN is available; simulated knockdown perturbation data provides verifiable true edges.

## When NOT to use

- The analysis goal is biological discovery on real disease data (e.g. COVID-19, Crohn's disease) where pathway enrichment, not precision/recall against a synthetic ground truth, is the evaluation criterion.
- Only one network complexity setting is of interest and the per-size simulation burden (500 simulations, or 250 for the 500-node setting) is unnecessary.
- The method under test cannot accept the two-condition expression matrix format required (tab-separated, rows as genes, columns as samples, identical feature sets with a 'Gene' first column).

## Inputs

- Simulated ground-truth GRN adjacency (dcanr SimulatorGRN one-gene-knockdown scheme) at 50/150/300/500-node complexity settings
- Simulated disease condition expression matrix (genes x samples, 100 samples per condition)
- Simulated control condition expression matrix (genes x samples, 100 samples per condition)
- Comparator method outputs (GENIE3, ARACNE/minet, RF Difftrees, Differential GENIE3, z-score, EBcoexpress, Diffcoex, GGM, chNet)

## Outputs

- Per-method ranked edge lists evaluated on the top-100 edges
- benchmark_performance.csv with precision, recall, and F1 per method per network-size setting
- Reproduced performance pattern (e.g. RF Difftrees leading at 50 genes, BoostDiff leading at 150/300/500 genes)

## How to apply

Generate simulated ground-truth GRNs at multiple complexity settings (e.g. 50-, 150-, 300-, and 500-node networks) with the dcanr SimulatorGRN one-gene-knockdown scheme, producing disease and control expression matrices with 100 samples per condition. Run 500 simulations per network size for the 50/150/300-node settings and 250 simulations for the 500-node setting so that each condition-pair serves as one benchmark instance. Feed each simulated disease/control pair through the method under test (BoostDiff with the recommended low ensemble size n_estimators=50 to avoid overfitting) and through all comparator methods (e.g. GENIE3, ARACNE via minet, RF Difftrees, Differential GENIE3, z-score, EBcoexpress, Diffcoex, GGM, chNet). Rank predicted edges per method and evaluate inference performance on the top-100 edges against the known ground-truth GRN, computing precision, recall, and F1 per method per network-size setting. The rationale is that simulated data is the only setting where the complete true network is known, allowing fair head-to-head comparison of edge-ranking quality across network complexity regimes.

## Related tools

- **BoostDiff** (Method under test — infers differential networks from each simulated disease/control pair using an AdaBoost ensemble of differential trees with differential variance improvement splitting criterion; run with n_estimators=50) — https://github.com/scibiome/boostdiff_inference
- **dcanr** (Provides the SimulatorGRN one-gene-knockdown scheme used to generate ground-truth GRNs and paired disease/control expression data for benchmarking)
- **conda** (Creates the bdenv environment from environment.yml to install and run BoostDiff on the simulated datasets)
- **pip** (Installs the BoostDiff package inside the conda environment (pip install .))
- **Python** (Runs the BoostDiff pipeline and benchmark evaluation scripts over the simulated data)
- **R** (Runs the comparator differential network methods implemented in R (e.g. minet/ARACNE, GENIE3 variants))

## Examples

```
git clone https://github.com/gihannagalindez/boostdiff_inference.git && cd boostdiff_inference; conda env create --name bdenv --file environment.yml; conda activate bdenv; pip install .
```

## Evaluation signals

- benchmark_performance.csv reproduces the reported pattern: RF Difftrees is the top performer in the 50-gene setting while BoostDiff is the best performing method in the 150-, 300-, and 500-gene settings
- Each simulated condition pair used 100 samples per condition via the dcanr SimulatorGRN one-gene-knockdown scheme
- 500 simulations per network size for 50/150/300-node settings; 250 simulations for the 500-node setting
- BoostDiff was run with the recommended low ensemble size n_estimators=50 to avoid overfitting
- Precision, recall, and F1 computed per method per setting, evaluated on the top-100 ranked edges against the ground-truth GRN

## Limitations

- Simulated networks only cover the tested complexity range (50–500 genes); performance on larger or denser networks is not established by this benchmark.
- The one-gene-knockdown perturbation scheme defines the simulation regime; other perturbation types may favor different methods.
- Benchmark conclusions are relative to the nine specific comparator methods tested; new methods may change the ranking.
- For real transcriptomics datasets, BoostDiff recommends a relatively low number of base learners (n_estimators=50) to avoid overfitting; the simulation benchmark reflects this configuration.

## Evidence

- [methods] simulation scheme: [evidence span withheld — anchor status: not_found]
- [results] performance pattern: [evidence span withheld — anchor status: not_found]
- [readme] ensemble size: "For real transcriptomics datasets, we recommend to set a relatively low number of base learners, e.g. set `n_estimators=50`, so that 50 adaptively boosted differential trees will be built to avoid"
- [abstract] benchmark claim: "BoostDiff outperforms existing differential network methods on simulated data evaluated in four different complexity settings."
- [readme] installation: "conda env create --name bdenv --file environment.yml
conda activate bdenv

pip install ."
