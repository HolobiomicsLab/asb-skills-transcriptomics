---
name: negative-binomial-cell-count-simulation
description: Use when you need to validate the false-positive rate of a proportion
  analysis tool (e.g., Scanpro) on simulated null data with two conditions and unchanged
  cluster proportions, with or without real replicates.
when_to_use_negative:
- You are analyzing real experimental data for proportion shifts rather than benchmarking
  a PA tool's false-positive control — run scanpro directly on the AnnData object
  instead
- Clusters of interest are supported by less than 2% of all cells, since benchmarking
  found such small clusters produce unstable results and bootstrapping has limited
  benefits for rare cell types
- The dataset already has sufficient biological replicates for all conditions, so
  pseudo-replicate bootstrapping simulation is unnecessary
edam_operation: http://edamontology.org/operation_0372
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Scanpro
  role: Provides sim_scanpro for generating negative-binomial null-simulation datasets
    and the scanpro function (with iteration-based bootstrapping) whose false-positive
    accuracy is being benchmarked
  repo: https://github.com/loosolab/scanpro
- name: Python
  role: Execution environment; Scanpro integrates into widely accepted Python frameworks
    and natively handles AnnData objects
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-024-66381-7
    title: Scanpro is a tool for robust proportion analysis of single-cell resolution
      data
  source_content_hash: sha256:44227c425750228157f8ed9e6edc7a63eff4d9be36827c82a80a0f48e7237604
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/loosolab/scanpro
  tools:
  - Python
  - Scanpro
  edam_operation: http://edamontology.org/operation_0372
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# negative-binomial-cell-count-simulation

## Summary

Generates synthetic null single-cell datasets with sim_scanpro from the Scanpro package, drawing per-cluster cell counts from a negative-binomial distribution to benchmark false-positive control of proportion analysis (PA) on replicated and pseudo-replicated data.

## When to use

Use when you need to validate the false-positive rate of a proportion analysis tool (e.g., Scanpro) on simulated null data with two conditions and unchanged cluster proportions, with or without real replicates.

## When NOT to use

- You are analyzing real experimental data for proportion shifts rather than benchmarking a PA tool's false-positive control — run scanpro directly on the AnnData object instead
- Clusters of interest are supported by less than 2% of all cells, since benchmarking found such small clusters produce unstable results and bootstrapping has limited benefits for rare cell types
- The dataset already has sufficient biological replicates for all conditions, so pseudo-replicate bootstrapping simulation is unnecessary

## Inputs

- Null-simulation parameter specification: number of conditions (2), clusters (5), replicate counts (2–14), negative-binomial parameters (mu=20000, dispersion=20), ~20,000 cells per condition
- AnnData objects (with sample, cluster, and condition columns) produced by sim_scanpro for each simulated dataset
- Scanpro run settings: samples_col, clusters_col, conds_col; samples_col omitted/None to trigger bootstrapping on unreplicated data

## Outputs

- 100 null-simulation AnnData datasets per replicate count, each with unchanged cluster proportions across the two conditions
- Per-setting accuracy values (fraction of truly unchanged clusters called non-significant), e.g. null_simulation_accuracy.csv with accuracy per replicate count and pseudo-replicate count
- Expected benchmark targets: replicated accuracy 0.98–0.99; bootstrapped accuracy matching the replicated result from 8 pseudo-replicates upward

## How to apply

Call the sim_scanpro function exported by the scanpro package to generate null-simulation datasets — in the Scanpro benchmark, 100 datasets per replicate count from 2 to 14, each with two conditions and five clusters whose proportions are unchanged, with per-cluster cell counts drawn from a negative-binomial distribution (mu=20000, dispersion=20) yielding roughly 20,000 cells per condition. For the replicated setting, run the scanpro function (from scanpro.scanpro import scanpro) on each simulated dataset, passing the sample, cluster, and condition column annotations. For the unreplicated setting, omit the samples column so Scanpro's iteration-based bootstrapping creates pseudo-replicates, and rerun scanpro for increasing pseudo-replicate counts. Score accuracy as the fraction of truly unchanged clusters correctly called non-significant, aggregated over the 100 simulated datasets per replicate count, and write the accuracy values per (pseudo-)replicate count to a results CSV. Rationale: a null simulation with unchanged proportions isolates false-positive control, which the paper's benchmark measured at 0.98–0.99 accuracy on replicated data.

## Related tools

- **Scanpro** (Provides sim_scanpro for generating negative-binomial null-simulation datasets and the scanpro function (with iteration-based bootstrapping) whose false-positive accuracy is being benchmarked) — https://github.com/loosolab/scanpro
- **Python** (Execution environment; Scanpro integrates into widely accepted Python frameworks and natively handles AnnData objects)

## Examples

```
from scanpro.scanpro import scanpro; out = scanpro(adata, samples_col='sample', clusters_col='cluster', conds_col='conds')
```

## Evaluation signals

- Accuracy for the replicated null simulation falls in the 0.98–0.99 range, indicating good false-positive control
- Accuracy is computed as the fraction of truly unchanged clusters correctly called non-significant, aggregated over 100 simulated datasets per replicate count
- Bootstrapped (pseudo-replicate) accuracy converges to the replicated result from 8 pseudo-replicates upward
- Each simulated dataset contains two conditions and five clusters with unchanged proportions and ~20,000 cells per condition, drawn from a negative-binomial distribution with mu=20000 and dispersion=20
- Results CSV contains one accuracy row per replicate count (2–14) and per pseudo-replicate count

## Limitations

- Null simulations only test false-positive control on unchanged proportions; they do not validate power to detect real proportion shifts
- Bootstrapping has limited benefits when handling rare cell types, and clusters supported by less than 2% of all cells produce unstable results
- Lack of replicates limits drawing definitive conclusions about variability; bootstrapped results should be interpreted with caution and require additional experimental validation

## Evidence

- [intro] we here present Scanpro (Single-cell analysis of proportions). Scanpro implements the linear regression approach proposed by propeller, and extends this by a bootstrapping functionality to internally: "we here present Scanpro (Single-cell analysis of proportions). Scanpro implements the linear regression approach proposed by propeller, and extends this by a bootstrapping functionality to internally"
- [discussion] From the benchmarking, we found clusters supported by less than 2% of all cells in an experiment to produce unstable results: "From the benchmarking, we found clusters supported by less than 2% of all cells in an experiment to produce unstable results"
- [readme] If samples_col is not given or set to None, the dataset is assumed to be not replicated and scanpro will run the bootstrapping method.: [evidence span withheld — anchor status: external_doc]
- [methods] from scanpro.scanpro import scanpro

    out = scanpro(adata, samples_col='sample', clusters_col='cluster', conds_col='conds'): "from scanpro.scanpro import scanpro

    out = scanpro(adata, samples_col='sample', clusters_col='cluster', conds_col='conds')"
