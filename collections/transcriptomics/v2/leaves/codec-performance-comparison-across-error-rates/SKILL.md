---
name: codec-performance-comparison-across-error-rates
description: Use when comparing two or more DNA storage codec pipelines and the question
  is whether pipeline A Pareto-dominates pipeline B, and how that dominance changes
  with the injected error rate.
when_to_use_negative:
- The study concerns a single codec at a single error rate with no cross-codec or
  cross-error-rate Pareto comparison required
- The error model is a specific real sequencer channel with memory (e.g., nanopore
  via DNArSim) rather than a swept i.i.d. error model, and the goal is channel fidelity
  rather than codec optimality ranking
tools:
- name: FrameD
  role: Fault-injection simulation infrastructure used to run the RS/HEDGES/HEDGES-MSA
    pipelines, inject i.i.d. errors at multiple rates, and compute read/write densities
    for the Pareto fronts
  repo: https://github.com/dna-storage/framed
- name: DNArSim
  role: Nanopore channel simulator (Julia) used by FrameD configs to simulate the
    storage process from synthesis to basecalling when a memory-bearing error model
    is needed instead of i.i.d. errors
  repo: https://github.com/BHam-1/DNArSim
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad572
    title: 'FrameD: framework for DNA-based data storage design, verification, and
      validation'
  source_content_hash: sha256:ed1ea3a7d8e8f639c8644902eb0edb0beb455b53d5ca050809afb137f1c68446
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dna-storage/framed
  tools:
  - DNArSim
  - FrameD
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# Codec performance comparison across error rates

## Summary

Fault-injection simulation comparing DNA storage pipelines (e.g., RS, HEDGES, HEDGES-MSA) at multiple i.i.d. error rates (1%, 5%, 10%) to derive read/write density Pareto fronts and determine whether each codec is optimal at which error rate.

## When to use

Use when comparing two or more DNA storage codec pipelines and the question is whether pipeline A Pareto-dominates pipeline B, and how that dominance changes with the injected error rate.

## When NOT to use

- The study concerns a single codec at a single error rate with no cross-codec or cross-error-rate Pareto comparison required
- The error model is a specific real sequencer channel with memory (e.g., nanopore via DNArSim) rather than a swept i.i.d. error model, and the goal is channel fidelity rather than codec optimality ranking

## Inputs

- Codec pipeline configurations (RS, HEDGES, HEDGES-MSA .json configs for FrameD)
- Input data files to encode (e.g., FASTA sequence as in DNArSim example/ref.txt)
- Error model specification: i.i.d. error rates of 1%, 5%, and 10% (or DNArSim nanopore channel with memory k=6)

## Outputs

- Read/write density Pareto front per pipeline per error rate (e.g., pareto_fronts.csv)
- Compiled fault-injection statistics (fi.stats files and Pandas dataframe via db_gen.py)
- Comparison table/figure showing which pipeline is Pareto-optimal at each error rate

## How to apply

Obtain the pipeline configurations (e.g., FrameD demonstration configs from Zenodo 10.5281/zenodo.7757762) and configure FrameD fault-injection simulations for each pipeline (RS, HEDGES, HEDGES-MSA) under the i.i.d. error model at 1%, 5%, and 10% error rates, using multiple fault-injection iterations per condition. Run the simulations with FrameD's transparent parallelism (MPI plus batch-level job submission; the paper's setup spans 240 simulations on HPC/LSF, with sub-shell background jobs for commodity hardware). For each pipeline and error rate, compute read density (number of reads required for recovery) and write density, and assemble the resulting read/write density Pareto fronts into a single table (e.g., pareto_fronts.csv mirroring Fig. 4). The key rationale is that MSA inclusion is not universally beneficial: HEDGES-MSA outperforms HEDGES at high error rates (5% and 10%) while HEDGES envelops HEDGES-MSA at 1%, so the comparison must be swept across error rates rather than evaluated at one point.

## Related tools

- **FrameD** (Fault-injection simulation infrastructure used to run the RS/HEDGES/HEDGES-MSA pipelines, inject i.i.d. errors at multiple rates, and compute read/write densities for the Pareto fronts) — https://github.com/dna-storage/framed
- **DNArSim** (Nanopore channel simulator (Julia) used by FrameD configs to simulate the storage process from synthesis to basecalling when a memory-bearing error model is needed instead of i.i.d. errors) — https://github.com/BHam-1/DNArSim

## Examples

```
cd $DNASTORAGE_HOME/tools/data_analysis && python db_gen.py --path <top result path> --name fault_injection_dataframe
```

## Evaluation signals

- At 1% i.i.d. error, the HEDGES pipeline envelops HEDGES-MSA (adding MSA is not beneficial); if the reproduced front shows otherwise, recheck the configuration
- At 5% and 10% i.i.d. error, HEDGES-MSA outperforms HEDGES on the read/write density Pareto front
- RS attains the best write density at 1% error
- Each (pipeline, error rate) condition was run with multiple fault-injection iterations, and per-leaf results contain a human-readable fi.stats file
- The compiled dataframe generated by db_gen.py reproduces the qualitative ordering of the paper's Fig. 4 Pareto fronts

## Limitations

- The full paper configuration (240 simulations) is only tractable on HPC systems with an LSF scheduler; extending to SLURM/Cobalt requires implementing a new job class in tools/lsf/submit.py
- The conclusion that MSA inclusion is beneficial depends on both the error rate and the type of errors injected — it is not always beneficial and the result may not transfer to other error models
- Encoding/decoding large files may perform poorly on standard computers; sequencing-data analysis performs best on HPC with MPI installed

## Evidence

- [abstract] MSA benefit depends on error rate: "We found that the choice to include multiple sequence alignment in the pipeline is dependent on the error rate and the type of errors being injected and is not always beneficial."
- [abstract] Fault-injection test bed: "it provides a fault-injection-based test bed in which DNA storage systems can be evaluated"
- [readme] i.i.d. config files in repo: [evidence span withheld — anchor status: external_doc]
- [readme] Non-HPC examples: [evidence span withheld — anchor status: external_doc]
