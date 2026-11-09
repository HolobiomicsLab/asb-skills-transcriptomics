---
name: kallisto-reference-assembly-pseudoalignment
description: Use when RNA-seq FASTQ samples come from an organism with a published
  reference assembly and you need gene-level counts to benchmark against reference-free
  (Seq2Fun/ExpressAnalyst) quantification, or to produce reference-based count tables
  for normalization, PCA QC, and limma differential.
when_to_use_negative:
- The organism has no reference transcriptome/assembly — use Seq2Fun/ExpressAnalyst
  KO-based quantification instead
- Input is already a count or expression matrix — skip quantification and start from
  ExpressAnalystR annotation/normalization (PerformDataAnnot, PerformExpressNormalization)
- The goal is functional/pathway profiling on KEGG orthologs rather than reference-based
  equivalence testing
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Kallisto
  role: Pseudoalignment quantification of reads against the reference assembly (version
    0.46.1, minimum quality score = 25) to build the reference-based count table
- name: ExpressAnalystR
  role: Normalization, QC PCA (PlotDataPCA/PlotMetaPCA), and downstream limma differential
    expression on the Kallisto count table
  repo: github.com/xia-lab/ExpressAnalystR
- name: limma
  role: Differential expression algorithm applied to the normalized Kallisto counts
    to compute per-contrast DEG counts
- name: Seq2Fun 2.0.2
  role: Reference-free comparator whose PCA structure and DEG counts are checked for
    equivalence against the Kallisto-based results
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-38785-y
    title: 'ExpressAnalyst: A unified platform for RNA-sequencing analysis in non-model
      species'
  source_content_hash: sha256:d2a7316d011dea0f77dd8a8fa47c45aa3ed72b18438a77045741b78c8b4e2ff0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ExpressAnalyst
  - ExpressAnalystR
  - Kallisto
  - Seq2Fun
  - Seq2Fun 2.0
  - Seq2Fun 2.0.2
  - limma
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Kallisto reference-assembly pseudoalignment quantification

## Summary

Quantifies RNA-seq FASTQ reads against a reference genome/transcriptome assembly using Kallisto pseudoalignment, serving as the traditional reference-based benchmark against which KO-based Seq2Fun quantification is compared in ExpressAnalyst equivalence studies.

## When to use

Use when RNA-seq FASTQ samples come from an organism with a published reference assembly and you need gene-level counts to benchmark against reference-free (Seq2Fun/ExpressAnalyst) quantification, or to produce reference-based count tables for normalization, PCA QC, and limma differential expression.

## When NOT to use

- The organism has no reference transcriptome/assembly — use Seq2Fun/ExpressAnalyst KO-based quantification instead
- Input is already a count or expression matrix — skip quantification and start from ExpressAnalystR annotation/normalization (PerformDataAnnot, PerformExpressNormalization)
- The goal is functional/pathway profiling on KEGG orthologs rather than reference-based equivalence testing

## Inputs

- RNA-seq FASTQ files (paired-end), e.g., NCBI SRA accessions SRR13332314–SRR13332325
- Reference genome/transcriptome assembly, e.g., GRCz11 GCA_000002035.4 (zebrafish) or GMGI_Hamer_2.0 GCA_018991925.1 (lobster)
- Sample metadata / experimental design (treatment groups) for limma contrasts

## Outputs

- Kallisto transcript/gene-level count table per sample
- Normalized expression matrix with QC PCA plot (e.g., qc_pca_dpi72.png via ExpressAnalystR PlotDataPCA)
- DEG table per treatment contrast (FDR < 0.05, |log2FC| > 1.5) for comparison against Seq2Fun results

## How to apply

Download the RNA-seq FASTQ files (e.g., accessions SRR13332314–SRR13332325 from NCBI SRA study SRP299836) and quantify them with Kallisto version 0.46.1 against the organism's reference assembly — for the paper's zebrafish case, the GRCz11 Danio rerio assembly GCA_000002035.4, and for American lobster, the GMGI_Hamer_2.0 assembly GCA_018991925.1 — setting the minimum quality score parameter to 25. Sum transcript-level estimates to gene level, then import the count table into ExpressAnalyst/ExpressAnalystR for normalization (variance filter 15, abundance filter 4), QC PCA plots, and limma differential expression with DEGs defined as FDR-adjusted p < 0.05 and absolute log2 fold change > 1.5. The rationale is to generate the reference-transcriptome-based result that Seq2Fun 2.0.2 (max mismatches = 2, min matching length = 19, min BLOSUM62 score = 80) must reproduce: compare PCA variance structures and relative per-contrast DEG counts between the two quantification methods to establish equivalence.

## Related tools

- **Kallisto** (Pseudoalignment quantification of reads against the reference assembly (version 0.46.1, minimum quality score = 25) to build the reference-based count table)
- **ExpressAnalystR** (Normalization, QC PCA (PlotDataPCA/PlotMetaPCA), and downstream limma differential expression on the Kallisto count table) — github.com/xia-lab/ExpressAnalystR
- **limma** (Differential expression algorithm applied to the normalized Kallisto counts to compute per-contrast DEG counts)
- **Seq2Fun 2.0.2** (Reference-free comparator whose PCA structure and DEG counts are checked for equivalence against the Kallisto-based results)

## Examples

```
dataSets <- PerformExpressNormalization("estrogen.txt", "none", 15, 4, 0); dataSets <- SetupDesignMatrix("estrogen.txt", "limma"); dataSets <- PerformDEAnal("estrogen.txt", "custom", "treated vs. control", "NA", "intonly"); PlotDataPCA("estrogen.txt", "qc_pca_", 72, "png");
```

## Evaluation signals

- PCA variance structure of normalized Kallisto counts is nearly identical to the Seq2Fun 2.0.2 PCA (Supplementary Figs. 2–3)
- Relative numbers of DEGs per treatment contrast agree between Kallisto and Seq2Fun (Supplementary Tables 1–2)
- All FASTQ accessions (SRR13332314–SRR13332325) quantified with Kallisto 0.46.1 and the correct assembly accessions (GCA_000002035.4 / GCA_018991925.1)
- Kallisto minimum quality score parameter = 25; DEG thresholds exactly FDR-adjusted p < 0.05 and |log2FC| > 1.5

## Limitations

- Requires a reference transcriptome/assembly, so it cannot serve organisms without one (e.g., two of the paper's salamander species)
- Reference-based analysis is computationally intensive and produces transcript IDs (usually in-house IDs) that are difficult to reproduce, compare across studies, and re-use
- Reference genomes of non-model organisms still contain many uncharacterized IDs, complicating gene-level interpretation

## Evidence

- [discussion] The minimum quality score parameter was set to 25.: "The minimum quality score parameter was set to 25."
- [discussion] Kallisto version pin and comparator: "quantification based on Kallisto (version 0.46.1) or Seq2Fun (version 2.0.2)"
- [discussion] Reference assembly used for zebrafish alignment: "zebrafish samples were aligned to the GRCz11 Danio rerio genome assembly (accession: GCA_000002035.4"
- [discussion] DEG threshold definition: "Genes were defined as differentially expressed if their false discovery rate (FDR) adjusted p-values were less than 0.05 and the absolute log fold changes greater than 1.5."
- [readme] PCA as QC for sample separation: "PCA plot shows sample separation both between absent and present, and also, low and high."
