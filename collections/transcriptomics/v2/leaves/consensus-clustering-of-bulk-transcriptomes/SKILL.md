---
name: consensus-clustering-of-bulk-transcriptomes
description: Use when you have a curated bulk PDAC expression dataset (e.g. TCGA-PAAD
  whitelisted to 150 PDAC samples) and gene-symbol-level subtype signatures, and you
  need to assign subtype labels or compare concordance between subtyping schemes.
when_to_use_negative:
- Input is single-cell RNA-seq data — use the rmoffitt/scOh single-cell atlas and
  classifier instead; bulk-consensus subtyping cannot separate cellular populations
- The dataset is uncurated TCGA-PAAD (all 182 samples) — non-PDAC tumors must be removed
  via the 'Decision'/'whitelist' filter first or signature derivation/validation is
  misled
- You need to interchangeably relabel samples as basal-like, squamous, or QM without
  measuring overlap — the paper shows the collapsed signatures are not completely
  redundant
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: pdacR
  role: Provides the curated bulk PDAC expression datasets, the 'Decision'/'whitelist'
    sample filter, and the SYMBOL-level subtype gene signature objects used as clustering
    input
  repo: https://github.com/rmoffitt/pdacR.git
- name: ConsensusClusterPlus
  role: R package used to cluster samples restricted to each subtype signature's genes
    and derive consensus subtype groups
- name: R
  role: Implementation environment for the concordance workflow and cross-tabulation
    of subtype labels
  repo: https://github.com/rmoffitt/pdacR
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-023-04461-6
    title: Open-source curation of a pancreatic ductal adenocarcinoma gene expression
      analysis platform (pdacR) supports a two-subtype model
  source_content_hash: sha256:c122f951f0dc42eff0d7ccc4434dea0854670fe84ecae082a3d42be9170b8d1e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rmoffitt/pdacR.git
  tools:
  - ConsensusClusterPlus
  - R
  - pdacR
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# consensus-clustering-of-bulk-transcriptomes

## Summary

Derives subtype groups from bulk PDAC expression matrices by restricting to curated subtype gene signatures (Moffitt, Collisson, Bailey) loaded from pdacR and clustering samples with ConsensusClusterPlus, enabling cross-scheme subtype concordance testing (~60% overlap between analogous calls).

## When to use

Use when you have a curated bulk PDAC expression dataset (e.g. TCGA-PAAD whitelisted to 150 PDAC samples) and gene-symbol-level subtype signatures, and you need to assign subtype labels or compare concordance between subtyping schemes.

## When NOT to use

- Input is single-cell RNA-seq data — use the rmoffitt/scOh single-cell atlas and classifier instead; bulk-consensus subtyping cannot separate cellular populations
- The dataset is uncurated TCGA-PAAD (all 182 samples) — non-PDAC tumors must be removed via the 'Decision'/'whitelist' filter first or signature derivation/validation is misled
- You need to interchangeably relabel samples as basal-like, squamous, or QM without measuring overlap — the paper shows the collapsed signatures are not completely redundant

## Inputs

- pdacR curated bulk PDAC expression dataset (e.g. TCGA-PAAD filtered to the 150 whitelisted PDAC samples via the 'Decision' sample track)
- Curated subtype gene signatures as SYMBOL-level gene sets (Moffitt, Collisson, Bailey) from pdacR gene_set_list.RData / parse_gene_lists

## Outputs

- Per-sample subtype assignments under Moffitt (basal-like/classical), Collisson (classical/QM/exocrine), and Bailey (progenitor/squamous/ADEX/immunogenic) schemes
- subtype_concordance.csv with per-sample assignments and pairwise concordance percentages between analogous subtypes

## How to apply

Load a curated PDAC bulk expression dataset from pdacR, filtering TCGA-PAAD via the 'Decision' sample track to retain only the 150 'whitelist' histologically confirmed PDAC samples (the raw mRNA dataset has 182 samples, 32 of which are non-PDAC). Load the curated subtype signatures (Moffitt basal-like/classical, Collisson classical/QM/exocrine, Bailey progenitor/squamous/ADEX/immunogenic) as SYMBOL-level gene sets using pdacR::parse_gene_lists / gene_set_list.RData. For each signature, restrict the expression matrix to that signature's genes and run ConsensusClusterPlus to derive subtype groups, then assign per-sample labels under all three schemes. Cross-tabulate Moffitt calls against Collisson and Bailey labels and compute pairwise overlap percentages between analogous subtypes, writing per-sample assignments and concordance percentages to subtype_concordance.csv. Rationale: the paper emphasizes that collapsed 3+ subtype signatures (squamous, QM) are 'not completely redundant' with the two-subtype basal-like/classical model, so concordance must be measured empirically rather than assumed.

## Related tools

- **pdacR** (Provides the curated bulk PDAC expression datasets, the 'Decision'/'whitelist' sample filter, and the SYMBOL-level subtype gene signature objects used as clustering input) — https://github.com/rmoffitt/pdacR.git
- **ConsensusClusterPlus** (R package used to cluster samples restricted to each subtype signature's genes and derive consensus subtype groups)
- **R** (Implementation environment for the concordance workflow and cross-tabulation of subtype labels) — https://github.com/rmoffitt/pdacR

## Examples

```
devtools::install_github("rmoffitt/pdacR"); pdacR::pdacShiny()  # filter TCGA-PAAD via Decision track to 'whitelist' (150 PDAC samples), then cluster signature genes with ConsensusClusterPlus
```

## Evaluation signals

- Moffitt-vs-Collisson and Moffitt-vs-Bailey overlap percentages for analogous subtypes should approximate the ~60% concordance reported in Supplementary Fig. 1b,c
- TCGA-PAAD analysis must contain exactly 150 samples after applying the 'Decision' track with only 'whitelist' retained (not 182)
- subtype_concordance.csv contains one row per sample with all three scheme labels plus the pairwise overlap percentage tables
- Clustering is restricted to each signature's own genes (Moffitt, Collisson, Bailey) — confirm gene sets were parsed as SYMBOL-level via pdacR::parse_gene_lists before clustering

## Limitations

- Bulk transcriptomic data limit the ability to fully separate cellular populations; purity and tissue-specificity confounders differ between the two-subtype and collapsed 3+ subtype signatures
- Datasets span multiple platforms (bulk RNA-seq and microarrays), which adds complexity to any attempt to merge or batch-correct across datasets before clustering
- Collapsing squamous/QM calls into basal-like/classical labels is not equivalent because the signatures are not completely redundant

## Evidence

- [readme] To filter to the correct 150, select "Decision", and then remove all variables except "whitelist".: "TCGA-PAAD has 182 samples, but only 150 are PDAC. The remainder have been identified to be non-PDAC tumors in previous publications. To filter to the correct 150, select "Decision", and then remove"
- [intro] Collapsing 3+ subtype models into basal-like/classical is not equivalent: "Given that the two-subtype and collapsed 3+ subtype signatures are not completely redundant, attempting to interchangeably label a subset of the data as basal-like, squamous, or QM is not equivalent."
- [discussion] Bulk data limit cellular population separation: "Given that the datasets provided here are all bulk analyses, we are limited in our ability to fully separate cellular populations"
- [intro] Validation often performed on only one dataset: "First, validation is frequently only performed on one publicly available dataset."
