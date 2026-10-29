---
name: cross-method-quantification-benchmarking
description: Use when an organism has a reference genome/transcriptome (e.g., zebrafish
  GRCz11, Homarus americanus GMGI_Hamer_2.0) and you must validate that Seq2Fun ortholog-based
  quantification reproduces Kallisto-based results.
when_to_use_negative:
- The organism lacks a reference transcriptome (e.g., the non-model salamander species
  in this study) — there is no Kallisto baseline to benchmark against, so only Seq2Fun
  ortholog-based quantification applies
- Input is already a normalized count or gene expression matrix — use ExpressAnalyst's
  matrix-based workflow (ReadTabExpressData/PerformExpressNormalization) instead of
  re-quantifying reads
- The goal is de novo transcriptome assembly or annotation transfer (Trinity, SOAPdenovo-Trans,
  Oases + BLAST) rather than comparing quantification strategies
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: Kallisto
  role: Reference-transcriptome-based baseline quantifier (version 0.46.1, minimum
    quality score 25) whose results Seq2Fun must reproduce
- name: Seq2Fun
  role: Ortholog-based quantifier (version 2.0.2; mismatches=2, min matching length=19,
    min BLOSUM62 score=80) being benchmarked against Kallisto
- name: ExpressAnalyst
  role: Web platform/R package for normalizing both count tables, generating QC PCA
    plots (PlotDataPCA/PlotMetaPCA), and running limma differential expression
  repo: github.com/xia-lab/ExpressAnalystR
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-38785-y
    title: 'ExpressAnalyst: A unified platform for RNA-sequencing analysis in non-model
      species'
  source_content_hash: sha256:6eafe480ecf486ec7bc090b3f8385f232232c85ad1d92444ee0309147c95d990
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ExpressAnalyst
  - Kallisto
  - Seq2Fun
  - Seq2Fun 2.0
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cross-method-quantification-benchmarking

## Summary

Benchmarks ortholog-based quantification (Seq2Fun 2.0) against reference-transcriptome-based quantification (Kallisto) on the same RNA-seq samples by comparing normalized count tables, PCA variance structures, and relative DEG counts per treatment contrast.

## When to use

Use when an organism has a reference genome/transcriptome (e.g., zebrafish GRCz11, Homarus americanus GMGI_Hamer_2.0) and you must validate that Seq2Fun ortholog-based quantification reproduces Kallisto-based results.

## When NOT to use

- The organism lacks a reference transcriptome (e.g., the non-model salamander species in this study) — there is no Kallisto baseline to benchmark against, so only Seq2Fun ortholog-based quantification applies
- Input is already a normalized count or gene expression matrix — use ExpressAnalyst's matrix-based workflow (ReadTabExpressData/PerformExpressNormalization) instead of re-quantifying reads
- The goal is de novo transcriptome assembly or annotation transfer (Trinity, SOAPdenovo-Trans, Oases + BLAST) rather than comparing quantification strategies

## Inputs

- RNA-seq FASTQ files (paired-end; e.g., NCBI SRA accessions SRR13332314–SRR13332325, study SRP299836)
- Reference genome/transcriptome assemblies (GRCz11 GCA_000002035.4 for Danio rerio; GMGI_Hamer_2.0 GCA_018991925.1 for Homarus americanus)
- Sample metadata / treatment group definitions for DE contrasts

## Outputs

- Two count tables (KO-level from Seq2Fun 2.0.2; transcript/gene-level from Kallisto 0.46.1) for the same samples
- QC PCA plots comparing variance structures of the two quantification methods
- DEG tables per treatment contrast (FDR < 0.05, |log2FC| > 1.5) with relative DEG counts for Kallisto vs. Seq2Fun

## How to apply

