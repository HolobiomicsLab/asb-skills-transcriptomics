---
name: single-cell-crispr-grna-cell-assignment-interpretation
description: Use when single-cell CRISPR screen (CROP-seq) data has gRNA assignments
  labeled as intact/spacer-mutant/scaffold-mutant and you must confirm functional
  impact of gRNA mutations on target gene downregulation.
when_to_use_negative:
- When gRNA-to-cell assignments have not yet been generated and mutation-aware annotation
  (intact vs mutant class) is unavailable — run GiRAFR assign_gRNA_to_cell first
- When the goal is detecting CRISPR-Cas9-induced DNA editing or indels passing the
  cutsite rather than assessing gRNA functional impact on expression — use GiRAFR's
  editing-effect analysis mode instead
- When analyzing bulk (non-single-cell) perturbation screens, where per-cell gRNA
  assignment and per-cell expression grouping do not apply
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: GiRAFR
  role: Produces the mutation-aware gRNA-to-cell assignments (intact, spacer-mutant,
    scaffold-mutant) via assign_gRNA_to_cell that are interpreted for functionality
  repo: https://github.com/FunctionalEpigeneticsLab/GiRAFR
- name: Python
  role: Used with pandas/numpy/scipy.stats to load assignments, restrict to shared
    filtered barcodes, extract target gene expression per gRNA class, and run Welch
    t-tests (scipy.stats.ttest_ind, equal_var=False)
- name: pysam
  role: Underlying library on which GiRAFR's gRNA assignment and mutation calling
    is built
  repo: https://github.com/pysam-developers/pysam
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-023-05351-7
    title: GiRAFR improves gRNA detection and annotation in single-cell CRISPR screens
  source_content_hash: sha256:59612319128fb3c2a810709fbef2026a698f2d6e3cb6a30526a1e8a88f47ba13
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FunctionalEpigeneticsLab/GiRAFR
  tools:
  - GiRAFR
  - Python
  - numpy
  - pandas
  - pysam
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# single-cell-crispr-grna-cell-assignment-interpretation

## Summary

Interpret GiRAFR gRNA-to-cell assignments in a single-cell CRISPR screen by testing whether cells carrying intact vs mutant (spacer- or scaffold-mutant) gRNAs show differential expression of the gRNA target genes, thereby empirically validating gRNA functionality.

## When to use

Use when single-cell CRISPR screen (CROP-seq) data has gRNA assignments labeled as intact/spacer-mutant/scaffold-mutant and you must confirm functional impact of gRNA mutations on target gene downregulation.

## When NOT to use

- When gRNA-to-cell assignments have not yet been generated and mutation-aware annotation (intact vs mutant class) is unavailable — run GiRAFR assign_gRNA_to_cell first
- When the goal is detecting CRISPR-Cas9-induced DNA editing or indels passing the cutsite rather than assessing gRNA functional impact on expression — use GiRAFR's editing-effect analysis mode instead
- When analyzing bulk (non-single-cell) perturbation screens, where per-cell gRNA assignment and per-cell expression grouping do not apply

## Inputs

- GiRAFR gRNA-to-cell assignment files: cells.gRNA.txt and cells.gRNA.single.txt
- Single-cell transcriptome count matrix (raw or normalized counts) from the expression library
- Filtered cell barcode list from the expression library mapping
- gRNA-to-target-gene mapping (which target gene each gRNA perturbs)

## Outputs

- Table of Welch t-test statistics, degrees of freedom, and p-values per target gene comparison (target_gene_expression_welch_ttest.tsv)
- Functional classification of mutant gRNA classes (dysfunctional vs functional) based on target gene downregulation

## How to apply

