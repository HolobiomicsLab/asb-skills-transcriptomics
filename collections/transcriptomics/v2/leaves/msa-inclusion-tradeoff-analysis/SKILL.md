---
name: msa-inclusion-tradeoff-analysis
description: 'Use when deciding whether MSA belongs in a DNA storage pipeline: you
  must compare HEDGES-MSA vs MSA-only configurations across error rates and error
  types to see where MSA helps or hurts.'
when_to_use_negative:
- Only a single error rate or a single error type is tested — the tradeoff cannot
  be characterized, since MSA inclusion 'is dependent on the error rate and the type
  of errors being injected'
- The goal is functional encoding/decoding testing of a pipeline rather than a design
  decision about whether the MSA stage is worth its density cost
- The error model does not represent the deployment channel (e.g. conclusions drawn
  only under i.i.d. errors will not transfer to nanopore's burst-error profile)
edam_operation: http://edamontology.org/operation_2945
tools:
- name: FrameD
  role: Fault-injection test bed used to simulate the pipelines under each error model,
    with transparent parallelism (MPI + batch-level LSF jobs) for large read counts
    and many iterations; db_gen.py and notebooks/framed.ipynb compile results into
    density figures
  repo: github.com/dna-storage/framed
- name: DNArSim
  role: Julia simulator providing the nanopore channel model with memory (k=6) used
    as the error-injection source for the nanopore condition of the tradeoff analysis
  repo: github.com/BHam-1/DNArSim
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad572
    title: 'FrameD: framework for DNA-based data storage design, verification, and
      validation'
  source_content_hash: sha256:c13ef888c5ddb1e16cfcaafff1cdce84b9fd8435a7932bf6755ea3d1456ca782
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DNArSim
  - FrameD
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# msa-inclusion-tradeoff-analysis

## Summary

Evaluates whether multiple sequence alignment (MSA) should be included in a DNA storage pipeline by running FrameD fault-injection simulations across error-rate settings and error types (i.i.d. vs DNArSim nanopore), then checking whether MSA-only configurations survive on the cross-pipeline read/write density Pareto frontier.

## When to use

Use when deciding whether MSA belongs in a DNA storage pipeline: you must compare HEDGES-MSA vs MSA-only configurations across error rates and error types to see where MSA helps or hurts.

## When NOT to use

- Only a single error rate or a single error type is tested — the tradeoff cannot be characterized, since MSA inclusion 'is dependent on the error rate and the type of errors being injected'
- The goal is functional encoding/decoding testing of a pipeline rather than a design decision about whether the MSA stage is worth its density cost
- The error model does not represent the deployment channel (e.g. conclusions drawn only under i.i.d. errors will not transfer to nanopore's burst-error profile)

## Inputs

- FrameD pipeline configuration .json files (examples/iid and examples/DNArSim) describing the storage pipelines to fault-inject
- DNArSim nanopore channel model settings (input FASTA reference sequence, e.g. example/ref.txt; number of reads -n; channel memory -k, fixed to k=6 by default)
- Error-rate sweep specification for fault injection iterations
- Raw fault-injection output leaf directories containing 'fi.stats' files and JSON statistics

## Outputs

- Pandas dataframe (fault_injection_dataframe) aggregating all fault-injection results per configuration, generated via db_gen.py
- Read density and write density values for each pipeline configuration (HEDGES-MSA and MSA-only variants)
- Cross-pipeline read/write density Pareto frontier (Fig. 5-style plot, e.g. from notebooks/framed.ipynb)
- Composition verdict: which pipelines' configurations appear on the frontier under each error model

## How to apply

Configure FrameD's fault-injection test bed for the storage pipelines of interest, using both the i.i.d. error model and the DNArSim nanopore channel model (Julia, channel memory k=6, FASTQ output without scores) to inject errors across a sweep of error-rate settings. Exploit FrameD's transparent parallelism — MPI plus batch-level parallelism (IBM LSF), ~240 simulations launched from the .json configs in examples/iid and examples/DNArSim — since many sequencing reads and many fault-injection iterations are required. After runs finish, compile per-run 'fi.stats' statistics with db_gen.py into a Pandas dataframe, then record read density and write density for every pipeline configuration, including HEDGES-MSA and MSA-only variants. Compute the cross-pipeline Pareto frontier over read/write density and inspect its composition: under nanopore's burst-error-dominated profile, verify whether any MSA-only configuration appears on the frontier. Compare the frontier composition between error models to conclude whether MSA inclusion is beneficial for the given error rate and error type, since the article finds this choice 'is not always beneficial'.

## Related tools

- **FrameD** (Fault-injection test bed used to simulate the pipelines under each error model, with transparent parallelism (MPI + batch-level LSF jobs) for large read counts and many iterations; db_gen.py and notebooks/framed.ipynb compile results into density figures) — github.com/dna-storage/framed
- **DNArSim** (Julia simulator providing the nanopore channel model with memory (k=6) used as the error-injection source for the nanopore condition of the tradeoff analysis) — github.com/BHam-1/DNArSim

## Examples

```
python db_gen.py --path <top result path> --name fault_injection_dataframe
```

## Evaluation signals

- Every point on the cross-pipeline read/write density Pareto frontier under the nanopore (DNArSim) model belongs to HEDGES-MSA configurations, and no MSA-only configuration appears on that frontier
- A 'fi.stats' file exists in every leaf fault-injection directory, and db_gen.py produces a dataframe covering all pipeline configurations in the sweep
- Frontier composition differs between the i.i.d. and DNArSim nanopore error models, confirming the MSA decision is error-rate- and error-type-dependent
- Reproduced densities match the paper's figures when regenerating them from notebooks/framed_DNArSim.pickle and notebooks/framed_iid.pickle via framed.ipynb

## Limitations

- The full configuration sweep (240 simulations) is only tractable on HPC systems; the shipped job-launch tooling assumes an IBM LSF scheduler and must be extended (e.g. via the TcshJob/LSFJob class in submit.py) for Slurm or Cobalt
- DNArSim error profiles are calibrated to a specific wet-lab setup — ThermoFisher GeneArt synthesis, MinION R9.4_180mv_450bps sequencing, Guppy v5.0.7 'super-accurate' basecalling — so findings may not transfer to other channels
- DNArSim requires Julia 1.5.3+ (tested through v1.6.2), and FrameD's full environment needs MPI; Docker image kvolkel/framed:revision is the fallback for local runs
- Only small subshell-based examples (examples/small) can be run on commodity hardware; a single error model or sparse error-rate sweep cannot support the MSA tradeoff conclusion

## Evidence

- [abstract] The choice to include multiple sequence alignment in the pipeline is dependent on the error rate and the type of errors being injected and is not always beneficial: "We found that the choice to include multiple sequence alignment in the pipeline is dependent on the error rate and the type of errors being injected and is not always beneficial."
- [abstract] FrameD fault-injection test bed and parallelism: "it provides a fault-injection-based test bed in which DNA storage systems can be evaluated"
- [abstract] FrameD transparent parallelism: "FrameD provides the user with transparent parallelism to deal with a large number of reads from sequencing and the need for many fault injection iterations"
- [readme] Error-model configuration files shipped with FrameD: [evidence span withheld — anchor status: external_doc]
- [readme] Compiling fault injection results into a dataframe: [evidence span withheld — anchor status: external_doc]
- [readme] DNArSim scope: [evidence span withheld — anchor status: external_doc]
