---
name: mutation-aware-grna-annotation-classification
description: Use when analyzing CROP-seq / single-cell CRISPR screen data where gRNAs
  must be assigned to cell barcodes in a mutation-aware manner, e.g. to verify that
  spacer- or scaffold-mutant gRNAs are dysfunctional rather than mislabeled as intact.
when_to_use_negative:
- When the goal is a fine-grained appraisal of which gRNA mutation types or locations
  are tolerable or damaging — the article declares this beyond scope.
- When gRNA reads cannot reach the required depth for accurate assignment and mutation
  detection (specific defined read depths are needed; low gRNA expression vs low detection
  rate confounds assignment).
- When inputs are standard transcriptome-only single-cell data without a gRNA capture
  library — there is no gRNA bam to annotate.
edam_operation: http://edamontology.org/operation_0361
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: GiRAFR
  role: 'Core tool: mutation-aware detection, variant calling, and gRNA-to-cell assignment
    for single-cell CRISPR screens'
  repo: https://github.com/FunctionalEpigeneticsLab/GiRAFR
- name: pysam
  role: Underlying library GiRAFR is built on for reading/manipulating SAM/BAM alignments
  repo: https://github.com/pysam-developers/pysam
- name: samtools
  role: Splits the gRNA alignment file into per-gRNA bam files during gRNA bam filtration
- name: umi_tools
  role: FeatureCount used to filter reads that do not align properly to the target
    gene
- name: Python
  role: Implementation language and environment for downstream Welch t-tests (pandas,
    numpy, scipy.stats.ttest_ind)
- name: Cell Ranger
  role: Default comparison baseline; labels 1-Hamming-distance mutant gRNAs as intact
    and misclassifies multiplets
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-023-05351-7
    title: GiRAFR improves gRNA detection and annotation in single-cell CRISPR screens
  source_content_hash: sha256:36e6da104e19a269a626395e6a076c2e8f1efffc154684f2fc0458487c158279
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FunctionalEpigeneticsLab/GiRAFR
  tools:
  - Cell Ranger
  - GiRAFR
  - Python
  - numpy
  - pandas
  - pysam
  - samtools
  - umi_tools
  edam_operation: http://edamontology.org/operation_0361
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# mutation-aware-grna-annotation-classification

## Summary

Mutation-aware assignment of intact vs mutant (spacer- or scaffold-mutant) gRNAs to single-cell CRISPR screen cells using GiRAFR, followed by functional validation that mutant gRNAs fail to downregulate their target genes via Welch t-tests on expression counts.

## When to use

Use when analyzing CROP-seq / single-cell CRISPR screen data where gRNAs must be assigned to cell barcodes in a mutation-aware manner, e.g. to verify that spacer- or scaffold-mutant gRNAs are dysfunctional rather than mislabeled as intact.

## When NOT to use

- When the goal is a fine-grained appraisal of which gRNA mutation types or locations are tolerable or damaging — the article declares this beyond scope.
- When gRNA reads cannot reach the required depth for accurate assignment and mutation detection (specific defined read depths are needed; low gRNA expression vs low detection rate confounds assignment).
- When inputs are standard transcriptome-only single-cell data without a gRNA capture library — there is no gRNA bam to annotate.

## Inputs

- gRNA library alignment BAM file (from the CRISPR droplet sequencing capture library)
- Oligo pool reference: oligo_pool.csv (oligo_name, sequence), oligo_pool_plasmid.fa / .gtf / _structure.gtf built via prepare.py
- Single-cell transcriptome count matrix and filtered cell barcode list from the expression library mapping
- ConfigFile for GiRAFR (samtools path, output folder settings)

## Outputs

- Filtered bam file gRNA.sorted.mapped.removedSecondaryAlignment.onlyMappedToGrnaChrom.bam
- Consensus sequence bam per UMI-cell barcode combination
- gRNA-to-cell assignment files cells.gRNA.txt and cells.gRNA.single.txt with variant type per cell
- target_gene_expression_welch_ttest.tsv with t-statistics, degrees of freedom, and p-values for intact vs mutant target-gene expression
- QC metrics on mutant gRNA frequency in the library and per dataset

## How to apply

