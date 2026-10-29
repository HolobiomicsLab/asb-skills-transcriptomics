---
name: ecdna-gene-annotation
description: Use when you have ecDNA interval calls (e.g., circlehunter Snakemake
  output) over tumor ATAC-Seq samples and need to count amplified genes, flag known
  oncogenes, and test their functional enrichment.
when_to_use_negative:
- Input is raw ATAC-Seq FASTQ or unprocessed signal — ecDNA calls must first be produced
  by the circlehunter Snakemake pipeline before gene annotation can be applied
- Your ecDNA calls come from WGS-based frameworks (e.g., AmpliconArchitect) with different
  coordinate semantics; this skill is calibrated on circlehunter output from ATAC-Seq
  data
- The goal is single-cell ecDNA characterization — single-cell ATAC-Seq application
  of circlehunter is unexplored per the article's discussion
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0622
tools:
- name: circlehunter
  role: Produces the ecDNA calls whose overlapped genes are annotated and functionally
    characterized in this skill
  repo: https://github.com/suda-huanglab/circlehunter
- name: Snakemake
  role: Workflow manager that runs the circlehunter pipeline generating the upstream
    ecDNA call output consumed here
- name: GSEApy
  role: Used for the over-representation/enrichment test of somatic mutations among
    ecDNA-carried genes
  repo: https://github.com/zqfang/GSEApy
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41389-023-00476-0
    title: 'Circlehunter: a tool to identify extrachromosomal circular DNA from ATAC-Seq
      data'
  source_content_hash: sha256:1d3636871217e245dde3a3e47a05d94de35fb16cceadbb91bc7cb25cba47628a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/suda-huanglab/circlehunter
  tools:
  - GSEApy
  - Snakemake
  - circlehunter
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# ecdna-gene-annotation

## Summary

Annotate genes overlapped by ecDNA calls (from circlehunter ATAC-Seq analysis) and characterize their oncogenic properties by joining copy-number, expression, CRISPR gene-effect, PCAWG gain, and somatic mutation data, yielding per-gene amplification status with five enrichment P values.

## When to use

Use when you have ecDNA interval calls (e.g., circlehunter Snakemake output) over tumor ATAC-Seq samples and need to count amplified genes, flag known oncogenes, and test their functional enrichment.

## When NOT to use

- Input is raw ATAC-Seq FASTQ or unprocessed signal — ecDNA calls must first be produced by the circlehunter Snakemake pipeline before gene annotation can be applied
- Your ecDNA calls come from WGS-based frameworks (e.g., AmpliconArchitect) with different coordinate semantics; this skill is calibrated on circlehunter output from ATAC-Seq data
- The goal is single-cell ecDNA characterization — single-cell ATAC-Seq application of circlehunter is unexplored per the article's discussion

## Inputs

- ecDNA interval calls from the circlehunter Snakemake pipeline (upstream task_005 output) for all analyzed tumor ATAC-Seq samples
- Gene annotation (genomic coordinates of genes overlapped by ecDNA calls)
- CCLE/DepMap copy-number, expression, and CRISPR gene-effect tables
- PCAWG copy-number-gain calls
- Somatic mutation data for over-representation testing

## Outputs

- Gene-level table (figure4_gene_analysis.csv) with per-gene ecDNA amplification status, oncogene annotation, and five enrichment P values (copy number, expression, gene effect, PCAWG coincidence, somatic mutation)
- Counts of total amplified genes (251) and known oncogenes (37) within ecDNAs

## How to apply

Load the ecDNA calls produced by the circlehunter Snakemake pipeline for all analyzed samples, then annotate the genes overlapped by each ecDNA call — in the source study this recovered 251 amplified genes (about 10 genes per ecDNA) across 1312 predicted ecDNAs from 94 public ATAC-Seq datasets, of which 37 were known oncogenes. Next, join the amplified-gene set with CCLE/DepMap copy-number, expression, and CRISPR gene-effect tables to test enrichment of oncogene copy number (P = 8.754e-16), expression (P = 2.047e-8), and gene effect (P = 8.408e-5). Intersect the ecDNA-amplified genes with PCAWG copy-number-gain calls and test their coincidence (P = 1.619e-15). Finally, test somatic mutation enrichment among ecDNA-carried genes using a GSEApy-based over-representation approach (P = 4.986e-2) and write the gene-level table with per-gene ecDNA amplification status, oncogene annotation, and all five P values to figure4_gene_analysis.csv.

## Related tools

- **circlehunter** (Produces the ecDNA calls whose overlapped genes are annotated and functionally characterized in this skill) — https://github.com/suda-huanglab/circlehunter
- **Snakemake** (Workflow manager that runs the circlehunter pipeline generating the upstream ecDNA call output consumed here)
- **GSEApy** (Used for the over-representation/enrichment test of somatic mutations among ecDNA-carried genes) — https://github.com/zqfang/GSEApy

## Examples

```
snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml
```

## Evaluation signals

- The annotated gene set totals 251 amplified genes across ecDNA calls, with an average of about 10 genes per ecDNA, of which 37 are known oncogenes
- The output figure4_gene_analysis.csv contains exactly five P-value columns: copy-number (8.754e-16), expression (2.047e-8), gene effect (8.408e-5), PCAWG coincidence (1.619e-15), and somatic mutation (4.986e-2)
- Every gene row carries both a binary ecDNA amplification status and an oncogene annotation flag
- Enrichment P values for copy number, expression, and gene effect are all well below 0.05, consistent with the article's reported oncogene amplification characteristics
- Intersection with PCAWG copy-number-gain calls is non-empty and yields a significant coincidence P value (1.619e-15)

## Limitations

- The current benchmarking strategy has limitations, and accuracy of ecDNA calls for all types of focal amplifications cannot be guaranteed — misannotated calls propagate into the gene set
- Most analyzed samples were cell lines and cancer-type proportions deviated significantly from each other; additional data are needed for generalization
- Short read-length sequencing may lead circlehunter to discover more computational sub-structures (e.g., COLO320DM), inflating gene counts per ecDNA
- If other classes of focal amplification have misregulated chromatin, they may be detected as false-positive ecDNA by circlehunter and spuriously contribute amplified genes

## Evidence

- [abstract] ecDNA calls annotated to 251 genes, 37 oncogenes: "Based on 1312 ecDNAs predicted from 94 publicly available datasets of ATAC-Seq assays, we found 37 oncogenes contained in these ecDNAs"
- [abstract] circlehunter identifies ecDNA from ATAC-Seq via enhanced accessibility: "We developed circlehunter ( https://github.com/suda-huanglab/circlehunter ), a tool for identifying ecDNA from ATAC-Seq data"
- [methods] Snakemake pipeline invocation: "snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml"
- [other] GSEApy as enrichment tooling: [evidence span withheld — anchor status: external_doc]
- [intro] ecDNA carries oncogenes such as MYC, MYCN, EGFR: "an increasing number of ecDNAs have been found to carry important oncogenes in different types of cancer [9], including MYC, MYCN, and EGFR"
