---
name: sclc-neurod1-expression-subtyping
description: Use when SCLC ATAC-Seq/RNA data are available and you need to confirm
  ecMYC-driven MYC amplification and check whether expression patterns match the NEUROD1-high
  (SCLC-N) subtype.
when_to_use_negative:
- Samples are not SCLC cell lines or lack matched ATAC-Seq and expression data needed
  to link ecMYC status to NEUROD1-high subtyping.
- The goal is generic ecDNA detection in non-SCLC samples without the MYC/NEUROD1
  expression subtype question.
- Only WGS data (no ATAC-Seq) are available; circlehunter relies on enhanced chromatin
  accessibility of ecDNA.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: circlehunter
  role: Snakemake pipeline that identifies ecDNA (including ecMYC/ecMYCN) from SCLC
    ATAC-Seq data by exploiting the enhanced chromatin accessibility of ecDNA.
  repo: https://github.com/suda-huanglab/circlehunter
- name: Snakemake
  role: Workflow engine used to execute the circlehunter pipeline with up to 16 parallel
    processes via a yaml config file.
  repo: https://snakemake.readthedocs.io/
- name: GSEApy
  role: Python/Rust gene set enrichment tool used for expression analysis of the NEUROD1-high
    (SCLC-N) signature.
  repo: https://github.com/zqfang/GSEApy
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41389-023-00476-0
    title: 'Circlehunter: a tool to identify extrachromosomal circular DNA from ATAC-Seq
      data'
  source_content_hash: sha256:ce6a4d97afe26c187ae70d0b0a6519ba46683ce6dbb0cd9776c75f9a82a6cc85
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
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# sclc-neurod1-expression-subtyping

## Summary

Validates that ecDNA containing MYC (ecMYC) in SCLC cell lines drives elevated MYC copy number and expression with a NEUROD1-high (SCLC-N) pattern, linking ecMYC status to an Aurora kinase inhibitor-sensitive subtype.

## When to use

Use when SCLC ATAC-Seq/RNA data are available and you need to confirm ecMYC-driven MYC amplification and check whether expression patterns match the NEUROD1-high (SCLC-N) subtype.

## When NOT to use

- Samples are not SCLC cell lines or lack matched ATAC-Seq and expression data needed to link ecMYC status to NEUROD1-high subtyping.
- The goal is generic ecDNA detection in non-SCLC samples without the MYC/NEUROD1 expression subtype question.
- Only WGS data (no ATAC-Seq) are available; circlehunter relies on enhanced chromatin accessibility of ecDNA.

## Inputs

- ATAC-Seq FASTQ/BAM data for 19 SCLC cell-line samples
- circlehunter yaml config file (config.yaml) listing samples and desired outputs
- MYC copy-number and MYC/NEUROD1 expression measurements per cell line

## Outputs

- Per-sample circlehunter ecDNA calls (ecMYC in NCI-H69/NCI-H82/NCI-H2171/NCI-H524, ecMYCN where reported)
- MYC copy-number ANOVA result (P = 1.425e-4)
- MYC expression elevation test result (P = 2.327e-3)
- NEUROD1-high (SCLC-N) expression pattern consistent with Figs. 5-6

## How to apply

Run the circlehunter Snakemake pipeline on the 19 SCLC cell-line ATAC-Seq samples declared in config.yaml (snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml) and inspect per-sample ecDNA calls to confirm ecMYC in NCI-H69, NCI-H82, NCI-H2171 and NCI-H524 (and ecMYCN where reported). Then test MYC copy-number elevation across the ecMYC-positive vs. other SCLC cell lines by ANOVA, expecting P = 1.425e-4, and test MYC expression elevation, expecting P = 2.327e-3. Finally, examine the expression pattern of MYC and NEUROD1 to confirm the NEUROD1-high (SCLC-N) signature corresponding to Figs. 5-6, consistent with an Aurora kinase inhibitor-sensitive subtype; GSEApy is available for enrichment analysis of the expression signatures.

## Related tools

- **circlehunter** (Snakemake pipeline that identifies ecDNA (including ecMYC/ecMYCN) from SCLC ATAC-Seq data by exploiting the enhanced chromatin accessibility of ecDNA.) — https://github.com/suda-huanglab/circlehunter
- **Snakemake** (Workflow engine used to execute the circlehunter pipeline with up to 16 parallel processes via a yaml config file.) — https://snakemake.readthedocs.io/
- **GSEApy** (Python/Rust gene set enrichment tool used for expression analysis of the NEUROD1-high (SCLC-N) signature.) — https://github.com/zqfang/GSEApy

## Examples

```
snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml
```

## Evaluation signals

- circlehunter recovers the reported ecMYC calls in NCI-H69, NCI-H82, NCI-H2171 and NCI-H524, and ecMYCN where reported.
- MYC copy number in ecMYC-positive SCLC cell lines is significantly higher than in the rest (ANOVA P = 1.425e-4).
- MYC expression elevation is reproduced with P = 2.327e-3.
- Expression pattern of ecMYC-positive lines matches the NEUROD1-high (SCLC-N) subtype shown in Figs. 5-6.
- All 19 SCLC cell-line ATAC-Seq samples are declared in config.yaml and the pipeline completes without rule failures.

## Limitations

- The current benchmarking strategy has limitations and accuracy of ecDNA calls for all types of focal amplifications cannot be guaranteed.
- Short read-length sequencing may lead circlehunter to discover more computational sub-structures (e.g. COLO320DM).
- Most analyzed samples were cell lines and cancer-type proportions deviated significantly; additional data are needed.
- Other focal amplification classes with misregulated chromatin may be detected as false-positive ecDNA.
- Application of circlehunter to single-cell ATAC-Seq data has yet to be explored.

## Evidence

- [abstract] ecDNA containing MYC leads to amplification of MYC and cis-regulates the expression of NEUROD1: "ecDNA containing MYC leads to amplification of MYC and cis-regulates the expression of NEUROD1, resulting in an expression pattern consistent with the NEUROD1 high expression subtype and sensitive to"
- [methods] circlehunter use a yaml format config file to figure out all samples and their output: "circlehunter use a `yaml` format config file to figure out all samples and their output"
- [methods] snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml: "snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml"
- [discussion] we cannot guarantee the accuracy of ecDNA calls for all types of focal amplifications: "it is important to note that the current benchmarking strategy has limitations, and we cannot guarantee the accuracy of ecDNA calls for all types of focal amplifications"
- [discussion] short read-length sequencing has a natural disadvantage for detecting structural variants: "short read-length sequencing has a natural disadvantage for detecting structural variants, which may lead circlehunter to discover more computational sub-structures"
