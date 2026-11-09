---
name: cross-subtyping-concordance-analysis
description: Use when the same curated PDAC bulk expression samples must be labeled
  under multiple published subtype schemes (Moffitt, Collisson, Bailey) to measure
  pairwise overlap between analogous subtypes.
when_to_use_negative:
- The goal is single-sample subtype classification rather than comparing schemes across
  a cohort — use the PurIST classifier instead of re-clustering with ConsensusClusterPlus.
- The expression matrix contains non-PDAC or low-purity cases that were not filtered
  (e.g. unfiltered TCGA-PAAD with 182 samples) — subtype signature derivation or validation
  will be misleading.
- Input is single-cell data; pdacR datasets are bulk analyses that limit the ability
  to fully separate cellular populations.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: pdacR
  role: Supplies the curated bulk expression datasets, sample-track filtering (Decision/whitelist),
    and the SYMBOL-level subtype gene signature objects (gene_set_list.RData) used
    as inputs.
  repo: github.com/rmoffitt/pdacR
- name: ConsensusClusterPlus
  role: Clusters samples restricted to each signature's genes to derive subtype groups
    for each subtyping scheme.
- name: R
  role: Language in which pdacR and the concordance analysis are implemented.
  repo: github.com/rmoffitt/pdacR
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-023-04461-6
    title: Open-source curation of a pancreatic ductal adenocarcinoma gene expression
      analysis platform (pdacR) supports a two-subtype model
  source_content_hash: sha256:2f415322a34fb8568dbd7d53a30555d183095b48665f060763b98f67ef6daa50
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ConsensusClusterPlus
  - R
  - pdacR
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# cross-subtyping-concordance-analysis

## Summary

Derives PDAC subtype calls for the same bulk expression samples under the Moffitt (basal-like/classical), Collisson (classical/QM/exocrine), and Bailey (progenitor/squamous/ADEX/immunogenic) schemes, then cross-tabulates analogous subtypes to quantify concordance (~60% overlap between analogous calls).

## When to use

Use when the same curated PDAC bulk expression samples must be labeled under multiple published subtype schemes (Moffitt, Collisson, Bailey) to measure pairwise overlap between analogous subtypes.

## When NOT to use

- The goal is single-sample subtype classification rather than comparing schemes across a cohort — use the PurIST classifier instead of re-clustering with ConsensusClusterPlus.
- The expression matrix contains non-PDAC or low-purity cases that were not filtered (e.g. unfiltered TCGA-PAAD with 182 samples) — subtype signature derivation or validation will be misleading.
- Input is single-cell data; pdacR datasets are bulk analyses that limit the ability to fully separate cellular populations.

## Inputs

- Curated PDAC bulk expression dataset from pdacR (e.g. TCGA-PAAD filtered to the 150 'whitelist' samples via the 'Decision' sample track)
- Curated SYMBOL-level subtype gene signature sets (Moffitt basal-like/classical, Collisson classical/QM/exocrine, Bailey progenitor/squamous/ADEX/immunogenic) from pdacR gene_set_list.RData / parse_gene_lists

## Outputs

- subtype_concordance.csv containing per-sample subtype assignments under all three schemes and pairwise concordance percentages between analogous subtypes
- Cross-tabulation tables of Moffitt vs. Collisson and Moffitt vs. Bailey subtype calls

## How to apply

Load a curated PDAC bulk expression dataset via pdacR — for TCGA-PAAD, filter 182 samples down to the 150 whitelisted PDAC cases using the 'Decision' sample track (removing all non-'whitelist' labels), since including the other 32 samples misleads signature validation. Load the curated SYMBOL-level subtype gene signatures (Moffitt, Collisson, Bailey) from the pdacR gene_set_list.RData object. For each signature, restrict the expression matrix to its genes and cluster samples with ConsensusClusterPlus to derive subtype groups, assigning each sample a label under all three schemes. Cross-tabulate the Moffitt basal-like/classical calls against the Collisson classical/QM and Bailey progenitor/squamous labels and compute pairwise overlap percentages between analogous subtypes, writing per-sample assignments and concordance percentages to subtype_concordance.csv. The rationale is that the two-subtype and collapsed 3+ subtype signatures are not completely redundant, so interchangeably relabeling them is not equivalent — concordance must be measured explicitly rather than assumed.

## Related tools

- **pdacR** (Supplies the curated bulk expression datasets, sample-track filtering (Decision/whitelist), and the SYMBOL-level subtype gene signature objects (gene_set_list.RData) used as inputs.) — github.com/rmoffitt/pdacR
- **ConsensusClusterPlus** (Clusters samples restricted to each signature's genes to derive subtype groups for each subtyping scheme.)
- **R** (Language in which pdacR and the concordance analysis are implemented.) — github.com/rmoffitt/pdacR

## Examples

```
Rscript -e 'devtools::install_github("rmoffitt/pdacR"); library(pdacR); library(ConsensusClusterPlus)'
```

## Evaluation signals

- Moffitt-vs-Collisson and Moffitt-vs-Bailey overlap percentages for analogous subtypes approximate the ~60% concordance reported in Supplementary Fig. 1b,c.
- subtype_concordance.csv contains a subtype label under all three schemes for every sample and pairwise overlap percentages for the analogous subtypes.
- The sample set is exactly the 150 whitelisted PDAC cases, not the full 182-sample TCGA-PAAD matrix.
- Each signature's clustering is run on the expression matrix restricted to that signature's genes, not on the full gene space.

## Limitations

- The basal-like/classical two-subtype model and the collapsed 3+ subtype signatures (squamous, QM) are not completely redundant, so substituting one label set for another is not equivalent.
- Bulk transcriptomic data limit the ability to fully separate cellular populations, confounding subtype calls with purity and tissue-specificity signals.
- pdacR datasets span multiple platforms (bulk RNA-seq and microarrays), so datasets cannot be robustly merged across platforms for a single concordance analysis.

## Evidence

- [readme] TCGA-PAAD filtering to whitelist: "For example, TCGA-PAAD has 182 samples, but only 150 are PDAC."
- [readme] Decision track whitelist filter: "To filter to the correct 150, select "Decision", and then remove all variables except "whitelist"."
- [methods] install command: "To install the package, simply run
```
devtools::install_github("rmoffitt/pdacR")
```"
- [intro] signatures not redundant: "Given that the two-subtype and collapsed 3+ subtype signatures are not completely redundant, attempting to interchangeably label a subset of the data as basal-like, squamous, or QM is not equivalent."
- [intro] only 150 histologically confirmed PDAC: "The mRNA dataset includes 182 samples, but only 150 have been confirmed histologically as PDAC"
