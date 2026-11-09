---
name: gene-importance-ranking
description: Use when a trained Cancer-Finder checkpoint exists (e.g., ST model with
  5000-gene input) and the goal is to explain malignant-vs-normal spot classification
  by extracting a ranked, per-gene importance list from spatial transcriptomics slides.
when_to_use_negative:
- No trained Cancer-Finder model or checkpoint is available — the saliency module
  is gradient-based attribution through the trained feature-extraction and classification
  layers, not a standalone gene-ranking method.
- Hematologic tumor data — the article explicitly does not recommend Cancer-Finder
  for hematologic tumor data.
- Analyses requiring spatially aware feature attribution — the paper notes Cancer-Finder
  overlooks the spatial relationships among spots in ST data.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Cancer-Finder
  role: Provides the trained feature-extraction and classification network whose gradients
    drive the modified saliency-map gene-importance ranking; repo for the implementation
    (SequencingCancerFinder).
  repo: https://github.com/Patchouli-M/SequencingCancerFinder
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-46413-6
    title: Domain generalization enables general cancer cell annotation in single-cell
      and spatial transcriptomics
  source_content_hash: sha256:324c906502839e508a4582599c84e01dc4ada8beb11b5cd8b4903af20b8a9603
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Patchouli-M/SequencingCancerFinder
  tools:
  - Cancer-Finder
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# gene-importance-ranking

## Summary

Ranks genes by importance using Cancer-Finder's gradient-based modified saliency-map interpretability module, producing a top-ten gene signature (e.g., the ccRCC signature NDRG1, TAGLN, MALAT1, IGKC, IGHA1, IGHG4, IGLC2, IGHG3, SOD2, KRT19) from malignant-spot predictions on spatial transcriptomics slides.

## When to use

Use when a trained Cancer-Finder checkpoint exists (e.g., ST model with 5000-gene input) and the goal is to explain malignant-vs-normal spot classification by extracting a ranked, per-gene importance list from spatial transcriptomics slides.

## When NOT to use

- No trained Cancer-Finder model or checkpoint is available — the saliency module is gradient-based attribution through the trained feature-extraction and classification layers, not a standalone gene-ranking method.
- Hematologic tumor data — the article explicitly does not recommend Cancer-Finder for hematologic tumor data.
- Analyses requiring spatially aware feature attribution — the paper notes Cancer-Finder overlooks the spatial relationships among spots in ST data.

## Inputs

- Pre-trained Cancer-Finder ST checkpoint (feature-extraction network for 5000-gene spatial transcriptomics input)
- Spot expression matrices from 5 QC-passed ccRCC Visium ST slides (10 downloaded minus 3 technical duplicates and 2 QC-failed)
- Gene annotation sufficient to flag mitochondrial genes for filtering

## Outputs

- top10_saliency_genes.csv — ranked top-ten gene list by aggregated saliency importance (expected ccRCC result: NDRG1, TAGLN, MALAT1, IGKC, IGHA1, IGHG4, IGLC2, IGHG3, SOD2, KRT19)
- Per-gene importance scores aggregated across all spots of the analyzed slides

## How to apply

Load the pre-trained Cancer-Finder ST checkpoint (feature-extraction network sized for 5000 ST input genes) into the PyTorch model and load the spot expression matrices of the ccRCC Visium slides (5 QC-passed slides from 10 downloaded, after excluding 3 technical duplicates and 2 QC-failed slides), applying log-transformed global-scaling normalization. Implement the modified saliency-map module: gradient-based feature attribution computed through the trained feature-extraction and classification layers (per Supplementary Note 1). Run the module over all spots of the slides and aggregate per-gene importance scores across slides. Filter mitochondrial genes out of the interpretability results before ranking, then rank remaining genes by aggregated importance and write the top ten to top10_saliency_genes.csv. Correctness is judged by whether the ranked list reproduces the reported ten-gene ccRCC signature.

## Related tools

- **Cancer-Finder** (Provides the trained feature-extraction and classification network whose gradients drive the modified saliency-map gene-importance ranking; repo for the implementation (SequencingCancerFinder).) — https://github.com/Patchouli-M/SequencingCancerFinder

## Evaluation signals

- The ranked top-ten list must reproduce the ten ccRCC signature genes reported in the article: NDRG1, TAGLN, MALAT1, IGKC, IGHA1, IGHG4, IGLC2, IGHG3, SOD2, KRT19.
- Mitochondrial genes must be absent from the final ranked list (they are filtered before ranking).
- Slides QC: exactly 5 ccRCC Visium slides analyzed (10 downloaded minus 3 technical duplicates and 2 QC-failed slides).
- Per-gene importance scores are aggregated across all spots of the 5 slides before ranking, not computed per slide independently.
- Output file top10_saliency_genes.csv exists and contains exactly ten ranked genes.

## Limitations

- Cancer-Finder overlooks the spatial relationships among spots in ST data, a factor with the potential to enhance its overall efficacy — so saliency attributions are spot-wise, not spatially contextualized.
- The skill requires a pre-trained checkpoint; attributions are specific to the trained feature-extraction network (e.g., 5000 ST input genes) and do not transfer to other model sizes without retraining.
- Prediction quality depends on sequencing depth; the article notes errors resolvable only by increasing sequencing depth.

## Evidence

- [readme] interpretability module uses saliency map: "Cancer-Finder integrated an interpretability module, which can utilizes a saliency map to identified important genes related to prognosis and tumor microenvironment."
- [abstract] ten-gene signature from ccRCC ST slides: "identifies a gene signature consisting of 10 genes that are significantly co-localized and enriched at the tumor-normal interface"
- [intro] signature enriched at tumor-normal interface: "we successfully identify a gene signature consisting of ten genes that tends to be enriched at the interface between tumor and normal tissue"
