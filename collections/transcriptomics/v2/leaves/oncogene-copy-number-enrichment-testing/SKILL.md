---
name: oncogene-copy-number-enrichment-testing
description: Use when you have a set of ecDNA or focal-amplification gene calls and
  need to test whether known oncogenes are significantly enriched for amplification,
  expression, or functional effects.
when_to_use_negative:
- Input is raw sequencing data and ecDNA calls have not yet been produced — run the
  circlehunter Snakemake pipeline first
- The gene set comes from whole-genome sequencing ecDNA callers (e.g. AmpliconArchitect,
  JaBbA, LINX) requiring WGS copy-number input rather than ATAC-Seq-based circlehunter
  calls
- No known-oncogene annotation or external CCLE/DepMap/PCAWG tables are available,
  so enrichment tests cannot be grounded
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0622
tools:
- name: circlehunter
  role: Produces the ecDNA calls whose overlapped genes form the amplified-gene set
    tested for oncogene enrichment
  repo: https://github.com/suda-huanglab/circlehunter
- name: Snakemake
  role: Orchestrates the circlehunter pipeline that generates upstream ecDNA calls
    and the analysis workflow
  repo: https://snakemake.readthedocs.io/
- name: GSEApy
  role: Performs over-representation testing of somatic mutation enrichment among
    ecDNA-carried genes
  repo: https://github.com/zqfang/GSEApy
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41389-023-00476-0
    title: 'Circlehunter: a tool to identify extrachromosomal circular DNA from ATAC-Seq
      data'
  source_content_hash: sha256:f4319bb93f0ef2f8e9981f22cc4f5e6b097277f1a925095c72d82d808c3dd563
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/suda-huanglab/circlehunter
  tools:
  - GSEApy
  - Snakemake
  - circlehunter
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# oncogene-copy-number-enrichment-testing

## Summary

Quantify and test enrichment of oncogenes among ecDNA-amplified genes identified by circlehunter, joining amplified-gene sets with CCLE/DepMap copy-number, expression, and CRISPR gene-effect tables plus PCAWG copy-number gains to obtain copy-number, expression, gene-effect, coincidence, and mutation-enrichment P values.

## When to use

Use when you have a set of ecDNA or focal-amplification gene calls and need to test whether known oncogenes are significantly enriched for amplification, expression, or functional effects.

## When NOT to use

- Input is raw sequencing data and ecDNA calls have not yet been produced — run the circlehunter Snakemake pipeline first
- The gene set comes from whole-genome sequencing ecDNA callers (e.g. AmpliconArchitect, JaBbA, LINX) requiring WGS copy-number input rather than ATAC-Seq-based circlehunter calls
- No known-oncogene annotation or external CCLE/DepMap/PCAWG tables are available, so enrichment tests cannot be grounded

## Inputs

- circlehunter ecDNA calls (upstream Snakemake pipeline output for all analyzed ATAC-Seq samples)
- Gene annotation to intersect ecDNA coordinates with overlapped genes
- CCLE/DepMap copy-number table
- CCLE/DepMap expression table
- CCLE/DepMap CRISPR gene-effect table
- PCAWG copy-number-gain calls
- Known oncogene list/annotation
- Somatic mutation calls for over-representation testing

## Outputs

- Per-gene table (figure4_gene_analysis.csv) with ecDNA amplification status, oncogene annotation, and five P values
- Enrichment statistics: oncogene copy number (P = 8.754e-16), expression (P = 2.047e-8), gene effect (P = 8.408e-5), PCAWG coincidence (P = 1.619e-15), somatic mutation over-representation (P = 4.986e-2)
- Counts of total amplified genes (251) and amplified known oncogenes (37)

## How to apply

Load ecDNA calls from the circlehunter Snakemake pipeline and annotate genes overlapped by each ecDNA call to build the amplified-gene set (e.g. 251 genes averaging 10 genes per ecDNA across 94 public ATAC-Seq tumor datasets). Classify which amplified genes are known oncogenes (e.g. 37 oncogenes in the circlehunter cohort). Join the amplified-gene set with CCLE/DepMap copy-number, expression, and CRISPR gene-effect tables and compute enrichment P values for oncogene copy number (P = 8.754e-16), expression (P = 2.047e-8), and gene effect (P = 8.408e-5). Intersect ecDNA-amplified genes with PCAWG copy-number-gain calls and test coincidence (P = 1.619e-15), then run an over-representation test of somatic mutation enrichment using GSEApy (P = 4.986e-2). Write the gene-level table with per-gene ecDNA amplification status, oncogene annotation, and all five P values to figure4_gene_analysis.csv.

## Related tools

- **circlehunter** (Produces the ecDNA calls whose overlapped genes form the amplified-gene set tested for oncogene enrichment) — https://github.com/suda-huanglab/circlehunter
- **Snakemake** (Orchestrates the circlehunter pipeline that generates upstream ecDNA calls and the analysis workflow) — https://snakemake.readthedocs.io/
- **GSEApy** (Performs over-representation testing of somatic mutation enrichment among ecDNA-carried genes) — https://github.com/zqfang/GSEApy

## Examples

```
snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml
```

## Evaluation signals

- Total amplified-gene count matches expectation (251 genes across 1312 ecDNAs, average 10 genes per ecDNA)
- Known-oncogene subset count matches 37 of the amplified genes
- Replication of the five reported P values: 8.754e-16 (copy number), 2.047e-8 (expression), 8.408e-5 (gene effect), 1.619e-15 (PCAWG coincidence), 4.986e-2 (mutation enrichment)
- Output file figure4_gene_analysis.csv contains per-gene ecDNA amplification status, oncogene flag, and all five P-value columns
- All reported P values fall below the conventional significance threshold of P < 0.05

## Limitations

- Most analyzed samples were cell lines and cancer type proportions deviated significantly from each other; additional data are needed
- Current benchmarking strategy has limitations, so accuracy of ecDNA calls for all types of focal amplifications cannot be guaranteed — false-positive ecDNA calls would propagate into the gene set
- GSEApy over-representation P value (4.986e-2) is marginal and near the P < 0.05 boundary, warranting caution

## Evidence

- [abstract] ecDNA calls contain 251 genes with 37 oncogenes: "Based on 1312 ecDNAs predicted from 94 publicly available datasets of ATAC-Seq assays, we found 37 oncogenes contained in these ecDNAs"
- [intro] ecDNA carries oncogenes like MYC, MYCN, EGFR and drives amplification: "an increasing number of ecDNAs have been found to carry important oncogenes in different types of cancer [9], including MYC, MYCN, and EGFR"
- [methods] Circlehunter pipeline is run via Snakemake with a config file: "snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml"
- [readme] GSEApy is used for enrichment analysis and can run in snakemake workflows: [evidence span withheld — anchor status: external_doc]
