---
name: gaussian-mixture-model-fitting
description: Use when assigning gRNAs to cells in a single-cell CRISPR screen and
  per-guide UMI count distributions must be split into endogenous vs ambient components
  without a user-defined fixed UMI threshold.
when_to_use_negative:
- Input is a transcriptome gene-expression matrix rather than gRNA UMI counts from
  consensus.sequence.gRNA.variant.txt — this model is for CRISPR gRNA capture data
  only.
- Per-guide UMI distributions are too sparse to support a two-component fit (e.g.,
  dramatically low average gRNA UMIs across the experiment), in which case ambient-vs-endogenous
  discrimination is unreliable and the fixed min_umi=3 fallback should be considered.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2269
tools:
- name: GiRAFR
  role: implements assign_gRNA_to_cell, which fits the two-component Gaussian mixture
    model (auto=True, pool=True) and writes the threshold and assignment output files
  repo: https://github.com/FunctionalEpigeneticsLab/GiRAFR
- name: sklearn
  role: Python library used to fit the two-component Gaussian mixture model per gRNA
- name: Cell Ranger
  role: reference tool that applies the same two-component Gaussian mixture thresholding
    strategy for dynamic UMI thresholds
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-023-05351-7
    title: GiRAFR improves gRNA detection and annotation in single-cell CRISPR screens
  source_content_hash: sha256:a5f0c07e3ab891ca2917e9d559793b5fea5937a42433bd58ce5f5f359eba2218
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FunctionalEpigeneticsLab/GiRAFR
  tools:
  - Cell Ranger
  - GiRAFR
  - Python
  - sklearn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# gaussian-mixture-model-fitting

## Summary

Fits a two-component Gaussian mixture model per gRNA over pooled intact and mutant UMI counts to derive a dynamic per-guide, per-cell UMI threshold that separates endogenously expressed gRNAs from ambient artifacts, replacing the fixed min_umi=3 cutoff.

## When to use

Use when assigning gRNAs to cells in a single-cell CRISPR screen and per-guide UMI count distributions must be split into endogenous vs ambient components without a user-defined fixed UMI threshold.

## When NOT to use

- Input is a transcriptome gene-expression matrix rather than gRNA UMI counts from consensus.sequence.gRNA.variant.txt — this model is for CRISPR gRNA capture data only.
- Per-guide UMI distributions are too sparse to support a two-component fit (e.g., dramatically low average gRNA UMIs across the experiment), in which case ambient-vs-endogenous discrimination is unreliable and the fixed min_umi=3 fallback should be considered.

## Inputs

- per-UMI mutation calls table consensus.sequence.gRNA.variant.txt (intact and mutant gRNA UMI counts per cell barcode)
- filtered cell barcode list from the expression library mapping
- consensus.bam-derived consensus sequence matrix

## Outputs

- gRNA.umi.threshold.txt (one fitted threshold per gRNA)
- consensus.sequence.matrix
- cells.gRNA.txt (cell-to-gRNA assignments passing the fitted threshold)
- cells.gRNA.single.txt (singlet assignments, distinguishing multiplets)

## How to apply

Load per-UMI mutation calls from consensus.sequence.gRNA.variant.txt together with the filtered cell barcode list from the expression-library mapping. For each gRNA, pool intact and mutant UMI counts together (pool=True) so a single distribution per guide is modeled, since mutant gRNAs are otherwise dropped and cells become falsely annotated as untransformed. Fit a two-component Gaussian mixture model with scikit-learn (auto=True) — the same strategy used by Cell Ranger — to obtain an automatically derived per-gRNA UMI threshold in place of the default min_umi=3. Assign gRNAs to cells whose UMI counts meet the fitted threshold, falling back to the fixed min_umi=3 default when auto is disabled, and classify cells as singlets or multiplets. Write gRNA.umi.threshold.txt (one threshold per gRNA), consensus.sequence.matrix, cells.gRNA.txt, and cells.gRNA.single.txt, then verify these outputs against the fitted and fallback thresholds.

## Related tools

- **GiRAFR** (implements assign_gRNA_to_cell, which fits the two-component Gaussian mixture model (auto=True, pool=True) and writes the threshold and assignment output files) — https://github.com/FunctionalEpigeneticsLab/GiRAFR
- **sklearn** (Python library used to fit the two-component Gaussian mixture model per gRNA)
- **Cell Ranger** (reference tool that applies the same two-component Gaussian mixture thresholding strategy for dynamic UMI thresholds)

## Examples

```
from sklearn.mixture import GaussianMixture; gm = GaussianMixture(n_components=2).fit(pooled_umi_counts)
```

## Evaluation signals

- gRNA.umi.threshold.txt exists and contains exactly one derived threshold per gRNA.
- cells.gRNA.txt and cells.gRNA.single.txt are populated only with cells whose gRNA UMI counts satisfy the fitted threshold.
- cells.gRNA.single.txt separates singlets from multiplets; compare multiplet rates against the fixed min_umi=3 baseline, since fixed thresholds underestimate multiplet frequency.
- Re-run with auto disabled and confirm the fallback min_umi=3 threshold reproduces the fixed-threshold assignments.
- Verify the Gaussian mixture converged to two components over the pooled intact+mutant UMI distribution for each guide.

## Limitations

- Dramatic inter-experiment differences in average gRNA UMIs — whether from low gRNA expression or low detection rates — have a detrimental impact on the accuracy of assigning gRNAs to cells, discriminating ambient from endogenously expressed gRNAs, and distinguishing singlets from multiplets.
- Even the two-component Gaussian mixture approach (as used by Cell Ranger) leaves up to 22% of cells labeled as singlets that are in fact multiplets in earlier studies.
- Ad hoc customized pipelines with fixed thresholds for gRNA assignment render head-to-head comparison between studies unfeasible; the fitted thresholds depend on the experiment's UMI distribution.

## Evidence

- [methods] assign_gRNA.assign_gRNA_to_cell(): "assign_gRNA.assign_gRNA_to_cell()"
- [methods] Integer number. By default = 3. Cells with UMIs >= min_umi will be assigned with gRNA.: "Integer number. By default = 3. Cells with UMIs >= min_umi will be assigned with gRNA."
- [discussion] up to 22% of all cells profiled in earlier studies that are labeled as singlets by Cell Ranger are in fact multiplets: "up to 22% of all cells profiled in earlier studies that are labeled as singlets by Cell Ranger are in fact multiplets"
- [discussion] This may either be due to low gRNA expression or low detection rates, but can have a detrimental impact on the accuracy of assigning gRNAs to cells: "This may either be due to low gRNA expression or low detection rates, but can have a detrimental impact on the accuracy of assigning gRNAs to cells"
- [readme] GiRAFR is a python package developed to perform quality control of single cell CRISPR screens: "GiRAFR is a python package developed to perform quality control of single cell CRISPR screens, and to assign gRNAs to cells in a sensitive, mutation-aware manner."
