---
name: tissue-specificity-classification-across-gtex-tissues
description: Use when re-quantifying eQTL tissue sharing from GTEx v8 cis-eQTL summary
  statistics and the goal is an LD-aware, per-gene tissue-specific vs shared classification
  robust to allelic heterogeneity.
when_to_use_negative:
- Loci whose effects are mediated via coding variants that may not demonstrate eQTL
  effects — these cannot be explored with CAFEH/colocalization-based classification
- Studies where LD differs greatly between tissues/ancestries and only a single LD
  matrix is available, since large LD differences could impact colocalization and
  fine-mapping inference
- Analyses that need to distinguish multiple causal signals among variants in nearly
  perfect or very high LD with each other — this remains difficult even with LD-aware
  methods
edam_operation: http://edamontology.org/operation_3791
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3517
tools:
- name: CAFEH
  role: Analysis environment and framework; CAFEH is the article's Bayesian colocalization/fine-mapping
    method that outperforms existing approaches and motivates the LD-aware reanalysis
    of tissue specificity; install via conda + pip from github.com/karltayeb/cafeh
  repo: https://github.com/karltayeb/cafeh
- name: coloc
  role: Runs approximate Bayes colocalization pairwise across all 49 GTEx v8 tissues
    with priors p1=1e-4, p2=1e-4, p12=1e-6; tissue pairs are called colocalized at
    PPH4 >= 0.5
- name: eCAVIAR
  role: Run on a random subset of 1,000 genes with a maximum of 2 causal variants
    per locus to accommodate allelic heterogeneity; combined with COLOC evidence for
    per-gene classification
- name: Python
  role: Drives the per-locus COLOC/eCAVIAR runs, combines evidence into the classification
    table, and renders the pie chart and histograms reproducing Figure 1C/1D
- name: conda
  role: Creates the cafeh environment with dependencies from environment.yml
- name: pip
  role: Installs the cafeh package inside the cafeh environment (pip install .)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ajhg.2022.01.002
    title: Redefining tissue specificity of genetic regulation of gene expression
      in the presence of allelic heterogeneity
  source_content_hash: sha256:454b260d7caa68d2abc2157b21e67babfc51815214c659376f1e37630b9ac8ed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/karltayeb/cafeh
  tools:
  - CAFEH
  - Python
  - coloc
  - conda
  - eCAVIAR
  - pip
  edam_operation: http://edamontology.org/operation_3791
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# tissue-specificity-classification-across-gtex-tissues

## Summary

Classifies each gene's cis-eQTL effects across GTEx tissues as tissue specific or shared using LD-aware colocalization (COLOC pairwise across 49 GTEx v8 tissues, PPH4 >= 0.5) combined with eCAVIAR fine-mapping on a random subset of 1,000 genes, correcting the inflation of tissue-sharing estimates caused by LD between distinct causal variants.

## When to use

Use when re-quantifying eQTL tissue sharing from GTEx v8 cis-eQTL summary statistics and the goal is an LD-aware, per-gene tissue-specific vs shared classification robust to allelic heterogeneity.

## When NOT to use

- Loci whose effects are mediated via coding variants that may not demonstrate eQTL effects — these cannot be explored with CAFEH/colocalization-based classification
- Studies where LD differs greatly between tissues/ancestries and only a single LD matrix is available, since large LD differences could impact colocalization and fine-mapping inference
- Analyses that need to distinguish multiple causal signals among variants in nearly perfect or very high LD with each other — this remains difficult even with LD-aware methods

## Inputs

- GTEx v8 cis-eQTL summary statistics per tissue (effect sizes and standard errors, or z scores, plus sample sizes)
- Reference LD matrices for each gene locus
- GTEx v8 tissue list (49 tissues)

## Outputs

- Per-gene tissue-specific vs shared classification table saved as coloc_ecaviar_tissue_specificity.csv
- Pie chart of the distribution of tissue-specific vs shared genes (reproducing Figure 1C)
- Histograms of colocalization patterns (reproducing Figure 1D)
- COLOC pairwise PPH4 scores per gene locus and tissue pair
- eCAVIAR CLPP/colocalization results for the 1,000-gene random subset

## How to apply

