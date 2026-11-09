---
name: functional-spectrum-feature-extraction
description: Use when raw gene expression profiles (data.frame with Entrez-ID columns,
  one row per patient) must be converted into pathway-level feature vectors before
  training or applying a DeepCC cancer subtype classifier.
when_to_use_negative:
- The input is already a functional spectra / feature matrix — pass it directly to
  train_DeepCC_model or get_DeepCC_label instead of re-transforming
- The goal is clustering of cancer patients rather than supervised classification
  — DeepCC is specifically developed for classification but not clustering
- Samples should be assigned labels without transformation when a trained classifier
  expects raw signature-gene input — DeepCC requires the functional spectra representation,
  not raw expression, as classifier input
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: DeepCC
  role: R package providing getFunctionalSpectra, getFunctionalSpectrum, get_gene_sets
    and calcEnrichmentScore for the GSEA-based transformation
  repo: https://github.com/CityUHK-CompBio/DeepCC
- name: R
  role: Implementation environment; DeepCC installs via devtools/remotes and depends
    on the keras R framework
- name: msigdbr
  role: Supplies MSigDBr gene sets via get_msigdbr() as an alternative to the bundled
    MSigDBv5/v6/v7 geneSets parameter
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41389-019-0157-8
    title: 'DeepCC: a novel deep learning-based framework for cancer molecular subtype
      classification'
  source_content_hash: sha256:702b36a1d48cdea551d8823a9cac0eec4885a4f6c12b83f2e9ff2fea6aa0f683
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CityUHK-CompBio/DeepCC
  tools:
  - DeepCC
  - R
  - msigdbr
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# functional-spectrum-feature-extraction

## Summary

Transforms raw tumor gene expression profiles into DeepCC 'functional spectra' by computing GSEA enrichment scores of every sample against all MSigDB gene sets (e.g., 17,779 gene sets in MSigDB v6.0), yielding a sample-by-gene-set feature matrix used as classifier input.

## When to use

Use when raw gene expression profiles (data.frame with Entrez-ID columns, one row per patient) must be converted into pathway-level feature vectors before training or applying a DeepCC cancer subtype classifier.

## When NOT to use

- The input is already a functional spectra / feature matrix — pass it directly to train_DeepCC_model or get_DeepCC_label instead of re-transforming
- The goal is clustering of cancer patients rather than supervised classification — DeepCC is specifically developed for classification but not clustering
- Samples should be assigned labels without transformation when a trained classifier expects raw signature-gene input — DeepCC requires the functional spectra representation, not raw expression, as classifier input

## Inputs

- Gene expression profile data.frame (eps), one row per patient sample, column names = Entrez gene identifiers
- MSigDB gene set list (v5.0: 10,348; v6.0: 17,779; v7: 22,596 gene sets) via get_gene_sets or msigdbr::get_msigdbr()
- For single samples: reference expression profile (pre-defined TCGA identifier such as "COADREAD"/"BRCA"/"OV" or a customized profile)

## Outputs

- Sample-by-gene-set matrix of GSEA enrichment scores (functional spectra), written to functional_spectra.csv with 17,779 values per sample row when using MSigDB v6.0
- Single-sample functional spectrum vector when called via getFunctionalSpectrum (new_fs_single)

## How to apply

Load the input expression data.frame (eps) with Entrez gene identifiers as column names, and select the MSigDB version via get_gene_sets — either "MSigDBv6" (17,779 gene sets as used in the case studies) or the default MSigDB v7 (22,596 gene sets), or MSigDBr from the msigdbr package via get_msigdbr(). For each sample, compute log2 fold changes of all genes and sort them into a ranked gene list, then call calcEnrichmentScore(geneList, geneSet) for every gene set, parallelized across cores as supported by getFunctionalSpectra(eps, geneSets, scale, cores); apply the scale parameter to the enrichment-score matrix. Assemble one functional spectrum vector per sample with one entry per gene set and write the sample-by-gene-set matrix to functional_spectra.csv. For a single sample, use getFunctionalSpectrum(expressionProfile, geneSets, refExp, logChange, inverseRescale, filter) supplying a reference expression profile — DeepCC ships average TCGA profiles per cancer type (e.g., refExp = "COADREAD"), and for microarray input enable inverseRescale because the TCGA reference is RNA-Seq. The rationale is to replace platform-dependent signature genes with platform-independent pathway activity features, mitigating batch effects and enabling single-sample classification.

## Related tools

- **DeepCC** (R package providing getFunctionalSpectra, getFunctionalSpectrum, get_gene_sets and calcEnrichmentScore for the GSEA-based transformation) — https://github.com/CityUHK-CompBio/DeepCC
- **R** (Implementation environment; DeepCC installs via devtools/remotes and depends on the keras R framework)
- **msigdbr** (Supplies MSigDBr gene sets via get_msigdbr() as an alternative to the bundled MSigDBv5/v6/v7 geneSets parameter)

## Examples

```
fs <- getFunctionalSpectra(eps)  # or: fs <- getFunctionalSpectra(eps, geneSets = "MSigDBv6", cores = 4)
```

## Evaluation signals

- functional_spectra.csv exists, is valid CSV, and each sample row contains one enrichment score per gene set — 17,779 values for MSigDB v6.0, matching the count reported in the paper
- Gene set count matches the declared MSigDB version: 10,348 (v5.0), 17,779 (v6.0), or 22,596 (v7)
- Every gene column in eps is an Entrez identifier (README requirement for the input data.frame)
- The scale option of getFunctionalSpectra was applied as configured; ranked lists derive from log2 fold changes of all genes before enrichment scoring
- For single-sample calls on microarray data, inverseRescale was enabled because the TCGA reference profiles are RNA-Seq

## Limitations

- Prior knowledge of gene sets needs to be comprehensive to avoid potential bias; major characteristic signaling pathways or biological processes for each cancer subtype should be included in the gene set database
- MSigDB version choice changes dimensionality (10,348 / 17,779 / 22,596 gene sets), so spectra from different versions are not directly interchangeable
- For single microarray samples versus the RNA-Seq TCGA reference, inverseRescale 'can overcome this distribution differently a little bit, but it's better to generate your customized reference'

## Evidence

- [methods] compute functional spectra from gene expression profiles: "fs <- getFunctionalSpectra(eps)"
- [methods] geneSets parameter controls MSigDB version: "Add parameter `geneSets = "MSigDBvx"` (x = 5/6/7) or `geneSets = MSigDBr` (MSigDBr attained from R package `msigdbr`"
- [readme] default gene set database in current release: "By default, DeepCC will use MSigDB v7 (22, 596 gene sets) to generate functional spectra."
- [methods] single-sample transformation requires a reference expression profile: "getFunctionalSpectrum(expressionProfile, geneSets, refExp, logChange, inverseRescale, filter)"
- [readme] microarray vs RNA-Seq reference mismatch handling: "if your single sample is microarray data, we strongly suggest you turn the parameter `inverseRescale` on, since TCGA is RNA-Seq data"
- [discussion] paper reports MSigDB v6.0 gene set count used in case studies: "MSigDB v6.0 database employed in our case studies, which includes 17,779 gene sets"
