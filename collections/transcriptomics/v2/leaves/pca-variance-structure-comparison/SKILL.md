---
name: pca-variance-structure-comparison
description: Use when two quantification pipelines have been run on the same RNA-seq
  FASTQ samples and you need to demonstrate equivalence by checking whether normalized
  count tables yield near-identical PCA variance structures and comparable DEG counts.
when_to_use_negative:
- The two pipelines were run on different samples or with different normalization/filtering
  settings, so PCA differences would be confounded by preprocessing rather than quantification
  method
- The goal is de novo functional profiling for a species with no reference transcriptome
  (use Seq2Fun alone rather than a comparison design)
- Input is already a single count table with no second quantification to compare against
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: ExpressAnalystR
  role: R package used locally to normalize count tables, generate QC PCA plots (PlotDataPCA/PlotMetaPCA),
    and run limma differential expression for both quantification tables
  repo: https://github.com/xia-lab/ExpressAnalystR
- name: Kallisto
  role: Reference-transcriptome quantification baseline (v0.46.1) producing the count
    table that Seq2Fun must reproduce
- name: Seq2Fun 2.0
  role: KO-based read quantification (v2.0.2) whose equivalence to the reference-based
    pipeline is being tested via PCA comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-38785-y
    title: 'ExpressAnalyst: A unified platform for RNA-sequencing analysis in non-model
      species'
  source_content_hash: sha256:1117d6f5b2c4efa0cad696a5691b08a47d12dfaf6f8c314a3312ffe36862e077
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/ExpressAnalystR
  tools:
  - ExpressAnalyst
  - ExpressAnalystR
  - Kallisto
  - Seq2Fun
  - Seq2Fun 2.0
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# pca-variance-structure-comparison

## Summary

Compares QC principal component analysis (PCA) variance structures between two quantification methods (e.g., Seq2Fun 2.0.2 KO-based vs. Kallisto 0.46.1 reference-transcriptome-based) on the same RNA-seq samples to establish quantification equivalence. Used in ExpressAnalyst to validate that non-model-species read mapping reproduces reference-based results.

## When to use

Use when two quantification pipelines have been run on the same RNA-seq FASTQ samples and you need to demonstrate equivalence by checking whether normalized count tables yield near-identical PCA variance structures and comparable DEG counts.

## When NOT to use

- The two pipelines were run on different samples or with different normalization/filtering settings, so PCA differences would be confounded by preprocessing rather than quantification method
- The goal is de novo functional profiling for a species with no reference transcriptome (use Seq2Fun alone rather than a comparison design)
- Input is already a single count table with no second quantification to compare against

## Inputs

- Two normalized gene/ortholog count tables for the same samples (e.g., Kallisto transcript-level counts aggregated to genes, and Seq2Fun KO-level counts)
- FASTQ samples shared across pipelines (e.g., SRA accessions SRR13332314–SRR13332325 from NCBI study SRP299836)
- Sample metadata defining treatment groups/contrasts

## Outputs

- QC PCA plots per quantification method (e.g., qc_pca_dpi72.png) for visual comparison of variance structure
- Side-by-side tabulation of DEG counts per treatment contrast (FDR < 0.05, |log2FC| > 1.5) for Kallisto vs. Seq2Fun
- Equivalence assessment comparing against reference figures/tables (Supplementary Figs. 2–3, Tables 1–2)

## How to apply

Quantify the same samples with both pipelines — here, Kallisto 0.46.1 against reference assemblies (zebrafish GRCz11/GCA_000002035.4, lobster GMGI_Hamer_2.0/GCA_018991925.1, minimum quality score = 25) and Seq2Fun 2.0.2 with maximum number of mismatches = 2, minimum matching length = 19, and minimum matching BLOSUM62 score = 80. Normalize both resulting count tables with identical settings (variance filter = 15, abundance filter = 4) so that differences reflect quantification, not preprocessing. Generate QC PCA plots from each normalized table using ExpressAnalystR's PlotDataPCA (single dataset) or PlotMetaPCA (multi-dataset), and visually/numerically compare whether samples cluster by treatment group with similar explained-variance structure. Then run differential expression with limma on both tables using the same thresholds (FDR-adjusted p < 0.05, absolute log2 fold change > 1.5) and tabulate relative DEG counts per treatment contrast to confirm the equivalence extends beyond ordination.

## Related tools

- **ExpressAnalystR** (R package used locally to normalize count tables, generate QC PCA plots (PlotDataPCA/PlotMetaPCA), and run limma differential expression for both quantification tables) — https://github.com/xia-lab/ExpressAnalystR
- **Kallisto** (Reference-transcriptome quantification baseline (v0.46.1) producing the count table that Seq2Fun must reproduce)
- **Seq2Fun 2.0** (KO-based read quantification (v2.0.2) whose equivalence to the reference-based pipeline is being tested via PCA comparison)

## Examples

```
library(ExpressAnalystR); dataSets <- PerformExpressNormalization("kallisto_counts.txt","none",15,4,0); PlotDataPCA("kallisto_counts.txt","qc_pca_",72,"png"); dataSets <- PerformDEAnal("kallisto_counts.txt","custom","control vs. treated","NA","intonly")
```

## Evaluation signals

- PCA variance structures of the two normalized tables are almost identical, with samples separating by treatment group in both (compare against Supplementary Figs. 2–3)
- Relative numbers of DEGs per treatment contrast (FDR < 0.05, |log2FC| > 1.5) are similar between Kallisto and Seq2Fun tables (compare against Supplementary Tables 1–2)
- Both tables used identical filtering: variance filter = 15 and abundance filter = 4
- Identical quantification parameters: Kallisto minimum quality score = 25; Seq2Fun max mismatches = 2, min matching length = 19, min BLOSUM62 score = 80

## Limitations

- Reads from the same gene may map to similar but different orthologs across species, so cross-species comparisons require care in interpretation
- Improper sub-group database selection (e.g., mapping protist reads to the 'fishes' database) may miss key functional groups
- Performance does not improve beyond 16 threads due to I/O constraints
- Sex details of samples were not included in the original manuscript, limiting full reproduction of metadata-driven comparisons

## Evidence

- [results] Seq2Fun 2.0 reproduced results from reference-transcriptome-based analysis for American lobster and zebrafish, producing almost identical PCA variance structures: [evidence span withheld — anchor status: not_found]
- [discussion] Seq2Fun parameter settings for equivalence test: [evidence span withheld — anchor status: not_found]
- [discussion] DEG threshold definition: "Genes were defined as differentially expressed if their false discovery rate (FDR) adjusted p-values were less than 0.05 and the absolute log fold changes greater than 1.5"
- [discussion] Normalization/filter settings: "setting the variance filter to 15 and the abundance filter to 4"
- [readme] README PCA QC plotting function: "PCA plot shows sample separation both between absent and present, and also, low and high. Depending of your experimental design, try to see if the samples are separated by the metadata of interest"