Load the GiRAFR gRNA-to-cell assignment outputs (cells.gRNA.txt and cells.gRNA.single.txt, produced by assign_gRNA.assign_gRNA_to_cell with min_umi default = 3 or auto two-component Gaussian mixture thresholds) to label each cell barcode as carrying an intact, spacer-mutant, or scaffold-mutant gRNA. Load the single-cell transcriptome count matrix and the filtered barcode list from the expression library, and restrict both the count matrix and gRNA assignments to the shared filtered cell barcodes. For each target gene, extract expression values (raw or normalized counts) for the three cell groups defined by gRNA class: intact, spacer-mutant, and scaffold-mutant. Run Welch two-sample t-tests (scipy.stats.ttest_ind with equal_var=False) comparing target gene expression between intact-gRNA cells and each mutant-gRNA class, and against untransduced cells, then assemble t-statistics, degrees of freedom, and p-values into a table written to target_gene_expression_welch_ttest.tsv. The rationale is that mutant gRNAs are dysfunctional: intact gRNAs should significantly reduce target gene expression while spacer- and scaffold-mutant gRNAs should fail to downregulate the target, so the direction and significance of the tests directly validate the mutation-aware annotation.

## Related tools

- **GiRAFR** (Produces the mutation-aware gRNA-to-cell assignments (intact, spacer-mutant, scaffold-mutant) via assign_gRNA_to_cell that are interpreted for functionality) — https://github.com/FunctionalEpigeneticsLab/GiRAFR
- **Python** (Used with pandas/numpy/scipy.stats to load assignments, restrict to shared filtered barcodes, extract target gene expression per gRNA class, and run Welch t-tests (scipy.stats.ttest_ind, equal_var=False))
- **pysam** (Underlying library on which GiRAFR's gRNA assignment and mutation calling is built) — https://github.com/pysam-developers/pysam

## Examples

```
from scipy.stats import ttest_ind; t, df, p = ttest_ind(intact_expr, spacer_mut_expr, equal_var=False)
```

## Evaluation signals

- Intact-gRNA cells must show statistically significant downregulation of their target genes, consistent with the reported Figure 2b statistic (P = 8.2e-6)
- Spacer-mutant and scaffold-mutant gRNA cell groups must show NO significant downregulation of their target genes in the Welch t-tests
- The count matrix and gRNA assignments must be restricted to the same set of shared filtered cell barcodes before testing
- Output table target_gene_expression_welch_ttest.tsv must contain t-statistics, degrees of freedom, and p-values for intact-vs-mutant and intact-vs-untransduced comparisons
- Cell assignment threshold consistency: cells were assigned gRNAs with min_umi default = 3 (or auto two-component Gaussian mixture thresholds) — verify the same thresholding was used

## Limitations

- Fine-grained appraisal of which gRNA mutation types and locations are tolerable or damaging was not performed (declared beyond the scope of the study)
- Inter-experiment differences in average gRNA UMIs (low gRNA expression vs low detection rates) can impact accuracy of assigning gRNAs to cells and discriminating singlets from multiplets
- Failure to detect and annotate mutant gRNAs leads to an inflated estimate of untransduced cells, attenuated downregulation of target genes, and underestimated multiplet frequency — up to 35% of cells in public datasets carry mutant gRNAs
- Design choices (multiplicity of infection, number of designed gRNAs, introduced mutations, cells per perturbation) influence accurate gRNA assignment and require optimization

## Evidence

- [results] gRNAs with a mutant spacer and scaffold failed to reduce expression of their target genes, whereas intact gRNAs significantly reduced target gene expression: [evidence span withheld — anchor status: not_found]
- [methods] min_umi default threshold: "Integer number. By default = 3. Cells with UMIs >= min_umi will be assigned with gRNA."
- [methods] assign gRNAs to cells workflow step: "Functions `assign_gRNA.add_variant_type` and `profile_MT_pattern.py` add mutation details for downstream analysis."
- [abstract] mutant gRNAs are dysfunctional: "We show that mutant gRNAs are dysfunctional"
- [abstract] consequence of failing to annotate mutants: "failure to detect and annotate them leads to an inflated estimate of the number of untransformed cells, attenuated downregulation of target genes"
- [readme] GiRAFR README installation: "GiRAFR is a python package developed to perform quality control of single cell CRISPR screens, and to assign gRNAs to cells in a sensitive, mutation-aware manner."
