---
name: genorm-normfinder-stability-measure-computation
description: Use when benchmarking candidate reference-gene (RG) combinations from
  a qPCR array (raw CT matrix, e.g. data_GSE78870/data_GSE50013) and you need GeNorm
  and NormFinder stability measures for each weighted k-gene combination.
when_to_use_negative:
- Input is already normalized expression or RNA-seq counts rather than raw CT values
  from a qPCR array — the pipeline expects CT matrices and normalizes internally with
  norm_method='high_exp' and norm_method_exp_thr=35.
- You only need SD/CV stability of combinations — adding algors=c('Genorm','NormFinder')
  requires building InterOptCuda with nvcc and a CUDA-capable GPU, which is unnecessary
  overhead otherwise.
- Candidate miRNAs have missing/high CT values — miRNAs with CT>37 in any sample are
  excluded by the rowSums(data>37)==0 filter, so datasets dominated by such genes
  yield too few candidates.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: InterOpt
  role: R package whose run_experiment() computes stability measures (SD, CV, Georm,
    NormFinder) for weighted RG combinations on normalized qPCR data
  repo: https://github.com/asalimih/InterOpt
- name: InterOptCuda
  role: CUDA kernel (modified from NormiRazor) that InterOpt calls to calculate NormFinder
    and Genorm measures in parallel across many combinations; must be built and placed
    on PATH
  repo: https://github.com/asalimih/InterOptCuda
- name: GeNorm
  role: Iterative pairwise-variation stability method whose measure is computed per
    RG combination inside run_experiment()
- name: NormFinder
  role: Model-based stability method (separates technical and biological variation)
    whose measure is computed per RG combination inside run_experiment()
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2023.107945
    title: 'InterOpt: Improved gene expression quantification in qPCR experiments
      using weighted aggregation of reference genes'
  source_content_hash: sha256:6562bad5d400ed0e74427acca1712a20f7f69d47bb8deb54df3a6a2090e6cd4c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/asalimih/InterOpt
  tools:
  - GeNorm
  - InterOpt
  - InterOptCuda
  - NormFinder
  - R
  - devtools
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# genorm-normfinder-stability-measure-computation

## Summary

Computes GeNorm and NormFinder stability measures for weighted reference-gene (RG) combinations in qPCR CT data, using InterOpt's run_experiment() with the GPU-accelerated InterOptCuda kernel alongside SD and CV measures.

## When to use

Use when benchmarking candidate reference-gene (RG) combinations from a qPCR array (raw CT matrix, e.g. data_GSE78870/data_GSE50013) and you need GeNorm and NormFinder stability measures for each weighted k-gene combination.

## When NOT to use

- Input is already normalized expression or RNA-seq counts rather than raw CT values from a qPCR array — the pipeline expects CT matrices and normalizes internally with norm_method='high_exp' and norm_method_exp_thr=35.
- You only need SD/CV stability of combinations — adding algors=c('Genorm','NormFinder') requires building InterOptCuda with nvcc and a CUDA-capable GPU, which is unnecessary overhead otherwise.
- Candidate miRNAs have missing/high CT values — miRNAs with CT>37 in any sample are excluded by the rowSums(data>37)==0 filter, so datasets dominated by such genes yield too few candidates.

## Inputs

- Raw CT qPCR array matrix (rows = miRNAs/genes, columns = samples), e.g. data_GSE50013
- Sample group labels vector, e.g. groups_GSE50013
- Pre-filtered candidate RG names (miRNAs with CT<37 in all samples)
- InterOptCuda executable on PATH (built via nvcc)
- run_experiment() parameters: k, norm_method, norm_method_exp_thr, weight_methods, algors, mc.cores

## Outputs

- Per-combination stability measures for GeNorm and NormFinder (plus SD and CV) attached to result from run_experiment()
- Per-method aggregation weights (result$res_source, e.g. result$res_source$geom_sd_hybrid with columns Gene1, Gene2, w1, w2, CV, SD)
- Aggregated reference values (result$aggregated_refs) per sample per combination
- Comparison tables written to stability_benchmark_results.csv

## How to apply

