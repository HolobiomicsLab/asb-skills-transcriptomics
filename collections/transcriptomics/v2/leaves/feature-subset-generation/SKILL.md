---
name: feature-subset-generation
description: 'Use when benchmarking how a co-expression or module-detection tool''s
  runtime scales with the number of features: build nested gene/transcript subsets
  from a fixed-sample expression matrix (e.g., 100-sample ENCODE matrices, up to 16,000
  genes or the full 96,000 transcripts).'
when_to_use_negative:
- The analysis goal is biological module discovery or functional enrichment rather
  than runtime/memory benchmarking — use the full matrix once instead of subsets.
- The dataset is small enough that both implementations run comfortably and scaling
  differences will not be measurable.
- The question concerns accuracy or module-quality differences between packages rather
  than runtime scaling with feature count.
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: PyWGCNA
  role: Tool whose co-expression module identification is run on each feature subset
    to record wall-clock runtime; installed via pip with Python 3.10+
  repo: https://github.com/mortazavilab/PyWGCNA
- name: WGCNA
  role: Reference R implementation run on the same subsets under identical settings
    to provide the baseline runtime and expose memory-constraint failures
- name: Python
  role: Environment for generating subsets and timing PyWGCNA runs; Python 3.9+ required
    by the paper, README specifies 3.10+
  repo: https://github.com/mortazavilab/PyWGCNA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad415
    title: 'PyWGCNA: a Python package for weighted gene co-expression network analysis'
  source_content_hash: sha256:bd1f52cae1b44fbbe58e3f6fd0c55ae382bd7c36f7517c49f63ffa9929a1acde
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mortazavilab/PyWGCNA
  tools:
  - PyWGCNA
  - Python
  - WGCNA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# feature-subset-generation

## Summary

Generates reduced-feature expression-matrix subsets (increasing numbers of genes or transcripts) so that runtime scaling of co-expression module identification can be benchmarked between PyWGCNA and the reference R WGCNA implementation.

## When to use

Use when benchmarking how a co-expression or module-detection tool's runtime scales with the number of features: build nested gene/transcript subsets from a fixed-sample expression matrix (e.g., 100-sample ENCODE matrices, up to 16,000 genes or the full 96,000 transcripts).

## When NOT to use

- The analysis goal is biological module discovery or functional enrichment rather than runtime/memory benchmarking — use the full matrix once instead of subsets.
- The dataset is small enough that both implementations run comfortably and scaling differences will not be measurable.
- The question concerns accuracy or module-quality differences between packages rather than runtime scaling with feature count.

## Inputs

- Gene-level expression matrix (ENCODE, 100 samples)
- Transcript-level expression matrix (ENCODE, 100 samples, up to 96,000 transcripts)
- List of feature counts at which to subset (increasing, up to 16,000 genes)

## Outputs

- Nested feature-subset expression matrices (genes or transcripts at increasing counts)
- Per-replicate wall-clock runtime logs for each package and subset size
- runtime_comparison.csv with columns: feature count, package, mean runtime

## How to apply

Start from the gene-level and transcript-level ENCODE expression matrices (100 samples) used in the paper. Sample features at increasing counts to create nested subsets, capping gene subsets at 16,000 genes and including the full 96,000-transcript matrix as the largest case. Run the tool under comparison (e.g., PyWGCNA co-expression module identification) on each subset three times and record wall-clock runtime per replicate, then run the reference (R WGCNA) under identical settings the same number of times, logging any out-of-memory failures as failed runs. Average the three replicate timings per package per subset size and compile them into a runtime-comparison table (runtime_comparison.csv) with columns for feature count, package, and mean runtime. The rationale is that feature count — not sample count — is the axis on which the R implementation becomes slow or exhausts memory, so controlled feature subsets isolate scaling behavior.

## Related tools

- **PyWGCNA** (Tool whose co-expression module identification is run on each feature subset to record wall-clock runtime; installed via pip with Python 3.10+) — https://github.com/mortazavilab/PyWGCNA
- **WGCNA** (Reference R implementation run on the same subsets under identical settings to provide the baseline runtime and expose memory-constraint failures)
- **Python** (Environment for generating subsets and timing PyWGCNA runs; Python 3.9+ required by the paper, README specifies 3.10+) — https://github.com/mortazavilab/PyWGCNA

## Examples

```
pip install PyWGCNA
```

## Evaluation signals

- runtime_comparison.csv exists with the required schema: feature count, package, and mean runtime columns.
- Measured runtimes show comparable performance between PyWGCNA and R WGCNA up to 16,000 genes.
- Above ~16,000 features, PyWGCNA's mean runtime is roughly 2x faster than R WGCNA's.
- PyWGCNA completes module identification on the full 96,000-transcript matrix while R WGCNA fails with out-of-memory errors on the same input.
- Each subset-size/package cell is backed by three replicate timings that were averaged, not a single run.

## Limitations

- The reference R WGCNA implementation cannot compute a co-expression network on the 96,000-transcript dataset due to memory constraints, so the largest subset yields only one-sided results.
- Runtime comparisons depend on identical settings across packages; differences in default parameters can confound the scaling measurement.
- Timing conclusions are tied to the 100-sample ENCODE matrices used in the paper; scaling behavior may differ with different sample counts or hardware.

## Evidence

- [other] PyWGCNA and R WGCNA had similar performance up to 16,000 genes, but PyWGCNA was twice as fast on larger datasets: [evidence span withheld — anchor status: not_found]
- [other] R WGCNA fails on the largest transcript set: [evidence span withheld — anchor status: not_found]
- [readme] Python version requirement from README: [evidence span withheld — anchor status: external_doc]
- [abstract] Slowness of the R implementation motivating benchmarking: "However, the current R implementation is slow"
