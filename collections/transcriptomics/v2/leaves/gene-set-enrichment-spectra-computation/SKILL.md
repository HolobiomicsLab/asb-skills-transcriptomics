---
name: gene-set-enrichment-spectra-computation
description: Use when converting raw gene expression matrices (rows=samples, columns=Entrez
  gene IDs) into pathway-activity features using MSigDB gene sets, prior to training
  a DeepCC subtype classifier or classifying new/individual samples.
when_to_use_negative:
- Input is already a pathway/feature table (functional spectra) rather than a raw
  gene expression matrix
- The goal is unsupervised clustering of cancer patients — DeepCC is specifically
  developed for classification, not clustering
- No suitable gene set coverage exists for the subtype biology (the gene set prior
  knowledge must be comprehensive to avoid potential bias)
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: DeepCC
  role: Provides getFunctionalSpectra(), getFunctionalSpectrum(), and calcEnrichmentScore()
    to compute gene-set enrichment spectra from expression profiles
  repo: github.com/CityUHK-CompBio/DeepCC
- name: msigdbr
  role: R package supplying the MSigDBr gene set collection via get_msigdbr() as an
    alternative geneSets input
  repo: github.com/CityUHK-CompBio/DeepCC
- name: R
  role: Runtime environment in which the DeepCC enrichment-spectrum functions are
    executed
  repo: github.com/CityUHK-CompBio/DeepCC
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41389-019-0157-8
    title: 'DeepCC: a novel deep learning-based framework for cancer molecular subtype
      classification'
  source_content_hash: sha256:64f360475fa4ee25e34766094b185d267cb7aa9548a02e63143f85226b2eafa3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepCC
  - R
  - msigdbr
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# gene-set-enrichment-spectra-computation

## Summary

Transforms gene expression profiles into pathway-level 'functional spectra' by scoring MSigDB gene set enrichment per sample, serving as the platform-independent feature representation that DeepCC trains on.

## When to use

Use when converting raw gene expression matrices (rows=samples, columns=Entrez gene IDs) into pathway-activity features using MSigDB gene sets, prior to training a DeepCC subtype classifier or classifying new/individual samples.

## When NOT to use

- Input is already a pathway/feature table (functional spectra) rather than a raw gene expression matrix
- The goal is unsupervised clustering of cancer patients — DeepCC is specifically developed for classification, not clustering
- No suitable gene set coverage exists for the subtype biology (the gene set prior knowledge must be comprehensive to avoid potential bias)

## Inputs

- data.frame of gene expression profiles (one row per patient sample; column names must be Entrez gene identifiers)
- character vector of training labels (NA allowed; ignored automatically)
- MSigDB gene set collection (default MSigDB v7, 22,596 gene sets; or v5.0/v7.0 or MSigDBr from msigdbr)
- reference expression profile for single-sample classification (pre-defined TCGA cancer type identifier or customized profile)

## Outputs

- functional spectra matrix (fs) with one row per sample and one column per MSigDB gene set (enrichment scores)
- single-sample functional spectrum (new_fs_single) computed against a reference expression profile
- new_fs functional spectra for validation cohorts used with get_DeepCC_label

## How to apply

Prepare a data.frame of gene expression profiles where each row is one patient sample and column names are Entrez identifiers, then call getFunctionalSpectra(eps), which by default scores all MSigDB v7 gene sets (22,596 gene sets) via calcEnrichmentScore(geneList, geneSet); alternate collections can be passed as geneSets="MSigDBv5"/"MSigDBv7" or MSigDBr obtained from the R package msigdbr via get_msigdbr(). The resulting functional-spectra matrix (fs) becomes the training input for train_DeepCC_model(fs, labels) and is regenerated identically for new cohorts (new_fs <- getFunctionalSpectra(new_eps)) so features are comparable across platforms. For classifying a single sample, use getFunctionalSpectrum(new_ep, refExp = "COADREAD") with a reference expression profile — either a pre-defined TCGA cancer-type average (e.g., COADREAD, BRCA, OV) or a customized profile from same/similar cancer type and platform — and turn on inverseRescale for microarray data since TCGA references are RNA-Seq. The rationale is that pathway-level enrichment scores depend on gene set membership rather than platform-specific signature genes, making features robust to missing data and batch effects.

## Related tools

- **DeepCC** (Provides getFunctionalSpectra(), getFunctionalSpectrum(), and calcEnrichmentScore() to compute gene-set enrichment spectra from expression profiles) — github.com/CityUHK-CompBio/DeepCC
- **msigdbr** (R package supplying the MSigDBr gene set collection via get_msigdbr() as an alternative geneSets input) — github.com/CityUHK-CompBio/DeepCC
- **R** (Runtime environment in which the DeepCC enrichment-spectrum functions are executed) — github.com/CityUHK-CompBio/DeepCC

## Examples

```
fs <- getFunctionalSpectra(eps)
```

## Evaluation signals

- Output spectra dimensions match (n_samples x n_gene_sets) — e.g., 22,596 columns for the default MSigDB v7 collection
- Downstream classifier trained on spectra from randomly subsampled genes (down to 1000 genes) retains balanced accuracy above 80% (Fig. S6 acceptance criterion)
- Spectra for new cohorts (TANSBIG, UNT, UPP, NKI) are computed with the same geneSets version as training data so feature columns are comparable
- Single-sample spectra only compute when a reference expression profile is supplied; microarray samples require inverseRescale turned on to mitigate RNA-Seq vs microarray distribution differences
- Expression matrix column names are validated as Entrez identifiers before enrichment scoring

## Limitations

- Prior knowledge of gene sets needs to be comprehensive to avoid potential bias; major characteristic signaling pathways or biological processes for each cancer subtype should be included in the gene set database
- Enrichment-based transformation is robust to missing genes, but the deep classifier built on it is for classification only, not clustering of cancer patients
- For single microarray samples, the TCGA RNA-Seq reference profile may fit poorly — inverseRescale helps a little but a customized reference from the same cancer type and platform is strongly suggested

## Evidence

- [methods] default MSigDB v7 collection: "By default, DeepCC will use MSigDB v7 (22, 596 gene sets) to generate functional spectra."
- [methods] single-sample reference: "new_fs_single <- getFunctionalSpectrum(new_ep, refExp = "COADREAD")"
- [readme] input format: "a data.frame containing gene expression profiles (each row represents one patient sample and the column names should be Entrez identifiers of genes)"
- [readme] msigdbr alternative gene sets: "Add parameter `geneSets = "MSigDBvx"` (x = 5/6/7) or `geneSets = MSigDBr` (MSigDBr attained from R package `msigdbr` with function get_msigdbr() in DeepCC)"
- [readme] microarray inverseRescale: "if your single sample is microarray data, we strongly suggest you turn the parameter `inverseRescale` on, since TCGA is RNA-Seq data"
- [abstract] robustness rationale: "Simulation analysis based on random subsampling of genes demonstrated the robustness of DeepCC to missing data"
