---
name: methylation-pipeline-runtime-comparison
description: Use when estimating or reproducing the computational cost of methylR's
  ChAMP or minfi methylation array pipelines on the paper's test dataset under stated
  hardware constraints, e.g. to plan capacity or validate the tool.
when_to_use_negative:
- Evaluating analysis correctness or biological output quality (DMCs, pathway enrichment)
  rather than runtime — use the paper's result-comparison evaluation instead
- Running on ARM64 hardware, where methylR's Docker container is explicitly unsupported
  and timings would not be comparable
- Benchmarking the public web server (https://methylr.serve.scilifelab.se/) rather
  than a controlled local Docker deployment with fixed core/memory limits
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: methylR
  role: Shiny-based graphical pipeline whose ChAMP and minfi backends are being timed
    on the test dataset
  repo: https://github.com/JD2112/methylr
- name: Docker
  role: Containerization used to launch methylR locally (jd21/methylr:latest) under
    controlled hardware constraints; AMD64 only
  repo: https://github.com/JD2112/methylr-devel/actions/workflows/docker-image.yml
- name: ChAMP
  role: One of the two complete methylation analysis pipelines executed and timed
    (~3 min reference target)
- name: minfi
  role: The second complete methylation analysis pipeline executed and timed after
    a browser-restart memory reset (~4 min reference target)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad184
    title: 'methylR: a graphical interface for comprehensive DNA methylation array
      data analysis'
  source_content_hash: sha256:865275911faf18f8290aa7ec80a94cff46d4df0ac33f3292f2af338de9820cb6
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# methylation-pipeline-runtime-comparison

## Summary

Benchmark wall-clock runtimes of the methylR Shiny pipeline using its two embedded DNA methylation analysis backends (ChAMP and minfi) on the authors' testData.zip dataset, replicating the paper's reported ~3 min (ChAMP) and ~4 min (minfi) figures.

## When to use

Use when estimating or reproducing the computational cost of methylR's ChAMP or minfi methylation array pipelines on the paper's test dataset under stated hardware constraints, e.g. to plan capacity or validate the tool.

## When NOT to use

- Evaluating analysis correctness or biological output quality (DMCs, pathway enrichment) rather than runtime — use the paper's result-comparison evaluation instead
- Running on ARM64 hardware, where methylR's Docker container is explicitly unsupported and timings would not be comparable
- Benchmarking the public web server (https://methylr.serve.scilifelab.se/) rather than a controlled local Docker deployment with fixed core/memory limits

## Inputs

- testData.zip DNA methylation array test dataset from https://sourceforge.net/projects/methylr/files/testData.zip
- methylR Docker container (jd21/methylr:latest, AMD64 architecture)
- Linux host with Docker (e.g. version 20.10.18) limited to 1 core and 12 GiB RAM

## Outputs

- Wall-clock runtime measurement for one complete ChAMP analysis (target ~3 minutes)
- Wall-clock runtime measurement for one complete minfi analysis (target ~4 minutes)
- pipeline_runtimes. containing both measurements for comparison against reference targets

## How to apply

Download testData.zip from https://sourceforge.net/projects/methylr/files/testData.zip and unpack the Illumina methylation array test dataset. Launch the methylR Docker container on an AMD64-only Linux host (ARM64 is unsupported) limited to the paper's minimal configuration (1 core, 12 GiB RAM) via `docker run --rm -p 3838:3838 jd21/methylr:latest` and open http://localhost:3838. Run one complete analysis with the ChAMP pipeline, timing wall-clock from start to finish, then close the browser to clear temporary memory and restart the session so memory from the first run does not carry over. Repeat identically with the minfi pipeline and compile both measurements (expect roughly ~3 min for ChAMP and ~4 min for minfi) into a JSON file such as pipeline_runtimes. for comparison against the reference targets within a reasonable tolerance band.

## Related tools

- **methylR** (Shiny-based graphical pipeline whose ChAMP and minfi backends are being timed on the test dataset) — https://github.com/JD2112/methylr
- **Docker** (Containerization used to launch methylR locally (jd21/methylr:latest) under controlled hardware constraints; AMD64 only) — https://github.com/JD2112/methylr-devel/actions/workflows/docker-image.yml
- **ChAMP** (One of the two complete methylation analysis pipelines executed and timed (~3 min reference target))
- **minfi** (The second complete methylation analysis pipeline executed and timed after a browser-restart memory reset (~4 min reference target))

## Examples

```
docker run --rm -p 3838:3838 jd21/methylr:latest  # then open http://localhost:3838 and run ChAMP, restart browser, run minfi
```

## Evaluation signals

- ChAMP complete-analysis wall-clock time falls within a reasonable tolerance band around ~3 minutes
- minfi complete-analysis wall-clock time falls within a reasonable tolerance band around ~4 minutes
- Minfi run executed after closing the browser and restarting the session, confirming no temporary-memory carry-over from the ChAMP run
- Both runs completed on the specified AMD64 Linux host under the 1-core / 12 GiB RAM constraint without ARM64 architecture warnings
- pipeline_runtimes. contains both measurements in a comparable unit and matches the recorded stopwatch values

## Limitations

- ARM64 chipset architecture is not supported by methylR, so the container must run on AMD64 (Linux, Intel macOS, or Windows via WSL2)
- Reported runtimes (~3–4 min) apply to the small testData.zip dataset on minimal hardware (1 core, 12 GiB RAM); larger cohort datasets will take substantially longer
- Session memory persists between analyses, so the browser must be closed and the session restarted after each complete pipeline run or the second timing will be inflated

## Evidence

- [methods] Test data download location: "If you want to run with the test data, Please download the testdata from [https://sourceforge.net/projects/methylr/files/testData.zip]"
- [methods] Browser restart between pipeline runs: "after one complete analysis (either with *ChAMP* or *minfi*), close the browser (to clear the temporary memory) and start again."
- [readme] Docker run command: "# with docker container
docker run --rm -p 3838:3838 jd21/methylr:latest"
- [readme] AMD64-only architecture requirement: "PLEASE NOTE: the ARM64 chipset architecture is actually not supported by *methylR*."
- [full_text] Paper-reported runtimes (finding): [evidence span withheld — anchor status: not_found]
