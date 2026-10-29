---
name: decode-failure-counter-aggregation
description: Use when fault-injection simulation results are scattered across one
  output directory per parameter combination and you need aggregated decode-failure/error-location
  statistics for cross-configuration analysis.
when_to_use_negative:
- Fault-injection simulations have not yet been run — no leaf directories or fi.stats
  files exist to aggregate
- The raw sequencing/simulation outputs have not been organized under a single top
  result path, so leaves from different fault-injected files would be mixed
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3316
tools:
- name: FrameD
  role: Generates the per-parameter fault-injection outputs (fi.stats and JSON files)
    that this skill aggregates; provides transparent parallelism over fault injection
    iterations
  repo: https://github.com/dna-storage/framed
- name: db_gen.py
  role: Compiles all leaf-directory statistics for a fault-injected file into a single
    Pandas dataframe
  repo: https://github.com/dna-storage/framed
- name: DNArSim
  role: External nanopore channel simulator whose runs (examples/DNArSim) produce
    fault-injection outputs amenable to this aggregation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad572
    title: 'FrameD: framework for DNA-based data storage design, verification, and
      validation'
  source_content_hash: sha256:35ba405b3af52d6bb2187b83a2c5ac8c8062ebeeb65b4f6d4e202435b6051426
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dna-storage/framed
  tools:
  - DNArSim
  - FrameD
  - db_gen.py
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# Decode failure counter aggregation

## Summary

Collects per-parameter fault-injection output from FrameD simulations — a human-readable 'fi.stats' file in each parameter-combination leaf directory — and compiles all leaves into a single Pandas dataframe so decode failures and byte/base error locations can be compared across run configurations.

## When to use

Use when fault-injection simulation results are scattered across one output directory per parameter combination and you need aggregated decode-failure/error-location statistics for cross-configuration analysis.

## When NOT to use

- Fault-injection simulations have not yet been run — no leaf directories or fi.stats files exist to aggregate
- The raw sequencing/simulation outputs have not been organized under a single top result path, so leaves from different fault-injected files would be mixed

## Inputs

- FrameD fault-injection output tree with one leaf directory per parameter combination
- Per-leaf 'fi.stats' statistics files and JSON parameter record files
- Top-level path of the result collection (--path)

## Outputs

- Pandas dataframe (e.g. fault_injection_dataframe) stored at the top directory of the result collection
- Aggregated counters including decode failures and byte-error and base-error locations per parameter combination

## How to apply

After fault injection runs complete, inspect each leaf output directory, which contains statistics and JSON files plus a human-readable 'fi.stats' file with counters gathered during fault injection (decode failures, byte-error and base-error locations). Verify that a statistics file exists for every parameter combination generated from the JSON configuration's encoder/decoder/fault-distribution/copy-distribution parameters and any parameter sweep. Then run the aggregation script `db_gen.py` from `tools/data_analysis`, passing the top result path; it compiles all leaves for a given fault-injected file into a Pandas dataframe stored at the top of the result collection. Analyze the dataframe (see `notebooks/framed.ipynb`, which reproduces the paper's figures from `framed_DNArSim.pickle` and `framed_iid.pickle`) to compare pipeline choices, e.g. whether multiple sequence alignment helps, as a function of error rate and injected error type.

## Related tools

- **FrameD** (Generates the per-parameter fault-injection outputs (fi.stats and JSON files) that this skill aggregates; provides transparent parallelism over fault injection iterations) — https://github.com/dna-storage/framed
- **db_gen.py** (Compiles all leaf-directory statistics for a fault-injected file into a single Pandas dataframe) — https://github.com/dna-storage/framed
- **DNArSim** (External nanopore channel simulator whose runs (examples/DNArSim) produce fault-injection outputs amenable to this aggregation)

## Examples

```
cd $DNASTORAGE_HOME/tools/data_analysis && python db_gen.py --path <top result path> --name fault_injection_dataframe
```

## Evaluation signals

- A statistics file (fi.stats) exists in every leaf directory, i.e. for every parameter combination
- The statistics files contain aggregated counters including decode failures and byte-error and base-error locations
- db_gen.py produces a single dataframe at the top result path covering all leaves for the given fault-injected file
- The dataframe supports comparisons of pipeline stage choices (e.g. multiple sequence alignment) across error rates and injected error types, matching the paper's findings
- Notebook figures (framed.ipynb) can be regenerated from the compiled data (framed_DNArSim.pickle / framed_iid.pickle)

## Limitations

- Full-scale paper runs (240 simulations from the 6 example JSONs) require an HPC system; the job generation assumes an IBM LSF scheduler and must be extended (TcshJob/LSFJob class in tools/lsf/submit.py) for Slurm, Cobalt, etc.
- Large files may encode/decode poorly on standard computers; encoding/decoding/analysis perform best on systems with many cores and MPI installed
- README was truncated before the acknowledgment section, so some dependency details may be missing

## Evidence

- [readme] In these leaves, you will also find a human readable "fi.stats" file containing statistics gathered during fault injection.: [evidence span withheld — anchor status: external_doc]
- [readme] python db_gen.py --path <top result path> --name fault_injection_dataframe: [evidence span withheld — anchor status: external_doc]
- [readme] The final command generates a Pandas dataframe, and stores it at the top directory path of the collection of results: [evidence span withheld — anchor status: external_doc]
- [abstract] We found that the choice to include multiple sequence alignment in the pipeline is dependent on the error rate and the type of errors being injected and is not always beneficial.: "We found that the choice to include multiple sequence alignment in the pipeline is dependent on the error rate and the type of errors being injected and is not always beneficial."
- [abstract] FrameD provides the user with transparent parallelism to deal with a large number of reads from sequencing and the need for many fault injection iterations: "FrameD provides the user with transparent parallelism to deal with a large number of reads from sequencing and the need for many fault injection iterations"
