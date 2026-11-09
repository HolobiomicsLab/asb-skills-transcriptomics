---
name: publication-figure-reproduction
description: Use when the goal is to validate or replicate a published quantitative
  figure by re-running its described pipeline on the original or equivalent data,
  and a qualitative match can be checked against the reported figure.
when_to_use_negative:
- When the goal is to develop or benchmark a new colocalization method rather than
  validate a published figure — CAFEH/COLOC/eCAVIAR outputs are then primary results,
  not reproduction artifacts
- When the publication does not document a re-executable workflow or release the underlying
  summary statistics/LD inputs, making a faithful re-run impossible
- When the target locus data differ substantially from the original (e.g., different
  ancestry/LD panel), since CAFEH-style inference relies on a single LD matrix and
  large LD differences between studies could impact colocalization
tools:
- name: CAFEH
  role: Analysis environment and colocalization/fine-mapping framework the paper's
    figures were generated with; installed via conda and pip before re-running the
    figure workflow
  repo: github.com/karltayeb/cafeh
- name: coloc
  role: Approximate Bayes colocalization run pairwise across 49 GTEx v8 tissues with
    priors p1=1e-4, p2=1e-4, p12=1e-6 and PPH4 >= 0.5 calling threshold
- name: eCAVIAR
  role: LD-aware colocalization run on a random subset of 1,000 genes with up to 2
    causal variants per locus, combined with COLOC evidence for classification
- name: conda
  role: Creates the pinned dependency environment from environment.yml to match the
    original analysis setup
- name: pip
  role: Installs the CAFEH package into the activated cafeh environment
- name: Python
  role: Used to combine COLOC/eCAVIAR outputs, write the classification CSV, and render
    the pie chart and histograms
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ajhg.2022.01.002
    title: Redefining tissue specificity of genetic regulation of gene expression
      in the presence of allelic heterogeneity
  source_content_hash: sha256:d88ffa37528b334b57e184ec23ba55cd4e804a349567c35e0e3ba166444cc656
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CAFEH
  - Python
  - coloc
  - conda
  - eCAVIAR
  - pip
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# publication-figure-reproduction

## Summary

Reproduces a published figure (here, Figure 1C/1D of the GTEx v8 tissue-sharing analysis) by re-executing the paper's analysis pipeline — LD-aware pairwise colocalization with COLOC and eCAVIAR across 49 GTEx v8 tissues — and verifying the output distributions against the published panels.

## When to use

Use when the goal is to validate or replicate a published quantitative figure by re-running its described pipeline on the original or equivalent data, and a qualitative match can be checked against the reported figure.

## When NOT to use

- When the goal is to develop or benchmark a new colocalization method rather than validate a published figure — CAFEH/COLOC/eCAVIAR outputs are then primary results, not reproduction artifacts
- When the publication does not document a re-executable workflow or release the underlying summary statistics/LD inputs, making a faithful re-run impossible
- When the target locus data differ substantially from the original (e.g., different ancestry/LD panel), since CAFEH-style inference relies on a single LD matrix and large LD differences between studies could impact colocalization

## Inputs

- GTEx v8 cis-eQTL summary statistics (effect sizes, standard errors or z scores, sample sizes) per gene locus
- Reference LD matrices for each analyzed gene locus
- Original paper's target figure (Figure 1C/1D) as a qualitative reference

## Outputs

- coloc_ecaviar_tissue_specificity.csv — per-gene classification table (tissue-specific vs shared) for every analyzed gene
- Pie chart of tissue-specific vs shared gene distribution
- Histograms of tissue-sharing estimates reproducing Figure 1C/1D

## How to apply

