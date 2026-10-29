---
name: chromatin-modeling-benchmark-comparison
description: Use when two chromatin modelling tools must be compared for computational
  performance or run-to-run stability, and per-run execution times are available across
  repeated simulations on defined chromosomes.
when_to_use_negative:
- The goal is structural accuracy of 3D chromatin models (e.g. contact-map fidelity)
  rather than computational performance or run-time stability.
- Only a single simulation run per tool is available, so no run-to-run dispersion
  (standard deviation) can be computed.
- The tools are not run on the same workstation and identical chromosome inputs, making
  timing comparisons confounded by hardware or data differences.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3315
tools:
- name: cudaMMC
  role: GPU-enhanced multiscale Monte Carlo chromatin 3D modelling engine whose execution
    times are benchmarked for stability (CoV) against the previous method.
  repo: https://github.com/SFGLab/cudaMMC
- name: 3D-GNOME
  role: Baseline chromatin 3D modelling software against which cudaMMC run-time stability
    and speed are compared.
  repo: https://bitbucket.org/3dome/3dgnome
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad588
    title: 'cudaMMC: GPU-enhanced multiscale Monte Carlo chromatin 3D modelling'
  source_content_hash: sha256:3ab9e36379db69f988a5d795d385b58429fe935b356d770cec5c8c35dcd71979
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SFGLab/cudaMMC
  tools:
  - 3D-GNOME
  - cudaMMC
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# chromatin-modeling-benchmark-comparison

## Summary

Benchmark and compare execution-time stability of chromatin 3D modelling engines (e.g. cudaMMC vs 3D-GNOME) by computing the coefficient of variation (CoV) of run times over repeated simulation runs on the same chromosomes and workstation.

## When to use

Use when two chromatin modelling tools must be compared for computational performance or run-to-run stability, and per-run execution times are available across repeated simulations on defined chromosomes.

## When NOT to use

- The goal is structural accuracy of 3D chromatin models (e.g. contact-map fidelity) rather than computational performance or run-time stability.
- Only a single simulation run per tool is available, so no run-to-run dispersion (standard deviation) can be computed.
- The tools are not run on the same workstation and identical chromosome inputs, making timing comparisons confounded by hardware or data differences.

## Inputs

- per-run execution-time logs for N repeated simulations of each modelling tool on the same chromosomes (e.g. 10 runs of cudaMMC and 3D-GNOME on chr1, chr14, chr21)
- benchmark chromosome regions used consistently across tools (e.g. chr14 input via config.ini ChIA-PET settings)

## Outputs

- per-chromosome coefficient-of-variation (CoV) values for each modelling method
- comparison table of mean and standard deviation of execution times per chromosome-method pair (e.g. cov_execution_times.csv)
- stability comparison summary, matching reported benchmark results such as Supplementary Fig. S2

## How to apply

Run each modelling engine (e.g. cudaMMC and 3D-GNOME) on the same chromosomes, such as chr1, chr14 and chr21, for a fixed number of repeated runs (10 in the cudaMMC study) on an identical workstation, recording the wall-clock execution time of each run. Load the per-run timing logs and, for each chromosome-method pair, compute the mean and standard deviation of the run times, then derive the coefficient of variation as CoV = (std / mean) x 100. Assemble the per-chromosome rows for each method into a comparison table (e.g. cov_execution_times.csv). For the cudaMMC vs 3D-GNOME benchmark the expected reference values are CoV of 2.48% (chr1), 1.24% (chr14) and 2.01% (chr21) for cudaMMC versus 32.05%, 19.79% and 6.45% for 3D-GNOME, so validate the recomputation against these reported stability results (Supplementary Fig. S2) within a small tolerance. In addition to stability, compare absolute run times: the article reports cudaMMC calculations demonstrate significantly faster performance with better stability than the previous method on the same workstation, and cudaMMC substantially reduces the computation time required for generating ensembles of large chromatin models.

## Related tools

- **cudaMMC** (GPU-enhanced multiscale Monte Carlo chromatin 3D modelling engine whose execution times are benchmarked for stability (CoV) against the previous method.) — https://github.com/SFGLab/cudaMMC
- **3D-GNOME** (Baseline chromatin 3D modelling software against which cudaMMC run-time stability and speed are compared.) — https://bitbucket.org/3dome/3dgnome

## Examples

```
./build/cudaMMC -s config.ini -c chr14 -o ~/chiapet/chr14/ -n chr14_test
```

## Evaluation signals

- Recomputed CoV for each chromosome-method pair matches the reported stability results: cudaMMC 2.48% (chr1), 1.24% (chr14), 2.01% (chr21); 3D-GNOME 32.05% (chr1), 19.79% (chr14), 6.45% (chr21), within a small tolerance.
- Each chromosome-method pair aggregates exactly the same number of repeated runs (10 runs in the reported benchmark).
- CoV is computed consistently as (std / mean) x 100 for all pairs, producing a comparable percentage-based stability metric.
- The comparison table covers the full set of benchmark chromosomes (chr1, chr14, chr21) for both 3D-GNOME and cudaMMC.
- cudaMMC CoV values are much lower than 3D-GNOME CoV values, reproducing the finding that cudaMMC shows far more consistent execution times.

## Limitations

- Benchmark stability depends on using the same workstation for both methods; the reported comparison is explicitly for the authors' previous method 'on the same workstation'.
- Comparison requires multiple repeated runs per chromosome-method pair; a single run cannot yield a coefficient of variation.
- Results are specific to the benchmarked chromosomes (chr1, chr14, chr21) and configuration used, so absolute timing values may not transfer to other hardware, data, or chromosome sizes.

## Evidence

- [results] CoV run-time stability comparison between cudaMMC and 3D-GNOME: [evidence span withheld — anchor status: not_found]
- [abstract] cudaMMC faster and more stable than previous method: "The cudaMMC calculations demonstrate significantly faster performance with better stability compared to our previous method on the same workstation."
- [abstract] Computation time reduction: "cudaMMC also substantially reduces the computation time required for generating ensembles of large chromatin models, making it an invaluable tool for studying chromatin spatial conformation."
- [methods] Example cudaMMC run command used for benchmarking: "./build/cudaMMC -s config.ini -c chr14 -o ~/chiapet/chr14/ -n chr14_test"
- [intro] cudaMMC is GPU-enhanced version of 3D-GNOME: "cudaMMC is a GPU-enhanced version of the 3D-GNOME software, which utilizes the Multiscale Monte Carlo approach for 3D chromatin structure modeling."
- [readme] cudaMMC binary execution form: "Example execution command:
```
./build/cudaMMC -s config.ini -c chr14 -o ~/chiapet/chr14/ -n chr14_test
```"
