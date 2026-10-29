---
name: ambient-grna-noise-separation
description: Use when assigning gRNAs to cells in a single-cell CRISPR screen where
  ambient (extracellular) gRNA UMIs must be distinguished from endogenous in-cell
  expression before calling singlets vs multiplets, and per-guide thresholds are preferable
  to a fixed UMI cutoff.
when_to_use_negative:
- When benchmarking against published studies that used ad hoc customized pipelines
  with fixed thresholds — the article notes such fixed thresholds render head-to-head
  comparisons unfeasible, so mode choice must match the reference protocol
- When the input lacks per-cell per-gRNA UMI counts (e.g. only raw FASTQ or expression
  counts are available) — upstream GiRAFR steps (consensus sequence construction,
  variant calling) must run first
- When analyzing CRISPR-Cas9-induced DNA editing in transcriptome data — that is a
  separate GiRAFR analysis mode, not cell assignment
tools:
- name: GiRAFR
  role: Hosts the assign_gRNA_to_cell step; implements both fixed min_umi=3 and two-component
    Gaussian mixture dynamic UMI thresholding modes
  repo: https://github.com/FunctionalEpigeneticsLab/GiRAFR
- name: sklearn
  role: Provides the GaussianMixture model fitted per gRNA (auto=True) to derive dynamic
    thresholds from pooled intact and mutant UMI counts
- name: Cell Ranger
  role: Reference implementation that also uses a two-component Gaussian mixture model
    for dynamic UMI thresholding; up to 22% of cells labeled singlets by Cell Ranger
    are in fact multiplets
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-023-05351-7
    title: GiRAFR improves gRNA detection and annotation in single-cell CRISPR screens
  source_content_hash: sha256:9c0b147fc9416975859b65cfaebc0fbe3b379a8d4ff95ce0d7863e0bee4493a7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FunctionalEpigeneticsLab/GiRAFR
  tools:
  - Cell Ranger
  - GiRAFR
  - Python
  - sklearn
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# ambient-grna-noise-separation

## Summary

Performs Step 4 of the GiRAFR pipeline — assigning gRNAs to single cells — by fitting a two-component Gaussian mixture model per gRNA over pooled intact and mutant UMI counts to derive a dynamic per-guide UMI threshold that separates endogenous in-cell UMIs from ambient artifacts, with a fixed min_umi=3 fallback.

## When to use

Use when assigning gRNAs to cells in a single-cell CRISPR screen where ambient (extracellular) gRNA UMIs must be distinguished from endogenous in-cell expression before calling singlets vs multiplets, and per-guide thresholds are preferable to a fixed UMI cutoff.

## When NOT to use

- When benchmarking against published studies that used ad hoc customized pipelines with fixed thresholds — the article notes such fixed thresholds render head-to-head comparisons unfeasible, so mode choice must match the reference protocol
- When the input lacks per-cell per-gRNA UMI counts (e.g. only raw FASTQ or expression counts are available) — upstream GiRAFR steps (consensus sequence construction, variant calling) must run first
- When analyzing CRISPR-Cas9-induced DNA editing in transcriptome data — that is a separate GiRAFR analysis mode, not cell assignment

## Inputs

- consensus.sequence.gRNA.variant.txt (per-UMI mutation calls from consensus sequences)
- Filtered cell barcode list from the expression library mapping
- Per-cell per-gRNA intact and mutant UMI counts

## Outputs

- gRNA.umi.threshold.txt (one fitted threshold per gRNA)
- consensus.sequence.matrix
- cells.gRNA.txt (gRNA-to-cell assignments meeting the threshold)
- cells.gRNA.single.txt (singlet assignments)

## How to apply

Load the per-UMI mutation calls from consensus.sequence.gRNA.variant.txt together with the filtered cell barcode list produced by the expression library mapping. For each gRNA, pool intact and mutant UMI counts together (pool=True) so a single distribution per guide is modeled, then fit a two-component Gaussian mixture model per gRNA with scikit-learn (auto=True) to replace the fixed min_umi=3 default with an automatically derived per-guide threshold — the same GMM approach used by Cell Ranger. Assign a gRNA to a cell when its UMI count meets the fitted threshold (or the fixed min_umi=3 cutoff when auto is disabled), then classify cells as singlets or multiplets. Write gRNA.umi.threshold.txt (one threshold per gRNA), consensus.sequence.matrix, cells.gRNA.txt, and cells.gRNA.single.txt.

## Related tools

- **GiRAFR** (Hosts the assign_gRNA_to_cell step; implements both fixed min_umi=3 and two-component Gaussian mixture dynamic UMI thresholding modes) — https://github.com/FunctionalEpigeneticsLab/GiRAFR
- **sklearn** (Provides the GaussianMixture model fitted per gRNA (auto=True) to derive dynamic thresholds from pooled intact and mutant UMI counts)
- **Cell Ranger** (Reference implementation that also uses a two-component Gaussian mixture model for dynamic UMI thresholding; up to 22% of cells labeled singlets by Cell Ranger are in fact multiplets)

## Examples

```
from girafr import assign_gRNA; assign_gRNA.assign_gRNA_to_cell()
```

## Evaluation signals

- gRNA.umi.threshold.txt contains exactly one threshold per gRNA
- cells.gRNA.txt and cells.gRNA.single.txt contain only cells whose gRNA UMI counts satisfy the fitted (or fixed min_umi=3) threshold
- Singlet vs multiplet calls are consistent: multiplet cells appear in cells.gRNA.txt with multiple gRNAs, singlets in cells.gRNA.single.txt
- Derived thresholds vary across gRNAs in proportion to their UMI count distributions rather than being a single global cutoff
- With auto disabled, assignments reproduce the fixed min_umi=3 default behavior

## Limitations

- Dramatic inter-experiment differences in average gRNA UMIs — whether due to low gRNA expression or low detection rates — can detrimentally impact accuracy of assigning gRNAs to cells and discriminating ambient from endogenously expressed gRNAs
- Specific defined read depths need to be attained for accurate gRNA assignment and mutation detection
- Design choices such as multiplicity of infection, number of designed gRNAs, and number of cells per perturbation influence accurate gRNA assignment and require optimization or parameter finetuning

## Evidence

- [methods] Step4: Assign gRNAs to cells: "Step4: Assign gRNAs to cells"
- [methods] Integer number. By default = 3. Cells with UMIs >= min_umi will be assigned with gRNA.: "Integer number. By default = 3. Cells with UMIs >= min_umi will be assigned with gRNA."
- [methods] assign_gRNA.assign_gRNA_to_cell(): "assign_gRNA.assign_gRNA_to_cell()"
- [discussion] This may either be due to low gRNA expression or low detection rates, but can have a detrimental impact on the accuracy of assigning gRNAs to cells, discriminating ambient from endogenously expressed: "This may either be due to low gRNA expression or low detection rates, but can have a detrimental impact on the accuracy of assigning gRNAs to cells, discriminating ambient from endogenously expressed"
- [readme] to assign gRNAs to cells in a sensitive, mutation-aware manner: "GiRAFR is a python package developed to perform quality control of single cell CRISPR screens, and to assign gRNAs to cells in a sensitive, mutation-aware manner."
