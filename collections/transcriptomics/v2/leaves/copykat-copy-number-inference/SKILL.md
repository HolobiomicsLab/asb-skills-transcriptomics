---
name: copykat-copy-number-inference
description: Use when a tumor-derived scRNA-seq dataset (e.g. the AML patient sample)
  must be split into diploid non-malignant vs aneuploid malignant cells by inferred
  copy-number profiles, validating SNV-score malignancy calls.
when_to_use_negative:
- Normal (non-tumor) scRNA-seq data where no aneuploid malignant population is expected,
  so a diploid/aneuploid split is uninformative
- Datasets where the goal is cell-type annotation only, which ScType's marker-based
  sctype_score alone handles without CNV inference
- Input is already a per-cell malignancy label table or SNV score table rather than
  a raw expression matrix
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: CopyKAT
  role: Infers copy-number profiles from the expression matrix and classifies cells
    as diploid (non-malignant) or aneuploid (malignant)
- name: Seurat
  role: Preprocessing of the scRNA-seq sample (normalize, scale, cluster) before copy-number
    inference
- name: vartrix
  role: Links called SNVs to individual cell barcodes/clusters so CopyKAT diploid/aneuploid
    calls can be cross-validated against ScType SNV scores
  repo: https://github.com/10XGenomics/vartrix
- name: ScType
  role: Provides cluster annotations and the single-cell SNV score whose malignancy
    calls are validated by CopyKAT diploid classification
  repo: https://github.com/IanevskiAleksandr/sc-type
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-28803-w
    title: Fully-automated and ultra-fast cell-type identification using specific
      marker combinations from single-cell transcriptomic data
  source_content_hash: sha256:96444ad7e6fb977692c83a4ce770caa3e8f56d8f4179f6599097bc4e4609678a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/10XGenomics/vartrix
  tools:
  - CopyKAT
  - R
  - ScType
  - Seurat
  - Strelka2
  - samtools
  - vartrix
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# copykat-copy-number-inference

## Summary

Infer copy-number profiles from a single-cell RNA-seq expression matrix with CopyKAT to classify cells as diploid (non-malignant) or aneuploid (malignant), complementing ScType's SNV-score-based malignancy calling.

## When to use

Use when a tumor-derived scRNA-seq dataset (e.g. the AML patient sample) must be split into diploid non-malignant vs aneuploid malignant cells by inferred copy-number profiles, validating SNV-score malignancy calls.

## When NOT to use

- Normal (non-tumor) scRNA-seq data where no aneuploid malignant population is expected, so a diploid/aneuploid split is uninformative
- Datasets where the goal is cell-type annotation only, which ScType's marker-based sctype_score alone handles without CNV inference
- Input is already a per-cell malignancy label table or SNV score table rather than a raw expression matrix

## Inputs

- Normalized single-cell expression matrix (e.g. Seurat-processed counts from an AML patient scRNA-seq sample)
- Cell barcode-to-cluster annotations from ScType/Seurat clustering
- Optionally, ScType per-cluster SNV scores derived from vartrix genome matrices for cross-validation

## Outputs

- Per-cell copy-number profile matrix
- Binary cell classification: diploid (non-malignant) vs aneuploid (malignant)
- Cluster/cell-type level malignant vs non-malignant labels for chord-diagram visualization

## How to apply

After preprocessing the scRNA-seq sample with Seurat (normalize, scale, cluster) and annotating clusters with ScType's sctype_score, run CopyKAT on the normalized expression matrix to infer per-cell copy-number profiles and label each cell as diploid (non-malignant) or aneuploid (malignant). Use the resulting diploid/aneuploid calls as an orthogonal check on the malignancy calls obtained from single-cell SNV calling (samtools mpileup and Strelka2 restricted to COSMIC cancer consensus genes, linked to barcodes via vartrix). In the AML case, the expected outcome is a diploid majority for the non-malignant populations (CD24+ CD66+ neutrophils and memory CD8+ T cells, which also carry ScType SNV score <20) and aneuploid calls for CD34+ HSC/MPP progenitors and CD34+ ISG+ blasts. The combined labels feed downstream visualization, e.g. a chord diagram separating non-malignant from malignant populations.

## Related tools

- **CopyKAT** (Infers copy-number profiles from the expression matrix and classifies cells as diploid (non-malignant) or aneuploid (malignant))
- **Seurat** (Preprocessing of the scRNA-seq sample (normalize, scale, cluster) before copy-number inference)
- **vartrix** (Links called SNVs to individual cell barcodes/clusters so CopyKAT diploid/aneuploid calls can be cross-validated against ScType SNV scores) — https://github.com/10XGenomics/vartrix
- **ScType** (Provides cluster annotations and the single-cell SNV score whose malignancy calls are validated by CopyKAT diploid classification) — https://github.com/IanevskiAleksandr/sc-type

## Evaluation signals

- Non-malignant populations (CD2+CD66+ neutrophils and memory CD8+ T cells) show a CopyKAT diploid majority
- Malignant populations (CD34+ HSC/MPP and CD34+ ISG+ blasts) are classified aneuploid by CopyKAT
- CopyKAT diploid/aneuploid calls agree with ScType SNV scores (<20 for non-malignant neutrophils and memory CD8+ T cells)
- Chord diagram shows a clean separation between non-malignant and malignant populations using the CopyKAT labels
- All cells in the input matrix receive a defined copy-number-based class (no missing/NaN labels)

## Limitations

- Variant calling from transcriptome data has a high false-positive rate due to reverse-transcription noise; CopyKAT CNV calls are best used alongside orthogonal evidence such as SNV scores
- CopyKAT infers copy number from expression, which is indirect; results depend on quality of Seurat normalization/clustering of the same matrix
- Distinguishing multiple malignant from non-malignant populations for targeted treatments 'remains a translational challenge and requires both systematic and highly selective' approaches (article intro)

## Evidence

- [other] CopyKAT diploid majority for non-malignant cells: [evidence span withheld — anchor status: not_found]
- [intro] ScType SNV-based malignancy distinction validated by CopyKAT: "ScType implements a single-cell single-nucleotide variant (SNV) calling option to distinguish between malignant and non-malignant cells (Fig. 1a)"
- [abstract] Single-cell SNV calling for healthy vs malignant separation: "based on single-cell calling of single-nucleotide variants"
