---
name: unclassifiable-sample-counting-across-cohorts
description: Use when comparing subtype classifiers by how many samples they cannot
  assign (max posterior probability at or below a cutoff), e.g. DeepCC vs the random-forest
  CMS classifier on CRC cohorts.
when_to_use_negative:
- The goal is discovery of new subtypes or clustering of cancer patients — DeepCC
  is a supervised classifier and 'not clustering of cancer patients'
- Input gene expression already consists of functional spectra or deep features rather
  than raw expression profiles with Entrez identifiers
- You need subtype probabilities without a classifiability assessment — then use get_DeepCC_prob
  alone rather than applying the 0.5 unclassifiability flag
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
tools:
- name: DeepCC
  role: 'Supervised classification framework: computes functional spectra and posterior
    subtype probabilities; get_DeepCC_label applies the 0.5 cutoff to identify unclassifiable
    samples'
  repo: https://github.com/CityUHK-CompBio/DeepCC
- name: msigdbr
  role: Supplies MSigDBr gene sets (via get_msigdbr()) used as alternative input to
    getFunctionalSpectra for building functional spectra
  repo: https://github.com/CityUHK-CompBio/DeepCC
- name: keras
  role: Loads the pre-trained .hdf5 DeepCC model (keras::load_model_hdf5) restored
    together with class levels from the .RData file
  repo: https://keras.rstudio.com/
- name: R
  role: Runtime environment in which the DeepCC functions, model loading, and counting
    of unclassifiable samples are performed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41389-019-0157-8
    title: 'DeepCC: a novel deep learning-based framework for cancer molecular subtype
      classification'
  source_content_hash: sha256:531b41e3677c9d3001a513eb17d9fffc228c58965c6e52d948a627fc6a9ef433
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CityUHK-CompBio/DeepCC
  tools:
  - DeepCC
  - R
  - keras
  - msigdbr
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# unclassifiable-sample-counting-across-cohorts

## Summary

Quantifies the fraction of tumor samples left unclassified by DeepCC under a posterior-probability cutoff (0.5) across the CRCSC-analyzed data sets and the CIT/GSE39582 cohort, benchmarking against the 273-gene random-forest CMS classifier.

## When to use

Use when comparing subtype classifiers by how many samples they cannot assign (max posterior probability at or below a cutoff), e.g. DeepCC vs the random-forest CMS classifier on CRC cohorts.

## When NOT to use

- The goal is discovery of new subtypes or clustering of cancer patients — DeepCC is a supervised classifier and 'not clustering of cancer patients'
- Input gene expression already consists of functional spectra or deep features rather than raw expression profiles with Entrez identifiers
- You need subtype probabilities without a classifiability assessment — then use get_DeepCC_prob alone rather than applying the 0.5 unclassifiability flag

## Inputs

- Gene expression profiles (data.frame, rows = samples, columns = Entrez identifiers) for each CRCSC-analyzed data set
- CIT/GSE39582 CRC cohort expression profiles (n=557)
- Pre-trained CRCSC-trained CRC DeepCC model files (.RData with levels + .hdf5 Keras model, e.g. loaded as CRC_TCGA-style prefix)

## Outputs

- Per-data-set unclassifiable-sample-count table (Fig. 3a equivalent) written to unclassifiable_counts.csv
- Posterior subtype probabilities per sample (get_DeepCC_prob) and predicted labels (get_DeepCC_label, cutoff = 0.5)

## How to apply

Load the CRCSC-trained pre-trained CRC DeepCC model with load_DeepCC_model(), which restores the .RData levels and the .hdf5 Keras model via keras::load_model_hdf5. Transform gene expression profiles of each CRCSC data set and the CIT/GSE39582 cohort (n=557) into functional spectra using getFunctionalSpectra/getFunctionalSpectrum with the default MSigDB v7 gene sets (22,596 gene sets). Compute posterior subtype probabilities for every sample with get_DeepCC_prob() and assign labels with get_DeepCC_label() using cutoff = 0.5. Flag samples whose maximum posterior probability is at or below the 0.5 cutoff as unclassifiable and tally counts per data set. Compare DeepCC unclassifiable fractions against the random-forest CMS classifier result (117 unclassifiable on CIT), then write the per-data-set table (Fig. 3a equivalent) to unclassifiable_counts.csv.

## Related tools

- **DeepCC** (Supervised classification framework: computes functional spectra and posterior subtype probabilities; get_DeepCC_label applies the 0.5 cutoff to identify unclassifiable samples) — https://github.com/CityUHK-CompBio/DeepCC
- **msigdbr** (Supplies MSigDBr gene sets (via get_msigdbr()) used as alternative input to getFunctionalSpectra for building functional spectra) — https://github.com/CityUHK-CompBio/DeepCC
- **keras** (Loads the pre-trained .hdf5 DeepCC model (keras::load_model_hdf5) restored together with class levels from the .RData file) — https://keras.rstudio.com/
- **R** (Runtime environment in which the DeepCC functions, model loading, and counting of unclassifiable samples are performed)

## Examples

```
CRC_model <- load_DeepCC_model("CRC_TCGA"); probs <- get_DeepCC_prob(CRC_model, fs); labels <- get_DeepCC_label(CRC_model, fs, cutoff = 0.5)
```

## Evaluation signals

- On CIT/GSE39582 (n=557), DeepCC must classify 531 samples (557-531=26 unclassified), matching the reported figure
- Across all CRCSC-analyzed data sets, the unclassifiable fraction under cutoff=0.5 must be approximately 5% of tumor samples
- The comparison baseline must show 117 unclassifiable samples for the 273-gene random-forest CMS classifier on CIT, which DeepCC should substantially undercut
- Every sample flagged unclassifiable must have maximum posterior probability at or below the 0.5 cutoff from get_DeepCC_prob
- unclassifiable_counts.csv must contain one row per analyzed data set with counts consistent with the per-data-set tallies

## Limitations

- The posterior probability > 0.5 criterion is arbitrary; samples at or below it are discarded rather than assigned a subtype, so results depend on the chosen cutoff
- DeepCC requires a comprehensive gene set database to avoid potential bias — 'the prior knowledge of gene sets needs to be comprehensive to avoid potential bias'
- DeepCC is 'specifically developed for addressing current challenges in classification but not clustering of cancer patients', so unclassifiability here reflects supervised classification only
- For single-sample (non-batch) microarray input, functional spectra must be computed against a reference expression profile (e.g. refExp = 'COADREAD'), and inverseRescale is strongly suggested

## Evidence

- [results] DeepCC failed to classify only ~5% of tumor samples under the posterior probability > 0.5 criterion: [evidence span withheld — anchor status: not_found]
- [methods] get_DeepCC_label cutoff: "get_DeepCC_label(DeepCCModel, newData, cutoff, prob_mode, prob_raw)"
- [methods] get_DeepCC_prob signature: "get_DeepCC_prob(DeepCCModel, newData)"
- [readme] load_DeepCC_model definition: "load_DeepCC_model <- function(prefix){
  load(file = paste0(prefix, ".RData"))
  classifer <- keras::load_model_hdf5(filepath =paste0(prefix, ".hdf5"))"
- [abstract] reduced unclassifiable samples finding: "deep features learned by DeepCC captured biological characteristics associated with distinct molecular subtypes, enabling more compact within-subtype distribution and between-subtype separation of"