Download paired FASTQ samples (e.g., SRR13332314–SRR13332325 from NCBI SRA study SRP299836) and quantify them twice: once with Kallisto 0.46.1 against a reference assembly (GRCz11 GCA_000002035.4 for zebrafish; GMGI_Hamer_2.0 GCA_018991925.1 for lobster) with the minimum quality score parameter set to 25, and once with Seq2Fun 2.0.2 using maximum number of mismatches = 2, minimum matching length = 19, and minimum matching BLOSUM62 score = 80. Normalize both count tables in ExpressAnalyst and generate QC PCA plots (PlotDataPCA/PlotMetaPCA) to compare variance structures between quantification methods. Run differential expression with limma in ExpressAnalyst, defining DEGs as FDR-adjusted p < 0.05 and absolute log fold change > 1.5. The rationale is that equivalence of PCA structure and relative DEG counts demonstrates that ortholog-based quantification is a valid substitute for reference-transcriptome workflows, avoiding de novo assembly entirely.

## Related tools

- **Kallisto** (Reference-transcriptome-based baseline quantifier (version 0.46.1, minimum quality score 25) whose results Seq2Fun must reproduce)
- **Seq2Fun** (Ortholog-based quantifier (version 2.0.2; mismatches=2, min matching length=19, min BLOSUM62 score=80) being benchmarked against Kallisto)
- **ExpressAnalyst** (Web platform/R package for normalizing both count tables, generating QC PCA plots (PlotDataPCA/PlotMetaPCA), and running limma differential expression) — github.com/xia-lab/ExpressAnalystR

## Examples

```
PlotDataPCA("estrogen.txt", "qc_pca_", 72, "png");
```

## Evaluation signals

- PCA plots from normalized Seq2Fun and Kallisto count tables show almost identical variance structures (samples cluster the same way by treatment group)
- Relative numbers of DEGs per treatment contrast (FDR < 0.05, |log2FC| > 1.5) are consistent between Kallisto and Seq2Fun, matching the paper's Supplementary Figs. 2–3 and Tables 1–2
- Both quantifications use the same raw FASTQ inputs (SRR13332314–SRR13332325) and identical normalization and limma DE settings, so the only differing variable is the quantification method
- Seq2Fun run reports the expected parameter set (max mismatches = 2, min matching length = 19, min BLOSUM62 score = 80) in its log/configuration

## Limitations

- KO-based annotation has limited transcriptome coverage (61.40% of zebrafish genes have KOs) and some biomarkers such as vitellogenin are not found in the KO system, so KO-level DEG counts are not directly comparable to full gene-level counts
- Reads from the same gene may map to similar but different orthologs for different species, requiring care in interpretation for taxonomically diverse comparisons
- Improper ortholog sub-group database selection (e.g., mapping a protist sample against the 'fishes' database) may miss key functional groups; the paper tested cross-species mapping with a customized 'Fishes' database with zebrafish sequences removed
- Seq2Fun performance does not improve beyond 16 threads due to an I/O constraint, though Seq2Fun 2.0 reduced memory footprint (e.g., from 1.49 GB to 0.94 GB)

## Evidence

- [discussion] quantification based on Kallisto (version 0.46.1) or Seq2Fun (version 2.0.2): "quantification based on Kallisto (version 0.46.1) or Seq2Fun (version 2.0.2)"
- [discussion] The minimum quality score parameter was set to 25.: "The minimum quality score parameter was set to 25."
- [discussion] “maximum number of mismatches” = 2, “minimum matching length” = 19, and “minimum matching BLOSUM62 score” = 80.: "“maximum number of mismatches” = 2, “minimum matching length” = 19, and “minimum matching BLOSUM62 score” = 80."
- [discussion] Genes were defined as differentially expressed if their false discovery rate (FDR) adjusted p-values were less than 0.05 and the absolute log fold changes greater than 1.5.: "Genes were defined as differentially expressed if their false discovery rate (FDR) adjusted p-values were less than 0.05 and the absolute log fold changes greater than 1.5."
- [readme] Check diagnostics plots to look at overall data distribution, sample separation.: "Check diagnostics plots to look at overall data distribution, sample separation."