Set up the CAFEH analysis environment (conda env create --file environment.yml; conda activate cafeh; pip install .) and load GTEx v8 cis-eQTL summary statistics (effect sizes and standard errors, or z scores, plus sample sizes) with reference LD matrices per gene locus. For every gene locus, run COLOC's approximate Bayes colocalization pairwise across all 49 GTEx v8 tissues using priors p1=1e-4, p2=1e-4, p12=1e-6, calling a tissue pair colocalized when PPH4 >= 0.5. Then draw a random subset of 1,000 genes and run eCAVIAR on each, allowing a maximum of 2 causal variants per locus to accommodate allelic heterogeneity. Combine COLOC and eCAVIAR evidence to classify each gene as tissue specific or shared based on the pattern of colocalization across tissue pairs, and save the per-gene classification table as coloc_ecaviar_tissue_specificity.csv. The rationale is that meta-analytic approaches are misled by LD, so LD-aware colocalization reveals that the majority of genes have distinct causal variants in different tissues.

## Related tools

- **CAFEH** (Analysis environment and framework; CAFEH is the article's Bayesian colocalization/fine-mapping method that outperforms existing approaches and motivates the LD-aware reanalysis of tissue specificity; install via conda + pip from github.com/karltayeb/cafeh) — https://github.com/karltayeb/cafeh
- **coloc** (Runs approximate Bayes colocalization pairwise across all 49 GTEx v8 tissues with priors p1=1e-4, p2=1e-4, p12=1e-6; tissue pairs are called colocalized at PPH4 >= 0.5)
- **eCAVIAR** (Run on a random subset of 1,000 genes with a maximum of 2 causal variants per locus to accommodate allelic heterogeneity; combined with COLOC evidence for per-gene classification)
- **Python** (Drives the per-locus COLOC/eCAVIAR runs, combines evidence into the classification table, and renders the pie chart and histograms reproducing Figure 1C/1D)
- **conda** (Creates the cafeh environment with dependencies from environment.yml)
- **pip** (Installs the cafeh package inside the cafeh environment (pip install .))

## Examples

```
conda env create --file environment.yml; conda activate cafeh; pip install .
```

## Evaluation signals

- The reproduced tissue-specific vs shared gene distribution (pie chart and histograms) qualitatively matches the paper's reported Figure 1C/1D
- coloc_ecaviar_tissue_specificity.csv contains a classification entry for every analyzed gene (no missing genes)
- COLOC runs cover all 49 GTEx v8 tissues pairwise per locus with the specified priors (p1=1e-4, p2=1e-4, p12=1e-6) and PPH4 >= 0.5 colocalization threshold
- The eCAVIAR subset is a random draw of exactly 1,000 genes with at most 2 causal variants allowed per locus
- The fraction of genes classified as shared is far lower than reported by meta-analysis approaches, reflecting LD-aware inference that most genes have distinct causal variants across tissues

## Limitations

- CAFEH relies on a single LD matrix, so large differences in LD between studies, due to ancestry differences or other factors, could impact colocalization and fine-mapping inference
- It is still difficult to distinguish the presence of multiple signals among variants in nearly perfect or very high LD with each other
- Colocalization does not necessarily suggest mediation; colocalizing signals can arise without a linking causal pathway due to horizontal pleiotropic effects
- CAFEH does not handle missing data in the effect size matrix, implicitly assuming causal variants are present in the tested datasets
- Loci whose effects are mediated via coding variants may not demonstrate eQTL effects and therefore cannot be explored with CAFEH

## Evidence

- [results] Using LD-aware colocalization (COLOC) across 49 GTEx v8 tissues, the majority of genes have distinct causal variants responsible for their cis-eQTL signals in different tissues: [evidence span withheld — anchor status: not_found]
- [abstract] linkage disequilibrium misleads meta-analysis: "We find evidence of pervasive tissue specificity of eQTLs, often masked by linkage disequilibrium that misleads traditional meta-analytic approaches."
- [readme] install instructions: "git clone https://github.com/karltayeb/cafeh.git
cd cafeh
conda env create --file environment.yml  # create envrionment with dependencies
conda activate cafeh  # activate environment
pip install .  #"
- [discussion] LD differences limitation: "the current version of CAFEH relies on a single LD matrix, therefore large differences in LD between studies, due to ancestry differences or other factors, could impact inference"
- [abstract] CAFEH outperforms: "CAFEH outperforms previous approaches in colocalization and fine-mapping."
