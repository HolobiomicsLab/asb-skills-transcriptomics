---
name: runtime-memory-profiling
description: Use when benchmarking a new algorithm implementation — e.g., KMD linkage
  runs across k values on simulated datasets — and you must log per-run timing and
  peak memory to verify computational-feasibility claims such as O(n^2) memory independent
  of k.
when_to_use_negative:
- When the goal is correctness validation (e.g., cophenetic correlation against SciPy's
  cluster.hierarchy.linkage single or average linkage) rather than resource benchmarking
  — profiling does not establish algorithmic equivalence
- When profiling an algorithm that does not maintain per-pair k-minimal-distance lists,
  where the 'inherently slower' KMD cost profile and k-independent O(n^2) memory rationale
  do not apply
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: psutil
  role: Python dependency used to check available memory before/during runs (recommended
    on Windows and macOS)
- name: KMDHierarchicalClustering
  role: Implementation being profiled; its fit(X) call is the instrumented unit of
    work
  repo: github.com/KaplanLab/KMDHierarchicalClustering
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-023-05480-z
    title: 'KMD clustering: robust general-purpose clustering of biological data'
  source_content_hash: sha256:0ce3625dbb8f6466a2a0e2493edc030f52d3d8c5015a2f5936a4880a2d7bbee2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - KMDHierarchicalClustering
  - Python
  - SciPy
  - psutil
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# runtime-memory-profiling

## Summary

Instruments algorithm runs (e.g., KMD linkage clustering at different k settings) with wall-clock timing and peak-memory measurement, using Python's psutil, so per-dataset resource logs can be written alongside correctness validation results.

## When to use

Use when benchmarking a new algorithm implementation — e.g., KMD linkage runs across k values on simulated datasets — and you must log per-run timing and peak memory to verify computational-feasibility claims such as O(n^2) memory independent of k.

## When NOT to use

- When the goal is correctness validation (e.g., cophenetic correlation against SciPy's cluster.hierarchy.linkage single or average linkage) rather than resource benchmarking — profiling does not establish algorithmic equivalence
- When profiling an algorithm that does not maintain per-pair k-minimal-distance lists, where the 'inherently slower' KMD cost profile and k-independent O(n^2) memory rationale do not apply

## Inputs

- Dataset matrix X (e.g., scikit-learn-generated toy dataset, 1000 objects, seed=1)
- Parameter configurations to profile (e.g., k=1 and large-k settings of KMDClustering)
- Python environment with KMDHierarchicalClustering installed and psutil>=5.8.0 available

## Outputs

- Per-dataset timing measurements for each k setting
- Peak-memory measurements for each k setting
- Timing/memory log section appended to the validation report (e.g., kmd_linkage_verification_report.md)

## How to apply

Wrap each clustering run (e.g., kmd_cluster.fit(X) on a scikit-learn toy dataset of 1000 objects) with a wall-clock timer and a memory checker, once per dataset and once per k setting (e.g., k=1 and a large k). Use psutil>=5.8.0 to check available memory before runs — the KMDHierarchicalClustering README recommends it specifically for Windows and macOS users. Interpret timings against the known cost profile: the python/scipy KMD implementation is slower than single and average linkage because it maintains, for each pair of clusters, a list of the k minimal distances, so elapsed time should grow with the k_scan_range when scanning k. Write the timing and memory log together with the dendrogram-validation results (e.g., the cophenetic-correlation section of kmd_linkage_verification_report.md) so feasibility evidence and correctness evidence are reported jointly.

## Related tools

- **psutil** (Python dependency used to check available memory before/during runs (recommended on Windows and macOS))
- **KMDHierarchicalClustering** (Implementation being profiled; its fit(X) call is the instrumented unit of work) — github.com/KaplanLab/KMDHierarchicalClustering

## Examples

```
import psutil, time; t0 = time.perf_counter(); kmd_cluster.fit(X); elapsed = time.perf_counter() - t0; print(elapsed, psutil.virtual_memory())
```

## Evaluation signals

- A timing/memory log is written for every dataset and k setting and lands in kmd_linkage_verification_report.md next to the cophenetic-correlation results
- Peak memory remains O(n^2) and independent of k when comparing the k=1 run to the large-k run on the same 1000-object dataset
- psutil confirms sufficient available memory before each run, with no out-of-memory failure on the toy datasets
- KMD runtimes are expected to exceed SciPy single/average linkage runtimes on identical data — a faster-than-SciPy KMD run suggests an instrumentation error
- Timing increases with the number of k values scanned (k_scan_range), consistent with the cost of maintaining k-minimal-distance lists per cluster pair

## Limitations

- The python/scipy implementation results in longer running times than widely used clustering algorithms implemented in optimized non-interpreted code
- A single KMD clustering run may be inherently slower than single and average linkage because it maintains a list of the k minimal distances for each pair of clusters
- Automatic selection of the hyperparameter k requires running the clustering over a large range of k values, multiplying the computational resources consumed during profiling

## Evidence

- [readme] psutil>=5.8.0  # recommended for Windows and macOS platform users, in order to check available memory: "psutil>=5.8.0  # recommended for Windows and macOS platform users, in order to check available memory"
- [discussion] our current implementation is in python/scipy. This makes the code more accessible to users, but results in longer running times than widely used clustering algorithms: "our current implementation is in python/scipy. This makes the code more accessible to users, but results in longer running times than widely used clustering algorithms"
- [discussion] a single run of KMD clustering may be inherently slower than some other approaches (such as single and average linkage clustering): "a single run of KMD clustering may be inherently slower than some other approaches (such as single and average linkage clustering)"
- [methods] pip install git+https://github.com/KaplanLab/KMDHierarchicalClustering: "pip install git+https://github.com/KaplanLab/KMDHierarchicalClustering"
