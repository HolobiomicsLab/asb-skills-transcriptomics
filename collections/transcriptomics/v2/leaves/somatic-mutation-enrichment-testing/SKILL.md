---
name: somatic-mutation-enrichment-testing
description: Use when you have a gene set defined by ecDNA amplification (e.g., 251
  genes on circlehunter ecDNA calls) and need to test whether somatic mutations are
  over-represented in that set versus a background.
when_to_use_negative:
- When testing continuous signals such as copy number, expression, or CRISPR gene
  effect rather than binary/over-represented mutation membership — those used separate
  statistical tests (P = 8.754e-16, 2.047e-8, 8.408e-5) instead of over-representation
- When ecDNA calls have not yet been annotated to genes (the 251-gene amplified set
  must exist first; upstream task_005 circlehunter output is a prerequisite)
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_2640
tools:
- name: GSEApy
  role: Python package used to run the over-representation (enrichment) test of somatic
    mutations among ecDNA-amplified genes
  repo: https://github.com/zqfang/GSEApy
- name: circlehunter
  role: Upstream pipeline whose Snakemake ecDNA calls define the amplified gene set
    being tested for mutation enrichment
  repo: https://github.com/suda-huanglab/circlehunter
- name: Snakemake
  role: Workflow engine orchestrating the circlehunter pipeline that produces the
    ecDNA calls feeding this analysis
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41389-023-00476-0
    title: 'Circlehunter: a tool to identify extrachromosomal circular DNA from ATAC-Seq
      data'
  source_content_hash: sha256:3db5d71b0b129f84bed31165ee28d8d909ca14b470e11c8608b810771008a074
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zqfang/GSEApy
  tools:
  - GSEApy
  - Snakemake
  - circlehunter
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# somatic-mutation-enrichment-testing

## Summary

An over-representation analysis that tests whether genes carried on circlehunter-predicted ecDNAs are enriched for somatic mutations, implemented with GSEApy and yielding P = 4.986e-2 for the 251 ecDNA-amplified genes (37 known oncogenes).

## When to use

Use when you have a gene set defined by ecDNA amplification (e.g., 251 genes on circlehunter ecDNA calls) and need to test whether somatic mutations are over-represented in that set versus a background.

## When NOT to use

- When testing continuous signals such as copy number, expression, or CRISPR gene effect rather than binary/over-represented mutation membership — those used separate statistical tests (P = 8.754e-16, 2.047e-8, 8.408e-5) instead of over-representation
- When ecDNA calls have not yet been annotated to genes (the 251-gene amplified set must exist first; upstream task_005 circlehunter output is a prerequisite)

## Inputs

- Gene-level table of ecDNA amplification status and oncogene annotation (figure4_gene_analysis.csv lineage), containing the 251 ecDNA-amplified genes
- Somatic mutation calls for the analyzed public tumor samples / CCLE-DepMap cell lines (gene-level mutation status or counts)
- Gene set definitions (GMT-format gene sets) usable by GSEApy over-representation/enrichment functions

## Outputs

- Enrichment P value for somatic mutation over-representation among ecDNA-carried genes (reported value: P = 4.986e-2)
- Updated figure4_gene_analysis.csv with per-gene ecDNA amplification status, oncogene annotation, and all five P values

## How to apply

Start from the gene-level table produced by the circlehunter Snakemake analysis (upstream tasks annotate the 251 genes overlapped by ecDNA calls and flag the 37 known oncogenes). Collect somatic mutation calls for the same tumor samples/cell lines and convert them to a gene-level mutation frequency or binary mutated/not status, using the ecDNA-amplified gene set as the target set and all annotated genes as the statistical background. Run an over-representation (enrichment) test through GSEApy — e.g., its Enrichr-backed or GSEA-style enrichment modules — to obtain an enrichment P value for somatic mutation co-occurrence with ecDNA amplification; in the source analysis this yielded P = 4.986e-2. Record the resulting P value as one of five gene-set-level statistics (alongside copy number P = 8.754e-16, expression P = 2.047e-8, gene effect P = 8.408e-5, and PCAWG coincidence P = 1.619e-15) in figure4_gene_analysis.csv.

## Related tools

- **GSEApy** (Python package used to run the over-representation (enrichment) test of somatic mutations among ecDNA-amplified genes) — https://github.com/zqfang/GSEApy
- **circlehunter** (Upstream pipeline whose Snakemake ecDNA calls define the amplified gene set being tested for mutation enrichment) — https://github.com/suda-huanglab/circlehunter
- **Snakemake** (Workflow engine orchestrating the circlehunter pipeline that produces the ecDNA calls feeding this analysis)

## Examples

```
import gseapy; enr = gseapy.enrichr(gene_list=ecDNA_genes_251, gene_sets='somatic_mutation_sets.gmt', outdir='mutation_enrichment')
```

## Evaluation signals

- The over-representation test on the ecDNA-amplified gene set should reproduce an enrichment P value on the order of the published P = 4.986e-2
- The input gene set must contain exactly the 251 genes amplified on circlehunter-predicted ecDNAs, of which 37 are annotated known oncogenes
- The somatic-mutation P value must be written alongside the other four statistics (P = 8.754e-16 copy number, P = 2.047e-8 expression, P = 8.408e-5 gene effect, P = 1.619e-15 PCAWG coincidence) in figure4_gene_analysis.csv
- Background gene universe for the enrichment must exclude nothing relevant to the annotation pipeline — sanity-check that mutation status is defined for all 251 tested genes
- GSEApy installation should succeed via bioconda/pip (requires Python 3.8+, Numpy, Scipy, Pandas; Rust toolchain for gseapy > 0.11.0)

## Limitations

- The parent study notes that most analyzed samples were cell lines and cancer-type proportions deviated significantly, which limits generalization of enrichment statistics
- circlehunter cannot guarantee accuracy of ecDNA calls for all types of focal amplifications, so the amplified gene set tested here may contain false positives
- Short read-length sequencing may lead circlehunter to discover computational sub-structures, potentially inflating the amplified gene list

## Evidence

- [results] GSEApy over-representation of somatic mutations: [evidence span withheld — anchor status: not_found]
- [results] gene table output: [evidence span withheld — anchor status: not_found]
- [abstract] ecDNA gene content: "Based on 1312 ecDNAs predicted from 94 publicly available datasets of ATAC-Seq assays, we found 37 oncogenes contained in these ecDNAs"
- [readme] GSEApy scope: [evidence span withheld — anchor status: external_doc]