Run GiRAFR on the gRNA library alignments: filter the gRNA bam (utils.gRNA_bam_filter, samtools), construct consensus sequences per UMI-cell barcode combination (min_reads default = 1), call variants against the reference (variant.call_gRNA_variant with the structure GTF), then assign gRNAs to cells with assign_gRNA.assign_gRNA_to_cell using min_umi default = 3 (or auto two-component Gaussian mixture thresholds), producing cells.gRNA.txt and cells.gRNA.single.txt that label each cell as carrying an intact, spacer-mutant, or scaffold-mutant gRNA; assign_gRNA.add_variant_type adds mutation details. Restrict both the single-cell count matrix and gRNA assignments to the shared filtered cell barcodes, extract expression of each target gene for intact vs mutant-gRNA cell groups, and run Welch two-sample t-tests (scipy.stats.ttest_ind, equal_var=False) comparing intact vs mutant classes. The rationale is that Cell Ranger labels single-mutation (1 Hamming distance) gRNAs as intact, which attenuates target-gene downregulation and inflates untransduced-cell estimates, whereas GiRAFR is mutation-aware.

## Related tools

- **GiRAFR** (Core tool: mutation-aware detection, variant calling, and gRNA-to-cell assignment for single-cell CRISPR screens) — https://github.com/FunctionalEpigeneticsLab/GiRAFR
- **pysam** (Underlying library GiRAFR is built on for reading/manipulating SAM/BAM alignments) — https://github.com/pysam-developers/pysam
- **samtools** (Splits the gRNA alignment file into per-gRNA bam files during gRNA bam filtration)
- **umi_tools** (FeatureCount used to filter reads that do not align properly to the target gene)
- **Python** (Implementation language and environment for downstream Welch t-tests (pandas, numpy, scipy.stats.ttest_ind))
- **Cell Ranger** (Default comparison baseline; labels 1-Hamming-distance mutant gRNAs as intact and misclassifies multiplets)

## Examples

```
python -c "from scipy.stats import ttest_ind; t,p = ttest_ind(intact_cells_target_expr, mutant_cells_target_expr, equal_var=False)"
```

## Evaluation signals

- Intact-vs-mutant Welch test on target gene expression must reproduce the reported significant downregulation for intact gRNAs (P = 8.2e-6, consistent with Figure 2b).
- Spacer-mutant and scaffold-mutant gRNA cell groups must show NO significant target-gene downregulation, confirming mutant gRNAs are dysfunctional.
- gRNA-to-cell assignment must respect the min_umi = 3 threshold (or auto two-component Gaussian mixture thresholds) before labeling cells intact vs mutant.
- Downstream analysis must be restricted to the shared filtered cell barcodes from the expression library mapping.
- Output files cells.gRNA.txt and cells.gRNA.single.txt must carry per-cell variant type (intact / spacer-mutant / scaffold-mutant) from assign_gRNA.add_variant_type.

## Limitations

- Missing signal: a fine-grained appraisal of which gRNA mutation types and locations are tolerable or damaging was not performed (declared beyond the scope of the study).
- Missing signal: dramatic inter-experiment differences in average gRNA UMIs may stem from low gRNA expression or low detection rates, which is undetermined and impacts assignment accuracy, ambient vs endogenous discrimination, and singlet vs multiplet discrimination.
- Missing signal: design choices (multiplicity of infection, number of designed gRNAs, introduced mutations, cells per perturbation) influence assignment accuracy and require optimization; specific defined read depths must be attained for accurate gRNA assignment and mutation detection.

## Evidence

- [methods] GiRAFR is a python package: "GiRAFR is a python package"
- [methods] min_umi threshold: "Integer number. By default = 3. Cells with UMIs >= min_umi will be assigned with gRNA."
- [methods] assign step: "assign_gRNA.assign_gRNA_to_cell()"
- [methods] add mutation details: "Functions `assign_gRNA.add_variant_type` and `profile_MT_pattern.py` add mutation details"
- [abstract] mutant gRNAs are dysfunctional: "We show that mutant gRNAs are dysfunctional"
- [discussion] Cell Ranger 1-Hamming mislabeling: "gRNAs with a single mutation (1 Hamming distance) are labeled as intact by CellRanger but similarly show an attenuated phenotype"
- [abstract] up to 35% mutant: "we find that up to 35% of cells are transduced with a mutant gRNA"
- [readme] GiRAFR pysam-based: "We therefore developed GiRAFR, a pysam-based software tool to characterize intact and mutant gRNAs."
- [readme] inflated untransduced estimate: "failure to detect and annotate them leads to an inflated estimate of the number of untransformed cells as well as an underestimated multiplet frequency"
