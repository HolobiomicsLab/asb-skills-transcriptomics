---
name: myc-copy-number-amplification-analysis
description: Use when analyzing ATAC-Seq samples from SCLC (or similar cancer) cell
  lines to determine whether MYC resides on ecDNA and whether that ecMYC status elevates
  MYC copy number and expression.
when_to_use_negative:
- Input data is whole-genome sequencing rather than ATAC-Seq; tools like AmpliconArchitect,
  which use copy number variation from WGS, are the appropriate alternative
- Focal amplifications other than ecDNA (e.g. complex structural variants) are the
  target — circlehunter's benchmarking cannot guarantee accuracy for all focal amplification
  classes
- Single-cell ATAC-Seq data — application of circlehunter to single-cell data has
  yet to be explored
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0203
tools:
- name: circlehunter
  role: Pipeline that identifies ecDNA (including ecMYC/ecMYCN) from ATAC-Seq data
    by exploiting the enhanced chromatin accessibility of ecDNA; driven by a yaml
    config and executed with Snakemake
  repo: https://github.com/suda-huanglab/circlehunter
- name: Snakemake
  role: Workflow engine that orchestrates the circlehunter pipeline rules with parallel
    jobs (-j 16) and a config.yaml file
- name: GSEApy
  role: Python gene set enrichment toolkit referenced in the article's repository
    context for downstream expression/enrichment analysis supporting the SCLC-N (NEUROD1-high)
    expression pattern
  repo: https://github.com/zqfang/GSEApy
- name: AmpliconArchitect
  role: WGS-based ecDNA identification framework cited as an alternative approach
    requiring copy number variation input, contrasted with the ATAC-Seq-based circlehunter
    route
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41389-023-00476-0
    title: 'Circlehunter: a tool to identify extrachromosomal circular DNA from ATAC-Seq
      data'
  source_content_hash: sha256:5a61ab00cc79292d34052b4a5196c5a9f68b0776dc0baebbdc74fc6202336166
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/suda-huanglab/circlehunter
  tools:
  - AmpliconArchitect
  - GSEApy
  - Snakemake
  - circlehunter
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# myc-copy-number-amplification-analysis

## Summary

Reproduce circlehunter-based detection of ecDNA carrying MYC (ecMYC) in SCLC cell-line ATAC-Seq data and verify that ecMYC status drives elevated MYC copy number (P = 1.425e-4, ANOVA) and expression (P = 2.327e-3) with a NEUROD1-high (SCLC-N) expression pattern.

## When to use

Use when analyzing ATAC-Seq samples from SCLC (or similar cancer) cell lines to determine whether MYC resides on ecDNA and whether that ecMYC status elevates MYC copy number and expression.

## When NOT to use

- Input data is whole-genome sequencing rather than ATAC-Seq; tools like AmpliconArchitect, which use copy number variation from WGS, are the appropriate alternative
- Focal amplifications other than ecDNA (e.g. complex structural variants) are the target — circlehunter's benchmarking cannot guarantee accuracy for all focal amplification classes
- Single-cell ATAC-Seq data — application of circlehunter to single-cell data has yet to be explored

## Inputs

- ATAC-Seq samples for 19 SCLC cell lines (publicly available, e.g. from SRA)
- config.yaml declaring all samples and desired outputs (yaml format, from config-template.yaml)
- circlehunter repository Snakefile and environment.yml

## Outputs

- Per-sample circlehunter ecDNA calls (ecMYC in NCI-69/NCI-H82/NCI-H2171/NCI-H524; ecMYCN where reported)
- MYC copy-number comparison across ecMYC-positive vs. other SCLC cell lines (ANOVA, P = 1.425e-4)
- MYC expression comparison (P = 2.327e-3) and NEUROD1-high (SCLC-N) expression pattern corresponding to Figs. 5-6

## How to apply

Install the circlehunter pipeline dependencies into a dedicated conda environment via mamba with the repository's environment.yml, then create a workspace and copy config-template.yaml to config.yaml. Edit config.yaml to declare all 19 SCLC cell-line ATAC-Seq samples and their desired outputs, since circlehunter uses a yaml-format config file to resolve samples and outputs. Run the Snakemake pipeline with up to 16 parallel jobs ('snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml') and inspect per-sample circlehunter ecDNA calls, expecting ecMYC in NCI-H69/NCI-H82/NCI-H2171/NCI-H524 and ecMYCN where reported. Judge success by recovering these ecDNA calls and reproducing the significant MYC copy-number elevation (P = 1.425e-4, ANOVA) and expression elevation (P = 2.327e-3), plus the NEUROD1-high (SCLC-N) pattern matching Figs. 5-6.

## Related tools

- **circlehunter** (Pipeline that identifies ecDNA (including ecMYC/ecMYCN) from ATAC-Seq data by exploiting the enhanced chromatin accessibility of ecDNA; driven by a yaml config and executed with Snakemake) — https://github.com/suda-huanglab/circlehunter
- **Snakemake** (Workflow engine that orchestrates the circlehunter pipeline rules with parallel jobs (-j 16) and a config.yaml file)
- **GSEApy** (Python gene set enrichment toolkit referenced in the article's repository context for downstream expression/enrichment analysis supporting the SCLC-N (NEUROD1-high) expression pattern) — https://github.com/zqfang/GSEApy
- **AmpliconArchitect** (WGS-based ecDNA identification framework cited as an alternative approach requiring copy number variation input, contrasted with the ATAC-Seq-based circlehunter route)

## Examples

```
mamba env create -f environment.yml; conda activate circlehunter; snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml
```

## Evaluation signals

- circlehunter recovers the reported ecMYC calls in the named SCLC cell lines (NCI-69/NCI-H82/NCI-H2171/NCI-H524) and ecMYCN where reported
- MYC copy number in ecMYC-positive cell lines is significantly higher than the rest (ANOVA P = 1.425e-4)
- MYC expression is significantly elevated in ecMYC-positive cell lines (P = 2.327e-3)
- Expression pattern corresponds to the NEUROD1-high (SCLC-N) subtype shown in Figs. 5-6
- Pipeline completes under the declared environment.yml environment with the yaml config resolving all 19 samples and their outputs

## Limitations

- The current benchmarking strategy has limitations, and accuracy of ecDNA calls for all types of focal amplifications cannot be guaranteed
- Other classes of focal amplification with misregulated chromatin may be detected as false-positive ecDNA
- Short read-length sequencing may lead circlehunter to discover more computational sub-structures (e.g. COLO320DM)
- Application of circlehunter to single-cell ATAC-Seq data has not yet been explored
- Most analyzed samples were cell lines and cancer-type proportions deviated significantly, so additional data are needed

## Evidence

- [abstract] ecDNA containing MYC leads to amplification of MYC and cis-regulates the expression of NEUROD1: "ecDNA containing MYC leads to amplification of MYC and cis-regulates the expression of NEUROD1, resulting in an expression pattern consistent with the NEUROD1 high expression subtype"
- [methods] circlehunter use a yaml format config file to figure out all samples and their output: "circlehunter use a `yaml` format config file to figure out all samples and their output"
- [methods] snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml: "snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml"
- [readme] All dependencies can be installed using `mamba`: "All dependencies can be installed using `mamba`"
- [discussion] we cannot guarantee the accuracy of ecDNA calls for all types of focal amplifications: "it is important to note that the current benchmarking strategy has limitations, and we cannot guarantee the accuracy of ecDNA calls for all types of focal amplifications"
