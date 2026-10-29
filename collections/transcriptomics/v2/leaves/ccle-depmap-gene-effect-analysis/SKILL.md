---
name: ccle-depmap-gene-effect-analysis
description: Use when you have a set of ecDNA calls (e.g., from the circlehunter Snakemake
  pipeline) and need to quantify oncogene content and test enrichment against CCLE/DepMap
  functional genomics data.
when_to_use_negative:
- Your samples are primary tumors or cancer types not represented in CCLE/DepMap —
  the article notes most analyzed samples were cell lines and cancer-type proportions
  deviated significantly, so functional joins may lack coverage
- Your goal is de novo ecDNA detection from raw ATAC-Seq data rather than functional
  characterization of existing circlehunter ecDNA calls — run the circlehunter Snakemake
  pipeline first
- You have no ecDNA-derived gene set to test, so enrichment P values and the per-gene
  ecDNA amplification status column cannot be computed
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0203
tools:
- name: circlehunter
  role: Upstream producer of the ecDNA calls whose overlapped genes are annotated
    and joined to CCLE/DepMap tables
  repo: https://github.com/suda-huanglab/circlehunter
- name: GSEApy
  role: Over-representation enrichment testing of somatic mutations among ecDNA-carried
    genes (step 5 of the workflow)
  repo: https://github.com/zqfang/GSEApy
- name: Snakemake
  role: Workflow engine that runs the circlehunter pipeline producing the ecDNA calls
    consumed by this analysis
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41389-023-00476-0
    title: 'Circlehunter: a tool to identify extrachromosomal circular DNA from ATAC-Seq
      data'
  source_content_hash: sha256:0e3a6651f34bcb8d08a0aa596e02e092bd923684dafc2db7649a89ad0b7ed94c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/suda-huanglab/circlehunter
  tools:
  - GSEApy
  - Snakemake
  - circlehunter
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# ccle-depmap-gene-effect-analysis

## Summary

Annotates genes carried on circlehunter-predicted ecDNAs and joins them with CCLE/DepMap copy-number, expression, and CRISPR gene-effect tables to test whether ecDNA-carried genes — especially the 37 known oncogenes among 251 amplified genes — are significantly enriched for amplification, expression, and fitness effects.

## When to use

Use when you have a set of ecDNA calls (e.g., from the circlehunter Snakemake pipeline) and need to quantify oncogene content and test enrichment against CCLE/DepMap functional genomics data.

## When NOT to use

- Your samples are primary tumors or cancer types not represented in CCLE/DepMap — the article notes most analyzed samples were cell lines and cancer-type proportions deviated significantly, so functional joins may lack coverage
- Your goal is de novo ecDNA detection from raw ATAC-Seq data rather than functional characterization of existing circlehunter ecDNA calls — run the circlehunter Snakemake pipeline first
- You have no ecDNA-derived gene set to test, so enrichment P values and the per-gene ecDNA amplification status column cannot be computed

## Inputs

- ecDNA calls from the circlehunter Snakemake pipeline for all analyzed samples (task_005 output)
- Gene annotations for genomic intervals overlapped by each ecDNA call
- Known oncogene reference list used to classify the 37-oncogene subset
- CCLE/DepMap copy-number table
- CCLE/DepMap expression table
- DepMap CRISPR gene-effect table
- PCAWG copy-number-gain calls
- Somatic mutation sets for over-representation testing

## Outputs

- figure4_gene_analysis.csv with per-gene ecDNA amplification status, oncogene annotation, and five enrichment P values (copy number 8.754e-16, expression 2.047e-8, gene effect 8.408e-5, PCAWG gain coincidence 1.619e-15, somatic mutation enrichment 4.986e-2)
- Annotated amplified-gene set (251 genes, 37 oncogenes, ~10 genes per ecDNA)

## How to apply

Load the ecDNA calls produced by the circlehunter Snakemake pipeline (upstream task_005 output) and annotate the genes overlapped by each call — the 1312 ecDNAs from 94 public ATAC-Seq datasets carry on average 10 genes each, totaling 251 amplified genes, of which 37 are known oncogenes. Join this amplified-gene set with CCLE/DepMap copy-number, expression, and CRISPR gene-effect tables on gene identifiers and compute enrichment significance for oncogene copy number (P = 8.754e-16), expression (P = 2.047e-8), and gene effect (P = 8.408e-5). Intersect ecDNA-amplified genes with PCAWG copy-number-gain calls and test their coincidence (P = 1.619e-15). Test somatic mutation enrichment among ecDNA-carried genes using a GSEApy-based over-representation approach (P = 4.986e-2). Write the per-gene table — ecDNA amplification status, oncogene annotation, and all five P values — to figure4_gene_analysis.csv.

## Related tools

- **circlehunter** (Upstream producer of the ecDNA calls whose overlapped genes are annotated and joined to CCLE/DepMap tables) — https://github.com/suda-huanglab/circlehunter
- **GSEApy** (Over-representation enrichment testing of somatic mutations among ecDNA-carried genes (step 5 of the workflow)) — https://github.com/zqfang/GSEApy
- **Snakemake** (Workflow engine that runs the circlehunter pipeline producing the ecDNA calls consumed by this analysis)

## Examples

```
gseapy enrichr -i ecDNA_amplified_genes.txt -g KEGG_2016 -o test
```

## Evaluation signals

- The annotated gene table totals 251 amplified genes with 37 classified as known oncogenes, averaging ~10 genes per ecDNA call
- The five enrichment P values reproduce: copy number P = 8.754e-16, expression P = 2.047e-8, gene effect P = 8.408e-5, PCAWG gain coincidence P = 1.619e-15, mutation enrichment P = 4.986e-2
- figure4_gene_analysis.csv exists and contains per-gene columns for ecDNA amplification status, oncogene annotation, and all five P values
- Every ecDNA-amplified gene from the circlehunter calls is present in the joined CCLE/DepMap copy-number, expression, and CRISPR gene-effect tables (or explicitly flagged as missing)
- The ecDNA-amplified gene set intersects non-trivially with PCAWG copy-number-gain calls (coincidence test P = 1.619e-15 indicates strong overlap)

## Limitations

- Most analyzed samples were cell lines and cancer-type proportions deviated significantly from each other; additional data are needed, limiting generalization of the DepMap-based functional conclusions
- The upstream circlehunter benchmarking strategy has limitations and accuracy of ecDNA calls for all types of focal amplifications cannot be guaranteed, so genes annotated on false-positive ecDNAs could contaminate the amplified-gene set
- Other classes of focal amplification with misregulated chromatin may be detected as false-positive ecDNA by circlehunter, inflating apparent oncogene amplification

## Evidence

- [abstract] 1312 ecDNAs predicted from 94 datasets with 37 oncogenes: "Based on 1312 ecDNAs predicted from 94 publicly available datasets of ATAC-Seq assays, we found 37 oncogenes contained in these ecDNAs"
- [intro] ecDNA drives oncogene expression to top 1% of transcriptome: "increased the expression level of oncogenes on ecDNA to the top 1% in the whole transcriptome and contributed to the pathogenesis of tumors"
- [discussion] Most samples were cell lines, limiting DepMap-based generalization: "most of the samples in the dataset we analyzed were cell lines, and the proportion of individual cancer types deviated significantly from each other. Additional data are needed"
- [readme] GSEApy Enrichr module for over-representation analysis: [evidence span withheld — anchor status: external_doc]
