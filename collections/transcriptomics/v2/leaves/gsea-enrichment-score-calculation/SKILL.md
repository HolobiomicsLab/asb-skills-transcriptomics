---
name: gsea-enrichment-score-calculation
description: Use when a gene expression matrix (samples x genes, Entrez IDs) must
  be converted into per-sample pathway activity vectors (functional spectra) as classification
  or visualization features.
when_to_use_negative:
- Input is already a feature/embedding table (e.g. deep features from get_DeepCC_features)
  rather than raw gene expression
- The goal is unsupervised clustering of cancer patients — DeepCC's functional spectra
  pipeline is built for supervised classification, not clustering
- The gene set database lacks the major characteristic signaling pathways for the
  cancer subtype, since comprehensive prior knowledge is needed to avoid bias
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3523
tools:
- name: DeepCC
  role: Provides getFunctionalSpectra, getFunctionalSpectrum, get_gene_sets, and calcEnrichmentScore
    functions that perform the ranked-list GSEA scoring and functional spectra assembly
  repo: github.com/CityUHK-CompBio/DeepCC
- name: R
  role: Language in which the enrichment scoring pipeline is implemented and executed
  repo: github.com/CityUHK-CompBio/DeepCC
- name: msigdbr
  role: Supplies MSigDB gene sets (MSigDBr via get_msigdbr()) as an alternative to
    the versioned MSigDBv5/6/7 options
  repo: github.com/CityUHK-CompBio/DeepCC
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41389-019-0157-8
    title: 'DeepCC: a novel deep learning-based framework for cancer molecular subtype
      classification'
  source_content_hash: sha256:ab3efa554eccc4b8f0f7ceefa3acd54b9332690d6dd3b302cba501ef4d2fb6ad
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepCC
  - R
  - msigdbr
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3523
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# gsea-enrichment-score-calculation

## Summary

Transforms each tumor sample's high-throughput gene expression profile into a functional spectrum by computing GSEA enrichment scores over every gene set in a MSigDB version (e.g. 17,779 sets for MSigDB v6.0), producing the pathway-activity feature vector consumed by the DeepCC classifier.

## When to use

Use when a gene expression matrix (samples x genes, Entrez IDs) must be converted into per-sample pathway activity vectors (functional spectra) as classification or visualization features.

## When NOT to use

- Input is already a feature/embedding table (e.g. deep features from get_DeepCC_features) rather than raw gene expression
- The goal is unsupervised clustering of cancer patients — DeepCC's functional spectra pipeline is built for supervised classification, not clustering
- The gene set database lacks the major characteristic signaling pathways for the cancer subtype, since comprehensive prior knowledge is needed to avoid bias

## Inputs

- Gene expression profile data.frame (eps), rows = patient samples, columns = Entrez gene identifiers
- MSigDB gene set list (e.g. MSigDB v6.0 with 17,779 gene sets, v5.0 with 10,348, v7 with 22,596, or MSigDBr from msigdbr)
- Reference expression profile (refExp) for single-sample mode, e.g. a TCGA cancer type identifier or a customized profile

## Outputs

- Sample-by-gene-set enrichment score matrix (functional spectra), written to functional_spectra.csv, with one enrichment score per gene set per sample
- Per-sample functional spectrum vector for single-sample prediction

## How to apply

Load the input gene expression profile (eps) as a data.frame with Entrez gene identifier columns and obtain the gene set list via get_gene_sets (e.g. geneSets = 'MSigDBv6' for the 17,779-set v6.0 database, or MSigDBr from the R package msigdbr via get_msigdbr()). For each sample, compute log2 fold changes of all genes and sort them into a ranked gene list, then call calcEnrichmentScore(geneList, geneSet) for every gene set, parallelizing across cores with getFunctionalSpectra(eps, geneSets, scale, cores). Apply the configured scale option to the enrichment score matrix and assemble one functional spectrum vector per sample with one entry per gene set. Single-sample classification additionally requires a reference expression profile passed to getFunctionalSpectrum (e.g. refExp = 'COADREAD' for pre-defined TCGA averages), with inverseRescale suggested for microarray data.

## Related tools

- **DeepCC** (Provides getFunctionalSpectra, getFunctionalSpectrum, get_gene_sets, and calcEnrichmentScore functions that perform the ranked-list GSEA scoring and functional spectra assembly) — github.com/CityUHK-CompBio/DeepCC
- **R** (Language in which the enrichment scoring pipeline is implemented and executed) — github.com/CityUHK-CompBio/DeepCC
- **msigdbr** (Supplies MSigDB gene sets (MSigDBr via get_msigdbr()) as an alternative to the versioned MSigDBv5/6/7 options) — github.com/CityUHK-CompBio/DeepCC

## Examples

```
fs <- getFunctionalSpectra(eps, geneSets = "MSigDBv6", cores = 8)
```

## Evaluation signals

- functional_spectra.csv exists and is a valid CSV
- Each sample row contains exactly 17,779 enrichment score values when using MSigDB v6.0, matching the gene set count reported in the paper
- Each functional spectrum has one entry per gene set in the selected MSigDB version (10,348 for v5.0; 22,596 for default v7)
- Enrichment scores are computed against log2 fold-change-ranked gene lists derived from the input expression profile
- For single-sample calls, the reference profile matches the sample's cancer type and platform (inverseRescale on for microarray vs TCGA RNA-Seq)

## Limitations

- Single-sample microarray input differs in distribution from TCGA RNA-Seq references; inverseRescale only partly overcomes this, and a customized reference profile from matched platform data is strongly suggested
- Prior knowledge of gene sets must be comprehensive (e.g. include the subtype's major characteristic signaling pathways) to avoid potential bias
- Computation over all MSigDB gene sets for every sample is costly; parallelization via the cores parameter of getFunctionalSpectra is expected

## Evidence

- [other] functional spectrum of a patient sample is defined as the vector of GSEA enrichment scores computed over all gene sets: [evidence span withheld — anchor status: not_found]
- [methods] getFunctionalSpectra signature: "getFunctionalSpectra(eps, geneSets, scale, cores)"
- [methods] gene set versions and counts: "By default, DeepCC will use MSigDB v7 (22, 596 gene sets) to generate functional spectra. You can also use MSigDB v5.0 (10, 348 gene sets)"
- [readme] single-sample reference profile requirement: "for a given single sample, you have to provide a reference expression profile.
new_fs_single <- getFunctionalSpectrum(new_ep, refExp = "COADREAD")"
- [readme] inverseRescale caveat for microarray: "if your single sample is microarray data, we strongly suggest you turn the parameter `inverseRescale` on, since TCGA is RNA-Seq data"
- [methods] MSigDBr from msigdbr: "MSigDBr attained from R package `msigdbr` with function get_msigdbr() in DeepCC"
