---
name: genomic-track-alignment-and-integration
description: Use when one or few DNase-seq (or ChIP-seq) replicates are available
  and a BIRD-predicted DH track can be merged with observed signal on shared genomic
  bins, then benchmarked against a held-out truth replicate via genome-wide Pearson
  correlation.
when_to_use_negative:
- The goal is to discover novel DHS locations — BIRD will not discover new locations
  of DHSs, it only predicts activity at ~1M pre-defined genomic loci
- No held-out replicate is available to validate the combined profile — the acceptance
  criterion (combined > observed-only Pearson correlation) cannot be evaluated
- Observed tracks are on a different genome build than the model (e.g. hg38 DNase-seq
  data with the hg19 model) without lifting coordinates to the model's bin frame
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: BIRD_predict
  role: Generates the predicted DH track from the FPKM matrix using a prebuilt model;
    outputs a log2(x+1) matrix bounded 0–14 that is combined with observed DNase-seq
    signal
  repo: https://github.com/WeiqiangZhou/BIRD
- name: BIRD
  role: Software package providing the prediction engine and models used to create
    the pseudo-replicate component of the integrated profile
  repo: https://github.com/WeiqiangZhou/BIRD
- name: R
  role: Used to align predicted and observed tracks on shared hg19 bin coordinates,
    build the combined profile, and compute genome-wide Pearson correlations against
    the truth replicate
  repo: https://www.r-project.org
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-017-01188-x
    title: Genome-wide prediction of DNase I hypersensitivity using gene expression
  source_content_hash: sha256:48f1a9b25bbc0a0b182b43edc3311ef753c721e6da38f180b2f18b8e93210fa4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WeiqiangZhou/BIRD
  tools:
  - BIRD
  - BIRD_predict
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# genomic-track-alignment-and-integration

## Summary

Aligns BIRD-predicted DNase I hypersensitivity (DH) tracks with observed ENCODE DNase-seq replicates on the hg19 genomic bin coordinates used by the model, then combines them into a BIRD-augmented pseudo-replicate profile to boost signal in low replicate or low signal-to-noise settings.

## When to use

Use when one or few DNase-seq (or ChIP-seq) replicates are available and a BIRD-predicted DH track can be merged with observed signal on shared genomic bins, then benchmarked against a held-out truth replicate via genome-wide Pearson correlation.

## When NOT to use

- The goal is to discover novel DHS locations — BIRD will not discover new locations of DHSs, it only predicts activity at ~1M pre-defined genomic loci
- No held-out replicate is available to validate the combined profile — the acceptance criterion (combined > observed-only Pearson correlation) cannot be evaluated
- Observed tracks are on a different genome build than the model (e.g. hg38 DNase-seq data with the hg19 model) without lifting coordinates to the model's bin frame

## Inputs

- RNA-seq FPKM data matrix with gene ensembl ids in the first column (e.g. GM12878 FPKM_data_matrix.txt)
- Prebuilt BIRD model file (.bin), e.g. human_hg19_model.bin or human_hg38_model.bin
- Observed ENCODE DNase-seq replicate signal tracks (one held out as truth, one as observed-only)
- Genomic bin coordinate frame used by the model (hg19, 200bp windows, ~1M loci)

## Outputs

- BIRD-predicted DH track in log2(x+1) scale bounded at 0–14 (data matrix format, or optional WIG output)
- Combined observed+BIRD pseudo-replicate profile aligned to hg19 bins
- Genome-wide Pearson correlation values for observed-only, prediction-only, and combined profiles versus the held-out truth replicate
- Summary table of correlations and combined-minus-observed deltas (dh_prediction_correlation_summary.csv)

## How to apply

Run BIRD_predict with the prebuilt model (e.g. human_hg19_model.bin or human_hg38_model.bin) on the RNA-seq FPKM matrix to obtain genome-wide predicted DH values in log2(x+1) scale, bounded 0–14 by default (-u 14). Designate one ENCODE DNase-seq replicate as the held-out truth track and keep the other replicate as the observed-only component. In R, combine the BIRD-predicted DH track with the observed replicate track, aligning all tracks on the same hg19 genomic bin (200bp window) coordinates so every candidate profile is a vector of identical length. Compute genome-wide Pearson correlations between the truth replicate and each candidate: observed replicate alone, BIRD prediction alone, and the combined observed+BIRD profile. Accept the integration only if the combined profile's correlation with the held-out replicate exceeds the observed-only baseline (paper reports ~0.82 vs ~0.76, with improvement in 12/16 test cell types and 9/12 ChIP-seq benchmark cases). Write a summary table of per-profile correlations and the combined-minus-observed delta (e.g. dh_prediction_correlation_summary.csv).

## Related tools

- **BIRD_predict** (Generates the predicted DH track from the FPKM matrix using a prebuilt model; outputs a log2(x+1) matrix bounded 0–14 that is combined with observed DNase-seq signal) — https://github.com/WeiqiangZhou/BIRD
- **BIRD** (Software package providing the prediction engine and models used to create the pseudo-replicate component of the integrated profile) — https://github.com/WeiqiangZhou/BIRD
- **R** (Used to align predicted and observed tracks on shared hg19 bin coordinates, build the combined profile, and compute genome-wide Pearson correlations against the truth replicate) — https://www.r-project.org

## Examples

```
path_to_BIRD/BIRD_predict -b path_to_model/human_hg19_model.bin -i FPKM_data_matrix.txt -o output_file.txt
```

## Evaluation signals

- Combined observed+BIRD profile's Pearson correlation with the held-out DNase-seq truth replicate exceeds the observed-only baseline (~0.82 vs ~0.76)
- Improvement reproduced across benchmark breadth: gain in 12/16 DNase-seq test cell types and 9/12 ChIP-seq cases
- All candidate profiles (observed-only, prediction-only, combined) have identical length and bin ordering after alignment on the hg19 200bp window coordinates
- Predicted values fall in log2(x+1) scale with the default upper bound of 14 (no values >14 or <0)
- The matched input matrix (input_file.match.txt) confirms ensembl gene ids were matched to the model before prediction, ensuring the predicted track corresponds to the sample's transcriptome

## Limitations

- BIRD will not discover new locations of DHSs when applied to analyze a new gene expression sample — integration is restricted to the ~1M pre-defined genomic loci
- Genomic information in the current version of BIRD is based on human genome hg19; newer hg38 models exist but tracks must be on the matching build
- Improvement is not universal — the paper reports gains in 12 of 16 test cell types, so some cell types show no benefit over observed data alone

## Evidence

- [methods] Genomic information in the current version of BIRD is based on human genome hg19.: "Genomic information in the current version of BIRD is based on **human genome hg19**."
- [readme] Standard output will save a matrix contained all predited value in log scale (log2(x+1) transformed).: "Standard output will save a matrix contained all predited value in log scale (log2(x+1) transformed)."
- [abstract] We show applications of BIRD-predicted DH in predicting transcription factor-binding sites (TFBSs): "We show applications of BIRD-predicted DH in predicting transcription factor-binding sites (TFBSs)"
- [discussion] one limitation of BIRD is that it will not discover new locations of DHSs when applied to analyze a new gene expression sample: "one limitation of BIRD is that it will not discover new locations of DHSs when applied to analyze a new gene expression sample"
