---
name: nanopore-error-model-fault-injection
description: Use when evaluating DNA storage pipeline designs under a channel-with-memory
  (nanopore) error model, specifically to test whether MSA-only pipelines can compete
  with HEDGES-MSA when errors are bursty rather than i.i.d.
when_to_use_negative:
- If the error model of interest is i.i.d. (uncorrelated) errors at fixed rates like
  1%, 5%, 10% — use the iid-model Pareto front skill with examples/iid configurations
  instead, since MSA optimality conclusions differ between the two models
- If you only need to encode/decode a single small file without fault-injection statistics
  — running 240 nanopore simulations requires HPC/LSF infrastructure and is overkill
- If the storage system under study uses error profiles unrelated to MinION nanopore
  sequencing (the DNArSim profiles derive from specific synthesis and Guppy v5.0.7
  super-accurate basecalling)
tools:
- name: FrameD
  role: Fault-injection test bed that wires DNArSim-generated nanopore reads through
    the RS/HEDGES/HEDGES-MSA pipelines with transparent parallelism and collects read/write
    density statistics
  repo: github.com/dna-storage/framed
- name: DNArSim
  role: Channel model with memory simulator for nanopore DNA data storage; simulates
    synthesis-to-basecalling to produce FASTQ reads with bursty error profiles
  repo: github.com/BHam-1/DNArSim
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad572
    title: 'FrameD: framework for DNA-based data storage design, verification, and
      validation'
  source_content_hash: sha256:61bda9cc1b9decb6dbcedf3e10be72f20a858763653d6b349989e83d2483ca9f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DNArSim
  - FrameD
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# nanopore-error-model-fault-injection

## Summary

Runs fault-injection simulations of DNA storage pipelines (RS, HEDGES, HEDGES-MSA) under the DNArSim nanopore channel model (memory length k=6) inside FrameD, then computes the cross-pipeline read/write density Pareto frontier. It tests whether nanopore's higher frequency of burst errors changes pipeline optimality relative to i.i.d. error models.

## When to use

Use when evaluating DNA storage pipeline designs under a channel-with-memory (nanopore) error model, specifically to test whether MSA-only pipelines can compete with HEDGES-MSA when errors are bursty rather than i.i.d.

## When NOT to use

- If the error model of interest is i.i.d. (uncorrelated) errors at fixed rates like 1%, 5%, 10% — use the iid-model Pareto front skill with examples/iid configurations instead, since MSA optimality conclusions differ between the two models
- If you only need to encode/decode a single small file without fault-injection statistics — running 240 nanopore simulations requires HPC/LSF infrastructure and is overkill
- If the storage system under study uses error profiles unrelated to MinION nanopore sequencing (the DNArSim profiles derive from specific synthesis and Guppy v5.0.7 super-accurate basecalling)

## Inputs

- DNArSim nanopore channel configuration (input reference sequence in FASTA, e.g. example/ref.txt, number of reads, channel memory k=6)
- FrameD pipeline configuration JSONs for RS, HEDGES, and HEDGES-MSA under the DNArSim error model (examples/DNArSim/*.json)
- HPC/LSF job submission setup (or the small sub-shell example for commodity hardware)

## Outputs

- Per-configuration fault-injection statistics files (fi.stats) and JSON outputs in leaf directories
- Compiled Pandas dataframe of fault-injection results (e.g. via db_gen.py)
- Cross-pipeline read/write density Pareto frontier, where every frontier point belongs to HEDGES-MSA and no MSA-only configuration appears

## How to apply

Load the DNArSim nanopore channel simulator (github.com/BHam-1/DNArSim) with its default channel memory length k=6, which models nanopore burst errors derived from MinION R9.4_180mv_450bps sequencing basecalled with Guppy v5.0.7 in super-accurate mode. Configure FrameD's fault-injection test bed using the three JSON configurations under examples/DNArSim, which drive the RS, HEDGES, and HEDGES-MSA pipelines; the DNArSim step produces FASTQ-format simulated reads that feed the pipelines. Run the fault-injection iterations relying on FrameD's transparent MPI and batch-level parallelism (240 total simulations in the paper setup), recording read density and write density for every configuration. Compute the cross-pipeline Pareto frontier by taking Pareto-optimal configurations across all three pipelines jointly. Rationale: nanopore errors are correlated/bursty, so the decision to include multiple sequence alignment (MSA) depends on error type, not just error rate — under this model the frontier should consist exclusively of HEDGES-MSA configurations with no MSA-only point.

## Related tools

- **FrameD** (Fault-injection test bed that wires DNArSim-generated nanopore reads through the RS/HEDGES/HEDGES-MSA pipelines with transparent parallelism and collects read/write density statistics) — github.com/dna-storage/framed
- **DNArSim** (Channel model with memory simulator for nanopore DNA data storage; simulates synthesis-to-basecalling to produce FASTQ reads with bursty error profiles) — github.com/BHam-1/DNArSim

## Examples

```
cd $DNASTORAGE_HOME; tcsh examples/small/run_small.csh
```

## Evaluation signals

- Every configuration on the computed cross-pipeline read/write density frontier is a HEDGES-MSA pipeline
- No MSA-only (alignment-without-HEDGES) configuration appears on the frontier under the nanopore model
- Fault-injection leaf directories contain fi.stats and JSON files for all 240 DNArSim-launched simulations
- db_gen.py produces a single Pandas dataframe over the DNArSim result tree that loads without missing runs
- Simulated reads from DNArSim are FASTQ-formatted (without scores) and generated with k=6 channel memory as recommended

## Limitations

- The DNArSim error profile derives from one specific setup (Thermofisher GeneArt synthesis, MinION R9.4_180mv_450bps, Guppy v5.0.7 super-accurate basecalling) and may not transfer to other sequencers or basecallers
- Full DNArSim fault-injection sweeps are only tractable on HPC systems; the shipped job scripts assume an IBM LSF scheduler and must be extended (e.g. TcshJob class in tools/lsf/submit.py) for Slurm or Cobalt
- MSA inclusion is error-rate- and error-type-dependent, so conclusions under the nanopore burst model must not be generalized to i.i.d. models (where HEDGES envelops HEDGES-MSA at 1% error)

## Evidence

- [abstract] fault-injection-based simulation of DNA storage system designs: "it provides a fault-injection-based test bed in which DNA storage systems can be evaluated"
- [abstract] MSA benefit depends on error rate and error type: "the choice to include multiple sequence alignment in the pipeline is dependent on the error rate and the type of errors being injected and is not always beneficial"
- [readme] DNArSim configurations in FrameD repo: [evidence span withheld — anchor status: external_doc]
- [readme] Parallelism requirement for fault injection: [evidence span withheld — anchor status: external_doc]
