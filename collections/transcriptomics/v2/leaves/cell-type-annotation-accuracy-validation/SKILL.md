---
name: cell-type-annotation-accuracy-validation
description: Use when GeneVector phenotype_probability annotations must be trusted
  before downstream analyses like latent-space arithmetic; trigger when an independent
  label source (e.g., CellAssign) exists for concordance checking.
when_to_use_negative:
- No independent label source (e.g., CellAssign output) is available for concordance
  comparison — validation would be circular
- Cell type composition differs drastically between batches, making batch correction
  vectors unreliable and annotations unverifiable against known signatures
- The goal is metagene discovery or gene similarity ranking rather than assigning
  cell identity to individual cells
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: GeneVector
  role: Provides CellEmbedding and phenotype_probability for marker-based cell type
    assignment via latent space arithmetic
  repo: https://github.com/nceglia/genevector
- name: Scanpy
  role: Loads and manipulates the AnnData objects with raw counts in .X used for annotation
    and downstream saving
- name: AnnData
  role: Stores annotation pseudo-probabilities and validated labels in .obs for export
    to h5ad
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-39985-2
    title: Identification of transcriptional programs using dense vector representations
      defined by mutual information with GeneVector
  source_content_hash: sha256:9ab9e5caeac8e4e346c033c37bb635d332304f7918bddf16d03ade8885a1d880
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nceglia/genevector
  tools:
  - AnnData
  - GeneVector
  - NumPy
  - PyTorch
  - Python
  - Scanpy
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# cell-type-annotation-accuracy-validation

## Summary

Validates GeneVector latent-space cell type annotations (via CellEmbedding.phenotype_probability with marker gene sets) against independent CellAssign labels, confirming annotation reliability before downstream vector-arithmetic analyses such as the HGSOC adnexa-vs-bowel metagene comparison.

## When to use

Use when GeneVector phenotype_probability annotations must be trusted before downstream analyses like latent-space arithmetic; trigger when an independent label source (e.g., CellAssign) exists for concordance checking.

## When NOT to use

- No independent label source (e.g., CellAssign output) is available for concordance comparison — validation would be circular
- Cell type composition differs drastically between batches, making batch correction vectors unreliable and annotations unverifiable against known signatures
- The goal is metagene discovery or gene similarity ranking rather than assigning cell identity to individual cells

## Inputs

- Scanpy AnnData object with raw counts in .X (e.g., HGSOC SPECTRUM adnexa/bowel samples)
- GeneEmbedding loaded from genes.vec (vector='average')
- CellEmbedding derived from GeneVectorDataset and GeneEmbedding
- Marker gene dictionary for target cell types (six HGSOC cell types plus EPCAM, CD24 for cancer cells)
- Independent reference labels (e.g., CellAssign cell type annotations in adata.obs)

## Outputs

- AnnData with per-cell-type pseudo-probability columns ('Pseudo-probability' in obs) and GeneVector label column
- Concordance metrics per cell type and overall (e.g., 99.7% in three cell types, ≥94.1% overall agreement vs CellAssign)
- Validated annotated AnnData saved to hgsoc_cell_annotations.h5ad

## How to apply

After training the GeneVector model (emb_dimension=100, 1000 epochs, threshold=1e-6) and building a GeneEmbedding with vector='average', construct a CellEmbedding and run cell_embed.phenotype_probability(adata, markers) using marker gene sets — for the HGSOC SPECTRUM data, six cell types plus EPCAM and CD24 for cancer cells. Independently, obtain cell type labels from a complementary method such as CellAssign. Measure concordance between the two label sets per cell type and overall; in the HGSOC workflow this achieved 99.7% agreement in three cell types and a minimum of 94.1% overall. Only accept annotations meeting these thresholds before computing directional vectors (e.g., Vadnexa − Vbowel restricted to cancer cells), since misannotated cells would contaminate the arithmetic.

## Related tools

- **GeneVector** (Provides CellEmbedding and phenotype_probability for marker-based cell type assignment via latent space arithmetic) — https://github.com/nceglia/genevector
- **Scanpy** (Loads and manipulates the AnnData objects with raw counts in .X used for annotation and downstream saving)
- **AnnData** (Stores annotation pseudo-probabilities and validated labels in .obs for export to h5ad)

## Examples

```
adata = cell_embed.phenotype_probability(adata_embedded, markers); concordance = (adata.obs['genevector'] == adata.obs['cellassign']).mean()
```

## Evaluation signals

- Per-cell-type concordance between GeneVector phenotype_probability labels and CellAssign labels reaches ≥99.7% in at least three cell types
- Overall annotation agreement between the two methods is ≥94.1%
- Output AnnData obs contains 'Pseudo-probability' columns for every marker set and a 'genevector' label column as in the README usage example
- Cancer cell annotation is supported by positive EPCAM and CD24 pseudo-probability in cancer-cell-restricted vector arithmetic (Vadnexa − Vbowel)
- The validated AnnData saves without error to hgsoc_cell_annotations.h5ad

## Limitations

- Batch correction is unreliable when batches differ in cell type composition, so annotation agreement must be manually verified against cell type signatures
- Concordance with an external method (CellAssign) validates only consistency, not ground truth; both could share systematic errors
- Benchmarking metrics referenced only as Supplementary Fig. 6A–L, a noted reproducibility gap for broader accuracy claims

## Evidence

- [methods] validate annotations by concordance with CellAssign labels: [evidence span withheld — anchor status: not_found]
- [abstract] GeneVector latent space arithmetic cell type classification: "GeneVector uses latent space arithmetic in a lower dimensional gene embedding to identify transcriptional programs and classify cell types."
- [readme] phenotype_probability API: "# Perform automated cell type assignment
annotated_adata = cell_embed.phenotype_probability(adata_embedded, markers)"
- [intro] TICA cell type validation objective: "We demonstrate that latent space arithmetic can be used to accurately label cell types in the TICA dataset and validate our cell type predictions against published annotations"
