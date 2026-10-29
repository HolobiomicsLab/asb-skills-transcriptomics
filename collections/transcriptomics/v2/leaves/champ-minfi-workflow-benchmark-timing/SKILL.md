---
name: champ-minfi-workflow-benchmark-timing
description: Use when you need to reproduce or validate the reported runtimes (~3
  min ChAMP, ~4 min minfi) of a methylR DNA methylation array analysis on the testData
  dataset under minimal hardware constraints.
when_to_use_negative:
- The host is ARM64 — the README explicitly states 'the ARM64 chipset architecture
  is actually not supported by methylR', so timing runs would fail or be unrepresentative.
- The goal is downstream module accuracy (DMPs, pathway enrichment) rather than runtime
  benchmarking — the skill only measures execution time, not result correctness.
- The dataset is a large user-supplied cohort rather than the testData.zip sample
  — runtimes will not be comparable to the ~3–4 minute reference targets.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3518
tools:
- name: methylR
  role: Shiny-based graphical pipeline that runs the complete DNA methylation analysis
    whose runtime is being benchmarked
  repo: https://github.com/JD2112/methylr
- name: Docker
  role: Container runtime used to launch the methylR app locally in a controlled environment
    (AMD64 image)
  repo: https://github.com/JD2112/methylr-devel
- name: ChAMP
  role: First of the two DNA methylation analysis pipelines executed end-to-end and
    timed (~3 min reference)
- name: minfi
  role: Second pipeline executed end-to-end after a browser restart to clear temporary
    memory (~4 min reference)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad184
    title: 'methylR: a graphical interface for comprehensive DNA methylation array
      data analysis'
  source_content_hash: sha256:9bf1a613f04c65b88163623bab51f0a26ff27c7f1e89cd610a84a190c40b166e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JD2112/methylr
  tools:
  - ChAMP
  - Docker
  - methylR
  - minfi
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# champ-minfi-workflow-benchmark-timing

## Summary

Benchmarks the wall-clock runtime of a complete methylR methylation analysis using the ChAMP or minfi pipelines on the authors' testData.zip dataset, under the stated minimal hardware configuration (1 core, 12 GiB RAM, AMD64 Linux Docker container).

## When to use

Use when you need to reproduce or validate the reported runtimes (~3 min ChAMP, ~4 min minfi) of a methylR DNA methylation array analysis on the testData dataset under minimal hardware constraints.

## When NOT to use

- The host is ARM64 — the README explicitly states 'the ARM64 chipset architecture is actually not supported by methylR', so timing runs would fail or be unrepresentative.
- The goal is downstream module accuracy (DMPs, pathway enrichment) rather than runtime benchmarking — the skill only measures execution time, not result correctness.
- The dataset is a large user-supplied cohort rather than the testData.zip sample — runtimes will not be comparable to the ~3–4 minute reference targets.

## Inputs

- testData.zip methylation array test dataset from SourceForge (https://sourceforge.net/projects/methylr/files/testData.zip)
- methylR Docker container (jd21/methylr:latest, AMD64 architecture)
- Linux host with 1 CPU core and 12 GiB RAM

## Outputs

- Wall-clock runtime measurement for a complete ChAMP pipeline analysis (~3 minutes reference)
- Wall-clock runtime measurement for a complete minfi pipeline analysis (~4 minutes reference)
- pipeline_runtimes. compiling both measurements

## How to apply

Download the methylation array test dataset from https://sourceforge.net/projects/methylr/files/testData.zip and launch the methylR Docker container (AMD64 only, e.g. 'docker run --rm -p 3838:3838 jd21/methylr:latest') on a Linux host limited to 1 core and 12 GiB RAM. Load the unpacked test data into the Shiny interface and execute one complete ChAMP analysis, timing the run with a wall-clock stopwatch from start to finish. Then close the browser to clear temporary memory and restart the session to prevent memory carry-over, and run one complete minfi analysis on the same data under identical constraints. Record both measurements (reference targets: ~3 minutes for ChAMP, ~4 minutes for minfi) and compare within a reasonable tolerance band. The close-browser-and-restart step is mandated by the authors to isolate each pipeline run.

## Related tools

- **methylR** (Shiny-based graphical pipeline that runs the complete DNA methylation analysis whose runtime is being benchmarked) — https://github.com/JD2112/methylr
- **Docker** (Container runtime used to launch the methylR app locally in a controlled environment (AMD64 image)) — https://github.com/JD2112/methylr-devel
- **ChAMP** (First of the two DNA methylation analysis pipelines executed end-to-end and timed (~3 min reference))
- **minfi** (Second pipeline executed end-to-end after a browser restart to clear temporary memory (~4 min reference))

## Examples

```
docker run --rm -p 3838:3838 jd21/methylr:latest
```

## Evaluation signals

- ChAMP run completes within a reasonable tolerance of ~3 minutes wall-clock time on 1 core / 12 GiB RAM.
- minfi run completes within a reasonable tolerance of ~4 minutes wall-clock time under identical hardware constraints.
- The minfi run was preceded by closing the browser and restarting the session, so no memory carry-over from the ChAMP run inflates the timing.
- Both measurements are compiled into pipeline_runtimes. with the same test dataset and identical hardware constraints for both runs.
- The Docker container runs on AMD64 architecture (Linux OS, e.g. Ubuntu 20.04 LTS, Docker version 20.10.18) as required by methylR.

## Limitations

- ARM64 chipset architecture is not supported by methylR; timings are only reproducible on AMD64 hardware.
- Each complete analysis must be followed by closing the browser to clear temporary memory before starting another pipeline, otherwise timings are contaminated by memory carry-over.
- The Docker container was built on AMD64 OS architecture specifically 'to get the full functional capabilities of Shiny/R', so other architectures are untested.

## Evidence

- [methods] Close browser between runs: "after one complete analysis (either with *ChAMP* or *minfi*), close the browser (to clear the temporary memory) and start again."
- [methods] Test data download: "If you want to run with the test data, Please download the testdata from [https://sourceforge.net/projects/methylr/files/testData.zip]"
- [methods] ARM64 not supported: "NOTE: the ARM64 chipset architecture is actually not supported by methylR."
- [readme] Docker run command: "# with docker container
docker run --rm -p 3838:3838 jd21/methylr:latest"
- [readme] Docker built for AMD64: [evidence span withheld — anchor status: external_doc]
