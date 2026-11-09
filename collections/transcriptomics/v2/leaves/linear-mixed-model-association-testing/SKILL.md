---
name: linear-mixed-model-association-testing
description: Use when you have CAFEH per-(SNP, tissue) results (pip, p_active, top_component,
  rank, pi) across many tissues and traits, and need to quantify whether tissue-restricted
  eQTL components are enriched for activity in GWAS loci.
when_to_use_negative:
- 'Loci with large LD differences between studies (e.g., ancestry differences): CAFEH
  relies on a single LD matrix, so cross-study inference may be biased before any
  mixed-model aggregation is attempted.'
- 'Signals mediated via coding variants that lack eQTL effects: these cannot be explored
  with CAFEH and thus contribute no p_active values to the tissue-sharing analysis.'
- 'Colocalization-focused questions where mediation is claimed: the article cautions
  that colocalization does not necessarily suggest mediation due to possible horizontal
  pleiotropic effects.'
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: CAFEH
  role: Generates the per-(SNP, tissue) p_active, pip, top_component, rank, and pi
    results table in z-score mode that feeds the linear mixed model of p_active versus
    tissue sharing.
  repo: github.com/karltayeb/cafeh
- name: conda
  role: Creates the cafeh environment with dependencies from environment.yml before
    running CAFEH.
  repo: github.com/karltayeb/cafeh
- name: pip
  role: Installs the CAFEH package inside the conda cafeh environment.
  repo: github.com/karltayeb/cafeh
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ajhg.2022.01.002
    title: Redefining tissue specificity of genetic regulation of gene expression
      in the presence of allelic heterogeneity
  source_content_hash: sha256:bf2b1b6950c49eed7ab1143b5beea53451516c7231cdd7015e0b3f05f2b9fec4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CAFEH
  - Python
  - conda
  - pip
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# linear-mixed-model-association-testing

## Summary

Fits a linear mixed model relating per-component CAFEH activity probability (p_active) to the number of GTEx tissues sharing each active eQTL component, testing whether tissue-specific components are preferentially active in GWAS loci across 19 UK Biobank traits.

## When to use

Use when you have CAFEH per-(SNP, tissue) results (pip, p_active, top_component, rank, pi) across many tissues and traits, and need to quantify whether tissue-restricted eQTL components are enriched for activity in GWAS loci.

## When NOT to use

- Loci with large LD differences between studies (e.g., ancestry differences): CAFEH relies on a single LD matrix, so cross-study inference may be biased before any mixed-model aggregation is attempted.
- Signals mediated via coding variants that lack eQTL effects: these cannot be explored with CAFEH and thus contribute no p_active values to the tissue-sharing analysis.
- Colocalization-focused questions where mediation is claimed: the article cautions that colocalization does not necessarily suggest mediation due to possible horizontal pleiotropic effects.

## Inputs

- CAFEH z-mode results tables (output/cafeh.z.results) per locus with pip, top_component, rank, p_active, pi columns for every (SNP, tissue) pair
- Reference LD matrices (-R) for genome-wide significant UK Biobank loci
- GTEx v8 cis-eQTL z-scores for 49 tissues (-z) with sample sizes (-n)

## Outputs

- Per-component tissue-sharing counts and quartile assignments for active components
- Linear mixed model estimate of mean p_active versus number of tissues shared, with p-value (expected ~3.3 × 10−160 inverse association)
- 2x2 CAD artery-versus-liver contingency comparison matching Figure 4D/4E

## How to apply

First run CAFEH in z-score mode (--mode z) for each genome-wide significant UK Biobank locus, providing the reference LD matrix (-R), cis-eQTL z-scores for 49 GTEx v8 tissues (-z), sample sizes (-n), and --save-model; set K = 20 causal components per locus so the algorithm prunes unneeded components. Parse each output/cafeh.z.results table, which contains a row for each (SNP, study) pair with pip, top_component, rank, p_active, and pi fields, and identify components with high p_active in each tissue. Count the number of GTEx tissues in which each active component is active and assign components to tissue-sharing quartiles. Then fit a linear mixed model of mean p_active versus the number of tissues shared across components, expecting a strong inverse association (linear mixed model p = 3.3 × 10−160), and assemble a 2x2 CAD artery-versus-liver contingency comparison analogous to Figure 4D/4E.

## Related tools

- **CAFEH** (Generates the per-(SNP, tissue) p_active, pip, top_component, rank, and pi results table in z-score mode that feeds the linear mixed model of p_active versus tissue sharing.) — github.com/karltayeb/cafeh
- **conda** (Creates the cafeh environment with dependencies from environment.yml before running CAFEH.) — github.com/karltayeb/cafeh
- **pip** (Installs the CAFEH package inside the conda cafeh environment.) — github.com/karltayeb/cafeh

## Examples

```
python cafeh.py --mode z -R example/LD_example.tsv -z example/z_example.tsv -n example/n_example.tsv --out output --save-model
```

## Evaluation signals

- The results table must contain pip, p_active, top_component, rank, and pi fields for every (SNP, tissue) pair.
- The linear mixed model must reproduce the significant negative p_active-versus-tissues-shared association with p = 3.3 × 10−160.
- Components shared by fewer tissues show higher probability of being active in GWAS (inverse p_active–tissue-sharing relationship).
- The 2x2 CAD artery/liver contingency comparison matches Figure 4D/4E qualitatively.

## Limitations

- CAFEH relies on a single LD matrix; large LD differences between studies (e.g., ancestry differences) could impact colocalization and fine-mapping inference.
- It is still difficult to distinguish the presence of multiple signals among variants in nearly perfect or very high LD with each other.
- CAFEH does not handle missing data in the effect size matrix, implicitly assuming causal variants are present in the tested datasets.

## Evidence

- [results] Components shared by fewer tissues show higher probability of being active in GWAS; there is a strong inverse association between p_active and the number of tissues colocalizing with the component (linear mixed model p = 3.3 × 10−160).: [evidence span withheld — anchor status: not_found]
- [readme] p_active gives the probability that top_component is an active component in this study: "`p_active` gives the probability that `top_component` is an active component in this study"
- [discussion] This is achieved by setting an arbitrarily high number of causal components for each CAFEH run (in our study, we used 20 components per locus).: [evidence span withheld — anchor status: not_found]
- [readme] To fit CAFEH with zscores, use `--mode z` in the arguments. You must provide input `--ld (-R)`, and `--zscores (-z)`.: "To fit CAFEH with zscores, use `--mode z` in the arguments. You must provide input `--ld (-R)`, and `--zscores (-z)`."
- [discussion] colocalization does not necessarily suggest mediation and it remains possible that colocalizing signals can be observed without a linking causal pathway due to horizontal pleiotropic effects: "colocalization does not necessarily suggest mediation and it remains possible that colocalizing signals can be observed without a linking causal pathway due to horizontal pleiotropic effects"
