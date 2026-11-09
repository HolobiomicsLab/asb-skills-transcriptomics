---
name: tsne-dimensionality-reduction-of-cell-shape-latents
description: Use when single-cell shape features are already encoded as autoencoder
  latent vectors (e.g., MIRIAM pipeline output) and you need a 2D t-SNE embedding
  to test whether similar geometries cluster and whether marker-positive cells form
  discrete populations.
when_to_use_negative:
- Input is already a per-cell marker intensity table — the article's Figure 3A covers
  t-SNE on marker intensities directly, which is a distinct analysis from shape-latent
  t-SNE
- Shape features have not yet been encoded by the MIRIAM autoencoder (raw cell shape
  masks/outlines are the wrong input for this skill)
- The latent vectors being embedded were used to train the autoencoder — the article
  explicitly excludes autoencoder training cells from this analysis
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MIRIAM
  role: Source pipeline that performs cell segmentation, per-marker quantification,
    and autoencoder-based cell shape characterization, producing the shape latent
    vectors used as t-SNE input
  repo: github.com/Coffey-Lab/MIRIAM
- name: Rtsne
  role: R package used to run t-SNE on the shape latent vectors to produce the 2D
    embedding
- name: Python
  role: Implementation environment for the MIRIAM pipeline that generates the autoencoder
    latent vectors
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/cyto.a.24541
    title: 'MIRIAM: A machine and deep learning single‐cell segmentation and quantification
      pipeline for multi‐dimensional tissue images'
  source_content_hash: sha256:013f1d40803022ecfc2a4fb28be1679d09969c29bd8961039c4748f4417bf768
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/Coffey-Lab/MIRIAM
  tools:
  - MIRIAM
  - Python
  - Rtsne
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# tsne-dimensionality-reduction-of-cell-shape-latents

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run t-SNE (Rtsne package) on held-out autoencoder-encoded latent vectors for cell shape from the MIRIAM pipeline to produce a 2D embedding in which cells of similar geometry lie near each other; then color by per-cell marker intensities (e.g., p-ERK) to test whether shape alone reveals cellular populations.

## When to use

Use when single-cell shape features are already encoded as autoencoder latent vectors (e.g., MIRIAM pipeline output) and you need a 2D t-SNE embedding to test whether similar geometries cluster and whether marker-positive cells form discrete populations.

## When NOT to use

- Input is already a per-cell marker intensity table — the article's Figure 3A covers t-SNE on marker intensities directly, which is a distinct analysis from shape-latent t-SNE
- Shape features have not yet been encoded by the MIRIAM autoencoder (raw cell shape masks/outlines are the wrong input for this skill)
- The latent vectors being embedded were used to train the autoencoder — the article explicitly excludes autoencoder training cells from this analysis

## Inputs

- Held-out autoencoder-encoded latent vectors for cell shape (excluding latent vectors used to train the autoencoder) from MIRIAM pipeline output
- Per-cell marker intensity table (e.g., p-ERK, p-EGFR, PCNA, Ki67) aligned to the same cells as the latent vectors

## Outputs

- 2D t-SNE embedding of cells by shape similarity
- Annotated t-SNE embedding figure colored by per-cell marker intensities (e.g., tsne_shape_latent_figure3c.png)

## How to apply

Load the held-out autoencoder-encoded latent vectors for cell shape from the MIRIAM pipeline output, explicitly excluding latent vectors that were used to train the autoencoder, so the embedding is evaluated on unseen cells. Run t-SNE on these shape latent vectors with the Rtsne package to produce a 2D embedding of cells by shape similarity. Color the embedding by per-cell marker intensities (the same markers quantified per cell by MIRIAM) to confirm that cells with similar geometries are located near each other. Overlay p-ERK staining intensity and check whether p-ERK-high cells form a discrete cluster, then verify overlap with p-EGFR, PCNA, and Ki67 distributions as reported for Figure 3C. Critically, compare against the marker-intensity t-SNE (Fig. 3A) and note that in the source study the shape-based t-SNE revealed no discernible cellular populations or clear patterns in marker expression (Supplemental Figure 5); save the annotated embedding as an annotated figure (e.g., tsne_shape_latent_figure3c.png).

## Related tools

- **MIRIAM** (Source pipeline that performs cell segmentation, per-marker quantification, and autoencoder-based cell shape characterization, producing the shape latent vectors used as t-SNE input) — github.com/Coffey-Lab/MIRIAM
- **Rtsne** (R package used to run t-SNE on the shape latent vectors to produce the 2D embedding)
- **Python** (Implementation environment for the MIRIAM pipeline that generates the autoencoder latent vectors)

## Examples

```
Rscript -e 'library(Rtsne); emb <- Rtsne(shape_latents, dims=2, check_duplicates=FALSE); saveRDS(emb, "tsne_shape_latent.rds")'
```

## Evaluation signals

- Cells with similar geometries are located near each other in t-SNE space when shape latent vectors are the input (visual check of the embedding)
- Latent vectors used to train the autoencoder are confirmed excluded from the t-SNE input set
- Coloring the embedding by marker intensities shows whether marker patterns track shape clusters; in the source study this check failed — no discernible cellular populations or clear marker patterns emerged
- p-ERK-high cells are checked for a discrete cluster and its overlap with p-EGFR, PCNA, and Ki67 distributions as reported for Figure 3C
- Output figure is saved as a 2D annotated embedding (e.g., tsne_shape_latent_figure3c.png)

## Limitations

- In the source study, the t-SNE shape analysis did not reveal any discernible cellular populations or clear patterns in marker expression (Supplemental Figure 5), despite shape similarity being preserved in the embedding
- Latent vectors from cells used to train the autoencoder must be excluded; including them would inflate apparent clustering quality
- Any clustering structure found in shape space must be validated against marker intensity distributions (p-ERK/p-EGFR/PCNA/Ki67) rather than assumed to reflect biological populations

## Evidence
