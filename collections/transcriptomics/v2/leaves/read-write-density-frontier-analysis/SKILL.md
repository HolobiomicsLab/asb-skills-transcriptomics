---
name: read-write-density-frontier-analysis
description: Use when comparing multiple DNA storage pipeline designs (e.g., with/without
  MSA, with/without an inner code like HEDGES) on measured read density and write
  density outputs from fault-injection simulations, and you need the Pareto-optimal
  set.
when_to_use_negative:
- You have only a single pipeline or error model — the frontier requires Pareto comparison
  across multiple pipelines' configurations.
- You lack fault-injection (simulated sequencing) outputs and only have analytical
  or theoretical density estimates not tied to an error model.
- Your metric of interest is not read/write density (e.g., you are studying decoding
  success rate alone).
edam_operation: http://edamontology.org/operation_0337
tools:
- name: FrameD
  role: Fault-injection test bed that runs the storage pipelines under an error model
    with transparent parallelism, and produces the per-run statistics consumed by
    the frontier analysis
  repo: https://github.com/dna-storage/framed
- name: DNArSim
  role: Nanopore channel model (with memory k=6) simulating synthesis-through-basecalling
    to generate error-injected reads for the nanopore error-model variant of the analysis
  repo: https://github.com/BHam-1/DNArSim
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad572
    title: 'FrameD: framework for DNA-based data storage design, verification, and
      validation'
  source_content_hash: sha256:7a75bd4b58673bbd67b0820f20b2ce76cdcf07394822f1dec16bdfaaa5757ddb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dna-storage/framed
  tools:
  - DNArSim
  - FrameD
  edam_operation: http://edamontology.org/operation_0337
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# read-write-density-frontier-analysis

## Summary

Computes the cross-pipeline Pareto frontier of read density vs. write density for DNA-based data storage designs under a fault-injection test bed (e.g., FrameD driven by the DNArSim nanopore channel model), and identifies which pipeline configurations compose the frontier.

## When to use

Use when comparing multiple DNA storage pipeline designs (e.g., with/without MSA, with/without an inner code like HEDGES) on measured read density and write density outputs from fault-injection simulations, and you need the Pareto-optimal set.

## When NOT to use

- You have only a single pipeline or error model — the frontier requires Pareto comparison across multiple pipelines' configurations.
- You lack fault-injection (simulated sequencing) outputs and only have analytical or theoretical density estimates not tied to an error model.
- Your metric of interest is not read/write density (e.g., you are studying decoding success rate alone).

## Inputs

- Fault-injection simulation outputs (fi.stats files and json statistics) per pipeline configuration and error-rate setting
- Pipeline configuration metadata (storage pipeline variant, error model, error-rate settings)
- Compiled Pandas dataframe from db_gen.py aggregating all runs

## Outputs

- Per-configuration table of read density and write density
- Cross-pipeline Pareto frontier over read/write density (Pareto-optimal configuration list)
- Read/write density frontier plot (paper Fig. 5-style figure) and frontier composition verdict (which pipelines appear)

## How to apply

Run fault-injection simulations for each pipeline configuration under the chosen error model (e.g., the DNArSim nanopore channel with memory k=6), recording read density and write density for every configuration; FrameD's transparent parallelism (MPI plus batch-level parallelism) handles the large number of sequencing reads and many fault injection iterations. After runs finish, compile per-run 'fi.stats' statistics into a single Pandas dataframe using db_gen.py, then load it for cross-configuration analysis. Compute the cross-pipeline frontier by taking the Pareto-optimal configurations across all three pipelines — a point is on the frontier if no other configuration from any pipeline dominates it on both read density and write density. Plot the frontier (e.g., as the Fig. 5 read/write density figure) using the provided notebooks, which reproduce paper figures from the raw 'framed_DNArSim.pickle' and 'framed_iid.pickle' data. Validate the composition of the frontier against expectations: under the nanopore error model, the frontier should consist exclusively of HEDGES-MSA configurations, because nanopore's higher frequency of burst errors makes MSA-only configurations non-optimal.

## Related tools

- **FrameD** (Fault-injection test bed that runs the storage pipelines under an error model with transparent parallelism, and produces the per-run statistics consumed by the frontier analysis) — https://github.com/dna-storage/framed
- **DNArSim** (Nanopore channel model (with memory k=6) simulating synthesis-through-basecalling to generate error-injected reads for the nanopore error-model variant of the analysis) — https://github.com/BHam-1/DNArSim

## Examples

```
python db_gen.py --path <top result path> --name fault_injection_dataframe
```

## Evaluation signals

- Every pipeline configuration has non-missing read density and write density rows in the db_gen.py dataframe.
- Each reported frontier point is dominated by no other configuration from any pipeline on both read and write density.
- Under the DNArSim nanopore error model, all frontier points belong to HEDGES-MSA configurations and no MSA-only configuration appears on the frontier.
- Notebook-generated figures reproduce the paper figures from framed_DNArSim.pickle / framed_iid.pickle within expected variation.
- Full run matches the paper setup: 240 simulations across the iid and DNArSim json configurations under examples/.

## Limitations

- Frontier composition is error-rate- and error-type-dependent: including multiple sequence alignment is 'not always beneficial', so conclusions from one error model (e.g., nanopore) do not transfer to others (e.g., i.i.d).
- Full-scale runs (240 simulations) are only tractable on HPC systems; the provided job-launch tooling assumes an IBM LSF scheduler and must be extended for Slurm/Cobalt via the TcshJob class.
- Encoding or decoding large files may perform poorly on standard hardware; the analysis infrastructure performs best with MPI on a many-core system.

## Evidence

- [abstract] it provides a fault-injection-based test bed in which DNA storage systems can be evaluated: "it provides a fault-injection-based test bed in which DNA storage systems can be evaluated"
- [abstract] MSA inclusion is error- and error-type-dependent: "We found that the choice to include multiple sequence alignment in the pipeline is dependent on the error rate and the type of errors being injected and is not always beneficial."
- [readme] Compile fault-injection results into a dataframe: [evidence span withheld — anchor status: external_doc]
- [readme] Notebooks generate paper figures from raw simulation data: [evidence span withheld — anchor status: external_doc]
- [readme] 240 simulations for the paper configurations: [evidence span withheld — anchor status: external_doc]
