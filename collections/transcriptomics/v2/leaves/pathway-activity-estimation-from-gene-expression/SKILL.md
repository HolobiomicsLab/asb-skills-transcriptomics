---
name: pathway-activity-estimation-from-gene-expression
description: Use when a gene expression matrix (rows = samples, columns = Entrez gene
  IDs) must be converted into gene-set/pathway activity features for cancer subtype
  classification or to test robustness to incomplete gene annotation.
when_to_use_negative:
- The goal is unsupervised clustering of cancer patients — DeepCC is developed for
  classification but not clustering
- Input is already a pathway/gene-set activity or deep-feature table rather than raw
  gene expression (spectra computation would be redundant)
- Only one sample is available with no reference expression profile for the same/similar
  cancer type and platform
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3517
tools:
- name: DeepCC
  role: R package providing getFunctionalSpectra / getFunctionalSpectrum to transform
    expression profiles into MSigDB-based functional spectra
  repo: https://github.com/CityUHK-CompBio/DeepCC
- name: msigdbr
  role: R package used via get_msigdbr() in DeepCC to supply the MSigDBr gene set
    collection for spectra computation
  repo: https://github.com/CityUHK-CompBio/DeepCC
- name: keras
  role: 'Downstream dependency: functional spectra are used to train the keras ANN
    DeepCC model and evaluate via cross_validation'
  repo: https://github.com/CityUHK-CompBio/DeepCC
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41389-019-0157-8
    title: 'DeepCC: a novel deep learning-based framework for cancer molecular subtype
      classification'
  source_content_hash: sha256:ad525468b150e0db74260543c474d727c12266177ea146bc9f57c7966a0cc060
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CityUHK-CompBio/DeepCC
  tools:
  - DeepCC
  - R
  - keras
  - msigdbr
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# pathway-activity-estimation-from-gene-expression

## Summary

Compute DeepCC 'functional spectra' — pathway/gene-set activity scores that transform a gene expression profile into MSigDB gene-set space using tools like getFunctionalSpectra over MSigDB v7 (22,596 gene sets). This transformation is what makes the DeepCC classifier platform independent and robust to missing genes across platforms.

## When to use

Use when a gene expression matrix (rows = samples, columns = Entrez gene IDs) must be converted into gene-set/pathway activity features for cancer subtype classification or to test robustness to incomplete gene annotation.

## When NOT to use

- The goal is unsupervised clustering of cancer patients — DeepCC is developed for classification but not clustering
- Input is already a pathway/gene-set activity or deep-feature table rather than raw gene expression (spectra computation would be redundant)
- Only one sample is available with no reference expression profile for the same/similar cancer type and platform

## Inputs

- data.frame of gene expression profiles (rows = patient samples, column names = Entrez gene identifiers)
- character vector of training labels (NA allowed)
- MSigDB gene set collection (MSigDB v5/v6/v7 string, or MSigDBr object from msigdbr::get_msigdbr())
- Reference expression profile for single-sample estimation (e.g., TCGA 'COADREAD' identifier or customized profile)

## Outputs

- Functional spectra matrix (samples x gene sets / pathway activity scores) consumable by train_DeepCC_model or get_DeepCC_label
- For robustness workflows: per-subset metrics such as accuracy, balanced accuracy, mean sensitivity, and mean specificity (e.g., gene_subsampling_performance.csv)

## How to apply

Prepare a data.frame of gene expression profiles with column names as Entrez identifiers and call getFunctionalSpectra(eps) in R; DeepCC defaults to MSigDB v7 (22,596 gene sets), but other collections can be passed via geneSets = 'MSigDBvx' (x = 5/6/7) or a MSigDBr object built from the msigdbr package with get_msigdbr(). For a single sample, use getFunctionalSpectrum(new_ep, refExp = 'COADREAD') with a reference expression profile — TCGA cancer-type references (COADREAD, BRCA, OV) are pre-defined, and inverseRescale should be enabled for microarray data versus RNA-Seq references. The resulting functional spectra feed directly into train_DeepCC_model or get_DeepCC_label, or support robustness experiments: rank genes by variance, draw random subsets (e.g., 1000 to 20,531 genes), recompute spectra per subset, and retrain/evaluate with cross_validation. The rationale is that gene-set-level aggregation absorbs cross-platform annotation discrepancies, so classification quality degrades only gradually as genes are removed.

## Related tools

- **DeepCC** (R package providing getFunctionalSpectra / getFunctionalSpectrum to transform expression profiles into MSigDB-based functional spectra) — https://github.com/CityUHK-CompBio/DeepCC
- **msigdbr** (R package used via get_msigdbr() in DeepCC to supply the MSigDBr gene set collection for spectra computation) — https://github.com/CityUHK-CompBio/DeepCC
- **keras** (Downstream dependency: functional spectra are used to train the keras ANN DeepCC model and evaluate via cross_validation) — https://github.com/CityUHK-CompBio/DeepCC

## Examples

```
fs <- getFunctionalSpectra(eps)
```

## Evaluation signals

- Balanced accuracy stays > 90% even when functional spectra are computed from only 2000 of the most variable TCGA CRC genes
- Accuracy and balanced accuracy degrade only gradually as the random gene subset shrinks from 20,531 toward 1000 genes, matching the Fig. 1c trend
- Output spectra dimensions correspond to the chosen MSigDB collection size (22,596 gene sets for default MSigDB v7; 10,348 for v5.0)
- Expression matrix column names are Entrez identifiers, as required by DeepCC inputs
- Downstream cross_validation on spectra-derived features yields overall higher sensitivity, specificity, and accuracy than random forests, SVM, GBM, and multinomial logistic regression

## Limitations

- Gene set prior knowledge must be comprehensive to avoid potential bias; major characteristic signaling pathways for each subtype should be included in the gene set database
- Single-sample spectra for microarray data versus TCGA RNA-Seq references may show distribution differences; inverseRescale only partially mitigates this, and a customized reference is preferred
- Performance degrades as the gene subset becomes very small; the exact accuracy curve depends on the variance-ranked gene pool of the source cohort

## Evidence

- [methods] MSigDB v7 default: "By default, DeepCC will use MSigDB v7 (22, 596 gene sets) to generate functional spectra."
- [readme] input format: "a data.frame containing gene expression profiles (each row represents one patient sample and the column names should be Entrez identifiers of genes)"
- [abstract] robustness finding: "Simulation analysis based on random subsampling of genes demonstrated the robustness of DeepCC to missing data"
- [readme] single sample: "new_fs_single <- getFunctionalSpectrum(new_ep, refExp = "COADREAD")"
- [methods] custom gene sets: "Add parameter `geneSets = "MSigDBvx"` (x = 5/6/7) or `geneSets = MSigDBr` (MSigDBr attained from R package `msigdbr` with function get_msigdbr() in DeepCC)"
