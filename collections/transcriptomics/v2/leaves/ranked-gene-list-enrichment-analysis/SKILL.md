---
name: ranked-gene-list-enrichment-analysis
description: Use when scTenifoldNet's diffRegulation output (genes ranked by differential
  regulation distance with BH-adjusted p-values) is available and the goal is to identify
  enriched biological processes from that ranked list.
when_to_use_negative:
- The two samples are highly dissimilar (e.g., different cell types) rather than two
  populations of cells of the same type — the article notes scTenifoldNet is more
  suitable for comparing highly similar samples.
- The analysis goal is differential expression of individual genes rather than differential
  transcriptional regulation; a DE tool, not comparative GRN analysis plus enrichment,
  is the right choice.
- The input is already a curated gene set or enrichment-ready table with no gene-level
  ranking statistic; re-ranking through the GRN pipeline is unnecessary.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scTenifoldNet
  role: Produces the ranked gene list (diffRegulation output) that feeds GSEA; installed
    via install.packages("scTenifoldNet")
  repo: github.com/cailab-tamu/scTenifoldNet
- name: R
  role: Runtime environment in which scTenifoldNet is run, the diffRegulation table
    is filtered (FDR < 0.05), and GSEA is executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.patter.2020.100139
    title: 'scTenifoldNet: A Machine Learning Workflow for Constructing and Comparing
      Transcriptome-wide Gene Regulatory Networks from Single-Cell Data'
  source_content_hash: sha256:e82f71783ecba1faae4a148264ea100e3cb39be6bd45b3b6104885a1e83e8d10
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/cailab-tamu/scTenifoldNet
  tools:
  - R
  - scTenifoldNet
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# ranked-gene-list-enrichment-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Interprets scTenifoldNet's ranked differential-regulation output by running gene set enrichment analysis (GSEA) on genes ordered by their differential regulation distance, to recover biological programs such as opioid signaling, GPCR signaling, and reduced cytosolic calcium among differentially regulated genes.

## When to use

Use when scTenifoldNet's diffRegulation output (genes ranked by differential regulation distance with BH-adjusted p-values) is available and the goal is to identify enriched biological processes from that ranked list.

## When NOT to use

- The two samples are highly dissimilar (e.g., different cell types) rather than two populations of cells of the same type — the article notes scTenifoldNet is more suitable for comparing highly similar samples.
- The analysis goal is differential expression of individual genes rather than differential transcriptional regulation; a DE tool, not comparative GRN analysis plus enrichment, is the right choice.
- The input is already a curated gene set or enrichment-ready table with no gene-level ranking statistic; re-ranking through the GRN pipeline is unnecessary.

## Inputs

- scTenifoldNet diffRegulation data frame (gene, distance, Z, FC, p.value, p.adj) from comparing two scGRNs
- Raw counts matrices (genes as rows, cell barcodes as columns) for the two conditions, e.g., 7,972 morphine-treated and 8,912 mock-treated nucleus accumbens neurons

## Outputs

- Ranked gene list ordered by differential regulation level
- GSEA enrichment results identifying enriched biological processes (e.g., opioid signaling, GPCR signaling, reduction of cytosolic calcium levels, morphine addiction)
- CSV export of the diffRegulation table (e.g., diffRegulation_morphine_vs_mock.csv)

## How to apply

Run the scTenifoldNet pipeline to obtain the diffRegulation data frame, whose columns (gene, distance, Z, FC, p.value, p.adj) provide the ranking: genes are sorted according to the differential regulation level, where p.adj is the Benjamini & Hochberg FDR-corrected chi-square (1 df) p-value from dRegulation's Box-Cox-transformed distances. Filter or rank the list — in the morphine-vs-mock nucleus accumbens application, genes with FDR < 0.05 (~56 genes) formed the significant set. Feed the full ranked gene list (not just the significant subset) into GSEA to detect coordinated shifts in gene expression programs. Verify that enriched terms reproduce the expected biology; for the morphine dataset these are opioid signaling, GPCR signaling, reduction of cytosolic calcium levels, and morphine addiction. Save the table (e.g., diffRegulation_morphine_vs_mock.csv) together with the enrichment results for reporting.

## Related tools

- **scTenifoldNet** (Produces the ranked gene list (diffRegulation output) that feeds GSEA; installed via install.packages("scTenifoldNet")) — github.com/cailab-tamu/scTenifoldNet
- **R** (Runtime environment in which scTenifoldNet is run, the diffRegulation table is filtered (FDR < 0.05), and GSEA is executed)

## Examples

```
out <- scTenifoldNet(X = mock, Y = morphine, nc_nNet = 10, nc_nCells = 500, td_K = 3); ranked <- out$diffRegulation[order(-out$diffRegulation$distance), c("gene", "distance", "p.adj")]
```

## Evaluation signals

- diffRegulation output has exactly six columns: gene, distance, Z, FC, p.value, p.adj.
- Under the null (H0: identical matrices supplied for X and Y), no genes reach significance at FDR < 0.1 in the QQ-plot summary.
- Under the perturbed condition, genes involved in the simulated perturbation are ranked at the top of the diffRegulation table (e.g., ng2, ng50, ng11 in the README example).
- For the morphine-vs-mock analysis, the number of genes with FDR < 0.05 is approximately 56.
- GSEA on the ranked list recovers enrichment of opioid signaling, GPCR signaling, reduction of cytosolic calcium levels, and morphine addiction terms.

## Limitations

- scTenifoldNet is more suitable for comparing highly similar samples, such as two populations of cells of the same type, restricting the applicability of downstream enrichment on its ranked output.
- Causal factors and their target genes remain unknown in many biological systems studied, limiting ground-truth validation of the enriched programs.
- Constructing scGRNs from scRNA-seq data consisting of cells in many different states is challenging at present, and regulatory relationships and technical noise are difficult to control.

## Evidence

- [readme] Under H0, no genes reach significance (FDR < 0.1). Under HA, the six genes involved in the perturbation are correctly identified: [evidence span withheld — anchor status: not_found]