Run InterOpt's run_experiment() on the raw CT matrix with k=2 pair combinations, a normalization method (e.g. norm_method='high_exp', norm_method_exp_thr=35), weight_methods such as c('geom','geom_sd_hybrid'), and algors=c('SD','CV','Genorm','NormFinder'). GeNorm and NormFinder require the InterOptCuda executable on PATH, built with nvcc -O2 InterOptCuda.cu -o InterOptCuda -std=c++11; the kernel is a modified NormiRazor kernel that, per sample, uses only elements with CT<35 for NormFinder's sample mean and restricts GeNorm comparisons to a list of stable genes. Restrict candidate miRNAs first (e.g. rownames(data_GSE50013)[rowSums(data_GSE50013>37)==0]) so only miRNAs with CT<37 in all samples enter combinations. Stability measures are calculated on the automatically normalized data, and per-method weights (result$res_source), aggregated references (result$aggregated_refs), and stability measures can be assembled into comparison tables (e.g. stability_benchmark_results.csv).

## Related tools

- **InterOpt** (R package whose run_experiment() computes stability measures (SD, CV, Georm, NormFinder) for weighted RG combinations on normalized qPCR data) — https://github.com/asalimih/InterOpt
- **InterOptCuda** (CUDA kernel (modified from NormiRazor) that InterOpt calls to calculate NormFinder and Genorm measures in parallel across many combinations; must be built and placed on PATH) — https://github.com/asalimih/InterOptCuda
- **GeNorm** (Iterative pairwise-variation stability method whose measure is computed per RG combination inside run_experiment())
- **NormFinder** (Model-based stability method (separates technical and biological variation) whose measure is computed per RG combination inside run_experiment())

## Examples

```
result = run_experiment(data_source=data_GSE50013, gr_source=groups_GSE50013, ctVal_source=T, sub_names=mirs_for_combinations, k=2, norm_method='high_exp', norm_method_exp_thr=35, weight_methods=c('geom','geom_sd_hybrid'), algors=c('SD','CV','Genorm','NormFinder'), mc.cores=4)
```

## Evaluation signals

- InterOptCuda is found on PATH and produces results; without it, run_experiment() only returns SD/CV, so the presence of Georm/NormFinder columns confirms the kernel is wired in.
- Stability measures are computed on the automatically normalized data (norm_method='high_exp', norm_method_exp_thr=35), so SD/CV/Genorm/NormFinder values should reflect normalized, not raw, CT.
- NormFinder values should reflect the kernel's convention of using only elements with CT<35 for the sample mean.
- Number of combinations matches the k=2 pairs over the CT<37-filtered candidates (rowSums(data>37)==0), e.g. has-miR-1305 pairings in the README output for data_GSE50013.
- result$res_source rows carry weights (w1, w2) alongside CV and SD, and result$aggregated_refs gives one aggregated reference value per sample per combination.

## Limitations

- GeNorm/NormFinder computation depends on the InterOptCuda binary: it must be compiled with nvcc and CUDA is platform-specific (Linux/macOS/Windows install guides referenced).
- The modified kernel restricts Genorm to a supplied list of stable genes and NormFinder to CT<35 elements per sample, so results differ from stock GeNorm/NormFinder implementations.
- The paper notes that aggregating more than 5 RGs by geom(sd) does not have expected benefits compared to the regular geometric mean, so extended-k stability comparisons have diminishing value.

## Evidence

- [intro] GeNorm is an iterative method that uses pairwise variation to measure the stability of RGs.: "GeNorm is an iterative method that uses pairwise variation to measure the stability of RGs."
- [intro] NormFinder separates technical and biological variation: "NormFinder is another widely used tool that suggests a mathematical model that separates technical and biological variations"
- [readme] add InterOptCuda to PATH for Genorm/NormFinder: "In order to have NormFinder and Genorm stability measures in the output, you need to first add `InterOptCuda` to your PATH"
- [readme] modified NormiRazor CUDA kernel: [evidence span withheld — anchor status: external_doc]
- [readme] NormFinder CT<35 sample mean convention: [evidence span withheld — anchor status: external_doc]
- [readme] build InterOptCuda with nvcc: [evidence span withheld — anchor status: external_doc]