Reconstruct the environment exactly as documented (conda env create --file environment.yml, conda activate cafeh, pip install .) so tool versions match the original run. Load GTEx v8 cis-eQTL summary statistics (effect sizes, standard errors or z scores, sample sizes) plus reference LD matrices per gene locus, then run COLOC's approximate Bayes colocalization pairwise across all 49 GTEx v8 tissues with priors p1=1e-4, p2=1e-4, p12=1e-6, calling a tissue pair colocalized at PPH4 >= 0.5. Draw a random subset of 1,000 genes and run eCAVIAR on each allowing a maximum of 2 causal variants per locus to accommodate allelic heterogeneity. Combine COLOC and eCAVIAR evidence to classify each gene as tissue-specific or shared, save the per-gene table as coloc_ecaviar_tissue_specificity.csv, and render the pie chart and histograms of the tissue-specific vs shared gene distribution to compare against the paper's Figure 1C/1D.

## Related tools

- **CAFEH** (Analysis environment and colocalization/fine-mapping framework the paper's figures were generated with; installed via conda and pip before re-running the figure workflow) — github.com/karltayeb/cafeh
- **coloc** (Approximate Bayes colocalization run pairwise across 49 GTEx v8 tissues with priors p1=1e-4, p2=1e-4, p12=1e-6 and PPH4 >= 0.5 calling threshold)
- **eCAVIAR** (LD-aware colocalization run on a random subset of 1,000 genes with up to 2 causal variants per locus, combined with COLOC evidence for classification)
- **conda** (Creates the pinned dependency environment from environment.yml to match the original analysis setup)
- **pip** (Installs the CAFEH package into the activated cafeh environment)
- **Python** (Used to combine COLOC/eCAVIAR outputs, write the classification CSV, and render the pie chart and histograms)

## Examples

```
python cafeh.py --mode z -R example/LD_example.tsv -z example/z_example.tsv -n example/n_example.tsv --out output --save-model
```

## Evaluation signals

- The reproduced tissue-specific vs shared gene distribution (pie chart and histograms) qualitatively matches the paper's reported Figure 1C/1D
- The results table contains a classification entry for every analyzed gene (no missing per-gene rows in coloc_ecaviar_tissue_specificity.csv)
- All COLOC pairwise runs used the documented priors (p1=1e-4, p2=1e-4, p12=1e-6) and the PPH4 >= 0.5 colocalization call threshold
- eCAVIAR was restricted to a random subset of 1,000 genes with a maximum of 2 causal variants per locus, matching the paper's design
- Environment setup completed without errors (conda env create --file environment.yml followed by pip install .) so library versions are traceable

## Limitations

- A qualitative match to Figure 1C/1D is the acceptance criterion; exact numeric agreement is not guaranteed when the random 1,000-gene subset or GTEx data versions differ
- LD within each locus and multiple causal alleles can mislead meta-analytic sharing estimates, so reproduction must use LD-aware methods to match the paper's conclusion of pervasive tissue specificity
- CAFEH relies on a single LD matrix, so large LD differences between studies (e.g., ancestry differences) could impact colocalization inference in the reproduction
- CAFEH does not handle missing data in the effect size matrix, implicitly assuming causal variants are present in the tested datasets
- Colocalization does not necessarily suggest mediation; colocalizing signals can arise from horizontal pleiotropic effects

## Evidence

- [abstract] Pervasive tissue specificity finding motivating the figure: "We find evidence of pervasive tissue specificity of eQTLs, often masked by linkage disequilibrium that misleads traditional meta-analytic approaches."
- [methods] Environment setup commands from the paper's methods: "conda env create --file environment.yml  # create envrionment with dependencies"
- [readme] CAFEH can be run as a CLI tool or Python package: "You can use CAFEH as a command line tool or a python package. Read below for a minimum example of how to use CAFEH from the command line."
- [methods] CAFEH z-score summary-stat mode invocation requirements: "To fit CAFEH with zscores, use `--mode z` in the arguments. You must provide input `--ld (-R)`, and `--zscores (-z)`."
- [discussion] CAFEH automatically prunes extra causal components: "The algorithm then automatically removes any additional components beyond the number needed to explain the tested association signals."
