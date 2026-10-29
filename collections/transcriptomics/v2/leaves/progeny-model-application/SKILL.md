---
name: progeny-model-application
description: Use when you need to infer signaling pathway activity (e.g. MAPK, PI3K,
  NFkB, TGFb, JAK-STAT) from bulk or single-cell gene expression, especially in perturbation
  experiments where pathway activity must be validated against controls or phospho-protein
  measurements.
when_to_use_negative:
- The pathway of interest has no PROGENy signature — the resource covers only the
  11 curated pathways (EGFR, Hypoxia, JAK-STAT, MAPK, NFkB, p53/DDR, PI3K, TGFb, Trail,
  VEGF/PDGF)
- You need tissue-specific pathway activity in a well-characterized context — the
  article notes specific-tissue signatures may more closely reflect activation than
  the consensus PROGENy signatures
- You want to quantify the expression level (abundance) of pathway members rather
  than pathway activity — the article warns the expression level of a pathway should
  not be interpreted as its activity
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3523
tools:
- name: progeny
  role: Bioconductor package providing the PROGENy pathway weight matrix used to compute
    pathway activity scores
  repo: https://github.com/saezlab/progeny
- name: decoupler
  role: Statistical method recommended by the progeny README to infer pathway activities
    from the PROGENy signatures in R or Python
  repo: https://github.com/scverse/decoupler
- name: DESeq2
  role: Normalization of RNA-seq counts (size-factor normalization and variance-stabilized
    expression) prior to scoring
- name: PROGENy
  role: 'The method itself: Pathway RespOnsive GENes signatures leveraged from a compendium
    of perturbation experiments for activity inference'
  repo: https://github.com/saezlab/progeny
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-017-02391-6
    title: Perturbation-response genes reveal signaling footprints in cancer gene
      expression
  source_content_hash: sha256:11507077ace11c4a264d3042b4538171610bdc482f81e79698d9a7f4e930dae6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saezlab/progeny
  tools:
  - DESeq2
  - PROGENy
  - decoupler
  - progeny
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3523
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# progeny-model-application

## Summary

Compute PROGENy pathway activity scores by taking weighted linear combinations of normalized gene expression using the Pathway RespOnsive GENes signatures, then compare perturbation conditions against controls to infer signaling pathway activity from transcriptomics.

## When to use

Use when you need to infer signaling pathway activity (e.g. MAPK, PI3K, NFkB, TGFb, JAK-STAT) from bulk or single-cell gene expression, especially in perturbation experiments where pathway activity must be validated against controls or phospho-protein measurements.

## When NOT to use

- The pathway of interest has no PROGENy signature — the resource covers only the 11 curated pathways (EGFR, Hypoxia, JAK-STAT, MAPK, NFkB, p53/DDR, PI3K, TGFb, Trail, VEGF/PDGF)
- You need tissue-specific pathway activity in a well-characterized context — the article notes specific-tissue signatures may more closely reflect activation than the consensus PROGENy signatures
- You want to quantify the expression level (abundance) of pathway members rather than pathway activity — the article warns the expression level of a pathway should not be interpreted as its activity

## Inputs

- RNA-seq or microarray count matrix (e.g. GSE97979 HEK293 perturbation samples with 4OHT, Ly294002, TNF-alpha, TGF-beta, IFN-gamma and BSA control)
- DESeq2 size-factor-normalized and variance-stabilized expression matrix
- PROGENy pathway weight matrix from the progeny Bioconductor package (human or mouse signatures)

## Outputs

- Per-sample PROGENy pathway activity scores for 11 signaling pathways
- pathway_score_deltas.csv with fields pathway, condition, delta_score (perturbation minus BSA control)

## How to apply

