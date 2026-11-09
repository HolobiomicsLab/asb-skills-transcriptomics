---
name: saliency-map-gene-attribution
description: Use when a trained Cancer-Finder ST or scRNA checkpoint must be interrogated
  for which genes drive malignant-vs-non-malignant predictions, e.g. to derive a ranked
  tumor-interface gene signature like the ccRCC 10-gene list.
when_to_use_negative:
- Data are hematologic tumor samples — the article explicitly does not recommend Cancer-Finder
  for hematologic tumor data
- A reference-based CNV deconvolution or cell-composition analysis (cell2location,
  CARD, inferCNV, CopyKAT) is the goal, not gradient-based gene attribution from a
  trained Cancer-Finder model
- No pre-trained Cancer-Finder checkpoint or matched training data exist; the saliency
  map requires gradient access to trained feature-extraction and classification layers
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3474
tools:
- name: Cancer-Finder
  role: Trained domain-generalization deep-learning model whose feature-extraction
    and classification layers are back-propagated through for gradient-based (saliency)
    gene importance scoring
  repo: https://github.com/Patchouli-M/SequencingCancerFinder
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-46413-6
    title: Domain generalization enables general cancer cell annotation in single-cell
      and spatial transcriptomics
  source_content_hash: sha256:9ecccc6fb7635332047db34b8bd43578bf3d626581e3bd25c08de35e57233dab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Patchouli-M/SequencingCancerFinder
  tools:
  - Cancer-Finder
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# saliency-map-gene-attribution

## Summary

Gradient-based gene attribution from Cancer-Finder's modified saliency-map interpretability module, used to rank feature importance through the trained feature-extraction and classification layers and extract a top-ten malignant-vs-non-malignant gene signature from ccRCC spatial transcriptomics slides.

## When to use

Use when a trained Cancer-Finder ST or scRNA checkpoint must be interrogated for which genes drive malignant-vs-non-malignant predictions, e.g. to derive a ranked tumor-interface gene signature like the ccRCC 10-gene list.

## When NOT to use

- Data are hematologic tumor samples — the article explicitly does not recommend Cancer-Finder for hematologic tumor data
- A reference-based CNV deconvolution or cell-composition analysis (cell2location, CARD, inferCNV, CopyKAT) is the goal, not gradient-based gene attribution from a trained Cancer-Finder model
- No pre-trained Cancer-Finder checkpoint or matched training data exist; the saliency map requires gradient access to trained feature-extraction and classification layers

## Inputs

- Pre-trained Cancer-Finder ST checkpoint (feature-extraction network for 5000 ST input genes)
- Spot-level expression matrices of 5 ccRCC Visium slides (log-transformed, global-scaling normalized)

## Outputs

- top10_saliency_genes.csv — ranked top-ten per-gene saliency importance list (NDRG1, TAGLN, MALAT1, IGKC, IGHA1, IGHG4, IGLC2, IGHG3, SOD2, KRT19)
- Per-gene aggregated importance scores across slides

## How to apply

Load the pre-trained Cancer-Finder checkpoint (ST feature-extraction network sized for 5000 ST input genes) into the PyTorch model. Load the spot expression matrices (e.g. 5 ccRCC Visium slides selected from 10 downloaded after excluding 3 technical duplicates and 2 QC-failed slides) and apply log-transformed global-scaling normalization. Implement the modified saliency-map module per Supplementary Note 1: gradient-based feature attribution flowing through the trained feature-extraction and classification layers. Run the module over all spots of each slide and aggregate per-gene importance scores across slides. Filter mitochondrial genes out of the attribution results before ranking, then rank remaining genes by aggregated importance and write the top ten (here: NDRG1, TAGLN, MALAT1, IGKC, IGHA1, IGHG4, IGLC2, IGHG3, SOD2, KRT19) to top10_saliency_genes.csv.

## Related tools

- **Cancer-Finder** (Trained domain-generalization deep-learning model whose feature-extraction and classification layers are back-propagated through for gradient-based (saliency) gene importance scoring) — https://github.com/Patchouli-M/SequencingCancerFinder

## Evaluation signals

- The ranked top-ten gene list reproduces the published ccRCC signature: NDRG1, TAGLN, MALAT1, IGKC, IGHA1, IGHG4, IGLC2, IGHG3, SOD2, KRT19
- top10_saliency_genes.csv contains exactly ten genes after mitochondrial-gene filtering
- Input matrices were normalized with log-transformed global scaling and match the 5000-gene ST input dimension of the checkpoint
- Attribution was aggregated across all spots of the 5 slides (10 downloaded minus 3 technical duplicates minus 2 QC-failed)

## Limitations

- Cancer-Finder overlooks the spatial relationships among spots in ST data
- Cancer-Finder is not recommended for hematologic tumor data
- Cancer-Finder is robust only to a certain percentage of mislabeling in the training set
- Prediction errors may require increased sequencing depth to resolve

## Evidence

- [readme] Additionally, Cancer-Finder integrated an interpretability module, which can utilizes a saliency map to identified important genes related to prognosis and tumor microenvironment.: "Cancer-Finder integrated an interpretability module, which can utilizes a saliency map to identified important genes related to prognosis and tumor microenvironment."
- [intro] By precisely identifying malignant spots on 5 ccRCC ST slides, we successfully identify a gene signature consisting of ten genes: "By precisely identifying malignant spots on 5 ccRCC ST slides, we successfully identify a gene signature consisting of ten genes"
- [abstract] identifies a gene signature consisting of 10 genes that are significantly co-localized and enriched at the tumor-normal interface and have a strong correlation with the prognosis of clear cell renal: "identifies a gene signature consisting of 10 genes that are significantly co-localized and enriched at the tumor-normal interface and have a strong correlation with the prognosis of clear cell renal"
