---
name: integration-method-benchmarking-across-repetitions
description: Use when comparing how robust filtering/integration methods are to incomplete
  prior knowledge (e.g., a partial cell-cycle marker gene set), quantified as mean
  percentage degradation of iLISI across repeated runs.
when_to_use_negative:
- When the goal is absolute integration performance on a full, high-quality marker
  set rather than sensitivity to a weakened prior — a direct iLISI comparison without
  the degradation calculation is more appropriate
- When no annotated attributes (cell cycle, sex labels) exist to compute iLISI, since
  degradation cannot be quantified without them
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SiFT (sift-sc)
  role: Primary filtering method under test; builds a cell-cell similarity kernel
    from the marker gene signal to filter cell cycle and sex signals at the gene expression
    level
  repo: https://github.com/nitzanlab/sift-sc
- name: scVI
  role: Latent-space integration baseline whose robustness to the weakened marker
    set is benchmarked against SiFT
  repo: https://github.com/scverse/scvi-tools
- name: pip
  role: Installation channel for the SiFT package (pip install sift-sc)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-44757-7
    title: 'SiFT: uncovering hidden biological processes by probabilistic filtering
      of single-cell data'
  source_content_hash: sha256:97f50658c85629979b68e815f60ff4421f2ebfa79210f116ff03e764ff61df54
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nitzanlab/sift-sc
  tools:
  - SiFT
  - SiFT (sift-sc)
  - pip
  - scVI
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9555555555555556
n_claims: 45
---

# integration-method-benchmarking-across-repetitions

## Summary

Benchmark single-cell filtering/integration methods (SiFT, scanpy regress_out, scVI) for robustness to a weakened prior-knowledge marker set by repeating each method 10 times on Drosophila wing disc data and averaging the percentage degradation of iLISI scores per attribute (cell cycle, cell cycle + sex).

## When to use

Use when comparing how robust filtering/integration methods are to incomplete prior knowledge (e.g., a partial cell-cycle marker gene set), quantified as mean percentage degradation of iLISI across repeated runs.

## When NOT to use

- When the goal is absolute integration performance on a full, high-quality marker set rather than sensitivity to a weakened prior — a direct iLISI comparison without the degradation calculation is more appropriate
- When no annotated attributes (cell cycle, sex labels) exist to compute iLISI, since degradation cannot be quantified without them

## Inputs

- Drosophila wing disc single-cell RNA-seq expression dataset (e.g., AnnData .h5ad)
- Weakened 55-gene marker set with 10 markers replaced by genes of interest (cell cycle / sex markers)
- Cell attribute annotations for cell cycle and sex used for iLISI computation

## Outputs

- degradation_scores.csv with mean percentage iLISI degradation per method (SiFT, regress_out, scVI) and attribute (cell cycle; cell cycle + sex) over 10 repetitions
- Per-repetition iLISI degradation values for each method/attribute combination

## How to apply

Load the Drosophila wing disc single-cell expression dataset together with the weakened 55-gene marker set in which 10 markers are replaced by genes of interest. For each of 10 repetitions, run SiFT (sift.SiFT from sift-sc) with a cell-cell similarity kernel built from the weakened marker signal (sift.kernels.KnnKernel or sift.kernels.MappingKernel) to filter the cell cycle signal and separately the combined cell cycle + sex signal; apply scanpy regress_out on the same covariates as the regression baseline; and train scVI (scvi-tools) as the latent-space integration baseline. After each repetition compute the percentage degradation of iLISI with respect to the cell cycle and cell cycle + sex attributes, average across the 10 repetitions, and write mean degradation per method/attribute to degradation_scores.csv. Rationale: averaging over repetitions and reporting degradation relative to the original marker set isolates sensitivity to prior-knowledge quality rather than raw integration performance; the target reference values are SiFT -0.6%/-4.2%, regress_out -1.0%/-5.9%, scVI -13.5%/-16.3% (Supplementary Fig. 4), showing SiFT degrades minimally versus scVI.

## Related tools

- **SiFT (sift-sc)** (Primary filtering method under test; builds a cell-cell similarity kernel from the marker gene signal to filter cell cycle and sex signals at the gene expression level) — https://github.com/nitzanlab/sift-sc
- **scVI** (Latent-space integration baseline whose robustness to the weakened marker set is benchmarked against SiFT) — https://github.com/scverse/scvi-tools
- **pip** (Installation channel for the SiFT package (pip install sift-sc))

## Examples

```
pip install sift-sc; sft = sift.SiFT(data=adata, kernel=sift.kernels.KnnKernel(n_neighbors=30)); sft.filter(attribute_to_filter=['cell_cycle_genes'], group_pct=0.95)
```

## Evaluation signals

- degradation_scores.csv exists with one row per method/attribute pair and means over 10 repetitions
- Mean iLISI degradation for SiFT is close to the Supplementary Fig. 4 targets: -0.6% (cell cycle) and -4.2% (cell cycle + sex)
- regress_out degradation approximates -1.0%/-5.9% and scVI approximates -13.5%/-16.3%; the ordering SiFT ≈ regress_out < scVI should hold
- Degradation values are negative percentages and reproducible in magnitude across the 10 repetitions (low variance relative to between-method differences)
- SiFT runs used a kernel built from the weakened 55-gene set (not the original marker set) as verified by the input configuration

## Limitations

- SiFT strongly relies on the prior knowledge used to define the cell-cell similarity kernel; an inaccurate or coarse-grained mapping may lead to a loss of signal
- Evaluation depends on integration metrics such as iLISI, which in general are application-specific, and no ground truth reference exists for filtering results
- SiFT modifies gene expression so the corrected data is no longer properly log-transformed, which matters for certain downstream tasks

## Evidence

- [intro] we compute a probabilistic cell-cell similarity kernel, which captures the similarity between cells according to the biological signal we wish to filter: "we compute a probabilistic cell–cell similarity kernel, which captures the similarity between cells according to the biological signal we wish to filter"
- [readme] Install SiFT via PyPI by running pip install sift-sc: "Install SiFT via PyPI by running::

    pip install sift-sc"
- [discussion] SiFT strongly relies on the prior knowledge used to define the cell-cell similarity kernel: "it is important to note that SiFT strongly relies on the prior knowledge used to define the cell–cell similarity kernel."
- [methods] sift.kernels.RBFKernel, KnnKernel, MappingKernel listed in methods: "sift.kernels.RBFKernel
    sift.kernels.LaplacianKernel
    sift.kernels.KnnKernel
    sift.kernels.MappingKernel"
- [intro] applying SiFT to filter nuisance signals in Drosophila wing disc development single-cell data: "applying SiFT to filter nuisance signals in Drosophila wing disc development single-cell data and removing cell cycle effects from a semi-synthetic single-cell dataset"