Download the perturbation RNA-seq count data (e.g. GSE97979 HEK293-ER samples: 4OHT, Ly294002, TNF-alpha, TGF-beta, IFN-gamma plus BSA vehicle controls) and normalize with DESeq2 size-factor normalization, using variance-stabilized expression for scoring. Load the PROGENy pathway weight matrix from the progeny Bioconductor package, which stores consensus signatures for 11 pathways (EGFR, Hypoxia, JAK-STAT, MAPK, NFkB, p53/DDR, PI3K, TGFb, Trail, VEGF/PDGF) derived from 208 ArrayExpress submissions with 580 experiments. Score each sample as the weighted linear combination of normalized expression using the PROGENy signatures (the progeny README recommends decoupleR for the scoring step). For each perturbation condition, subtract the corresponding BSA control score to get a delta_score per pathway. Rationale: PROGENy's consensus signatures capture a common core of responsive genes, avoiding context specificity of single-experiment signatures and approximating pathway activity despite post-translational control of signaling.

## Related tools

- **progeny** (Bioconductor package providing the PROGENy pathway weight matrix used to compute pathway activity scores) — https://github.com/saezlab/progeny
- **decoupler** (Statistical method recommended by the progeny README to infer pathway activities from the PROGENy signatures in R or Python) — https://github.com/scverse/decoupler
- **DESeq2** (Normalization of RNA-seq counts (size-factor normalization and variance-stabilized expression) prior to scoring)
- **PROGENy** (The method itself: Pathway RespOnsive GENes signatures leveraged from a compendium of perturbation experiments for activity inference) — https://github.com/saezlab/progeny

## Examples

```
BiocManager::install("progeny"); scores <- progeny(vst_expr_matrix, scale = TRUE, organism = "Human", top = 100)
```

## Evaluation signals

- pathway_score_deltas.csv exists and contains the required fields pathway, condition, delta_score
- The sign of each delta_score matches the direction of change reported for the stimulated pathways in Fig. 2d of the article
- PROGENy scores agree with independently measured phosphorylation changes of key pathway proteins (4OHT/RAF-MAPK, Ly294002/PI3K, TNF-alpha/NFkB, TGFb, IFN-gamma/JAK-STAT)
- Each perturbed pathway's delta_score is positive relative to the BSA vehicle control for activated stimuli

## Limitations

- Pathway methods on gene expression have produced mixed results due to lack of robustness, as suggested by heterogeneity in responses of individual signatures
- Consensus PROGENy signatures provide a good estimate in large heterogeneous datasets, but tissue-specific signatures may more closely reflect pathway activation in a specific context
- Signaling is post-translationally controlled; gene-expression-based scores only approximate pathway activity, and more studies are needed on differences between the common response and additional transcriptional modules
- Scores are not directly applicable to pathways outside the 11 signatures; the footprints repository notes excluded experiments where the perturbation did not correspond to the intended phenotype

## Evidence

- [abstract] PROGENy accurately infers pathway activity from gene expression in a wide range of conditions: "these results show that PROGENy accurately infers pathway activity from gene expression in a wide range of conditions"
- [intro] It is unclear if pathway scores reflect signaling activity since they do not account for post-translational modifications: "do not take into account the effect of post-translational modifications (Fig. 1a). It is therefore unclear if and under what circumstances the pathway scores obtained by these methods reflect"
- [methods] PROGENy weight matrix is loaded from the progeny Bioconductor package: "package: "`r pkg_ver('progeny')`""
- [readme] PROGENy leverages perturbation experiments to yield a common core of pathway responsive genes: "PROGENy is resource that leverages a large compendium of publicly available signaling perturbation experiments to yield a common core of pathway responsive"
- [readme] To infer pathway activities use decoupleR: "To infer pathway activities, please check out
[decoupleR](https://doi.org/10.1093/bioadv/vbac016)"
- [readme] The 11 pathways span 208 ArrayExpress submissions with 580 experiments: "The 11 pathways comprised of 208 submissions to
[ArrayExpress](https://www.ebi.ac.uk/arrayexpress/) with a total of 580
experiments"
