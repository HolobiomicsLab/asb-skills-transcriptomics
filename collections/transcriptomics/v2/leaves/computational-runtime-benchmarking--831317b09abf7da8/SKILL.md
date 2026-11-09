---
name: computational-runtime-benchmarking
description: Use when a new tool claims speed advantages over established comparators
  and you need reproducible runtime/memory benchmarks plus accuracy checks on the
  same input datasets with default settings.
when_to_use_negative:
- When the question is output correctness alone and compute cost is irrelevant — a
  pure accuracy comparison on matched parameters suffices without timing harnesses.
- When tools solve different subtasks with non-overlapping outputs (e.g. wf-single-cell
  produces full count matrices, not just barcode discovery) — normalize scope before
  comparing runtimes or the comparison is invalid.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: Flexiplex
  role: Tool under test; run in discovery mode ('flexiplex -f 0') with four threads
    to discover cellular barcodes from noisy long reads
  repo: https://github.com/DavidsonGroup/flexiplex
- name: flexiplex-filter
  role: Filters discovered barcodes against the whitelist and removes empty droplets
    using the inflection point method to produce comparable barcode frequency lists
  repo: https://github.com/DavidsonGroup/flexiplex
- name: BLAZE
  role: Purpose-built comparator for long-read cell barcode discovery; runtime baseline
  repo: https://github.com/shimlab/BLAZE
- name: scTagger
  role: Purpose-built comparator for barcode discovery; runtime baseline
- name: wf-single-cell
  role: Nextflow comparator workflow; uses an order of magnitude more compute resources
    (recommended 64 CPUs, 256GB memory)
  repo: https://github.com/epi2me-labs/wf-single-cell
- name: conda
  role: 'Installs Flexiplex for benchmarking: conda install -c bioconda -c conda-forge
    flexiplex'
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae102
    title: 'Flexiplex: a versatile demultiplexer and search tool for omics data'
  source_content_hash: sha256:7f7015fb3a514b81ec8d8b2510186d1a6b820088fe0d568ff884e390273ce67c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DavidsonGroup/flexiplex
  tools:
  - BLAZE
  - Flexiplex
  - conda
  - flexiplex-filter
  - scTagger
  - wf-single-cell
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# computational-runtime-benchmarking

## Summary

Benchmark the computational efficiency (wall-clock runtime and memory) of a new tool against task-specific comparators on identical datasets, while jointly verifying that accuracy (sensitivity/specificity) is not sacrificed. Grounded in Flexiplex's barcode-discovery benchmark against BLAZE, scTagger, and wf-single-cell.

## When to use

Use when a new tool claims speed advantages over established comparators and you need reproducible runtime/memory benchmarks plus accuracy checks on the same input datasets with default settings.

## When NOT to use

- When the question is output correctness alone and compute cost is irrelevant — a pure accuracy comparison on matched parameters suffices without timing harnesses.
- When tools solve different subtasks with non-overlapping outputs (e.g. wf-single-cell produces full count matrices, not just barcode discovery) — normalize scope before comparing runtimes or the comparison is invalid.

## Inputs

- Long-read single-cell RNA-seq FASTQ datasets (e.g. scmixology 2, GridION Q20, GridION, PromethION hiPSC replicates)
- Short-read-derived barcode whitelist used as withheld ground truth
- Comparator tool installations (BLAZE, scTagger, wf-single-cell, and the tool under test)

## Outputs

- benchmark_table.csv comparing sensitivity, specificity, wall-clock runtime, and memory across tools
- Per-tool barcode frequency lists after empty-droplet filtering (e.g. filtered.txt from flexiplex-filter)
- Recorded wall-clock time and memory use per tool per replicate

## How to apply

Download the same input datasets (e.g. scmixology 2 and GridION Q20/GridION/PromethION hiPSC long-read single-cell RNA-seq FASTQ files) plus a short-read-derived barcode whitelist used as withheld truth for all tools. Run each tool — including the new tool and comparators (Flexiplex in discovery mode via 'flexiplex -f 0', BLAZE, scTagger, wf-single-cell) — on identical replicates with a fixed thread count (four threads in the Flexiplex paper) and default parameters, recording wall-clock time and peak memory for each run. Then post-process outputs identically for each tool (e.g. run 'flexiplex-filter --whitelist ... --outfile filtered.txt flexiplex_barcodes_counts.txt' to remove empty droplets using the inflection point method) so the accuracy comparison is fair. Evaluate sensitivity and specificity of each tool's output (discovered barcode set) against the withheld truth whitelist, then assemble a benchmark table (benchmark_table.csv) comparing sensitivity, specificity, runtime, and memory. Validate that speedups do not come at the cost of accuracy — in the Flexiplex case, comparable sensitivity/specificity to BLAZE and scTagger while completing in roughly 5%-16% of BLAZE's runtime and 25%-42% of scTagger's runtime.

## Related tools

- **Flexiplex** (Tool under test; run in discovery mode ('flexiplex -f 0') with four threads to discover cellular barcodes from noisy long reads) — https://github.com/DavidsonGroup/flexiplex
- **flexiplex-filter** (Filters discovered barcodes against the whitelist and removes empty droplets using the inflection point method to produce comparable barcode frequency lists) — https://github.com/DavidsonGroup/flexiplex
- **BLAZE** (Purpose-built comparator for long-read cell barcode discovery; runtime baseline) — https://github.com/shimlab/BLAZE
- **scTagger** (Purpose-built comparator for barcode discovery; runtime baseline)
- **wf-single-cell** (Nextflow comparator workflow; uses an order of magnitude more compute resources (recommended 64 CPUs, 256GB memory)) — https://github.com/epi2me-labs/wf-single-cell
- **conda** (Installs Flexiplex for benchmarking: conda install -c bioconda -c conda-forge flexiplex)

## Examples

```
flexiplex -f 0 reads.fastq; flexiplex-filter --whitelist whitelist.txt --outfile filtered.txt flexiplex_barcodes_counts.txt
```

## Evaluation signals

- Flexiplex achieves sensitivity/specificity similar to BLAZE and scTagger against the withheld short-read-derived barcode truth
- Flexiplex completes barcode discovery in roughly 5%-16% of BLAZE's runtime and 25%-42% of scTagger's runtime with four threads (Fig. 1H)
- wf-single-cell uses an order of magnitude more compute resources than the dedicated barcode-discovery tools
- All tools run on identical FASTQ replicates with default settings and a fixed thread count so runtimes are directly comparable
- benchmark_table.csv contains sensitivity, specificity, runtime, and memory for every tool-dataset combination

## Limitations

- Runtimes are hardware- and thread-count-dependent; the reported ratios assume four threads and the paper's compute environment, so absolute times will differ elsewhere
- The benchmark measures barcode discovery only; wf-single-cell performs additional tasks (mapping, count matrices) so it is not strictly equivalent in scope
- Sensitivity/specificity depend on the quality of the short-read-derived whitelist used as withheld truth

## Evidence

- [results] Flexiplex runtime ratios vs BLAZE and scTagger: [evidence span withheld — anchor status: not_found]
- [methods] flexiplex-filter invocation: "flexiplex-filter --whitelist 3M-february-2018.txt --outfile filtered.txt flexiplex_barcodes_counts.txt"
- [methods] whitelist filtering and empty droplet removal capability: "Flexiplex can search for barcodes, filter against a whitelist, and remove empty droplets using an inflection point method."
- [abstract] Flexiplex accuracy-efficiency balance claim: "Flexiplex achieves an excellent balance of accuracy and computational efficiency compared to leading task-specific tools"
- [readme] wf-single-cell compute requirements: "Recommended requirements:

+ CPUs = 64
+ Memory = 256GB"
