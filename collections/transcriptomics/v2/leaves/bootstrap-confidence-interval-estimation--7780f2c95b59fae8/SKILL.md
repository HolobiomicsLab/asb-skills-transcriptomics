---
name: bootstrap-confidence-interval-estimation
description: Use when estimating inter- vs intra-cell splicing heterogeneity (phi)
  from single-cell Nanopore isoform counts and you need to distinguish statistically
  confident phi estimates from unstable ones before plotting or interpretation.
when_to_use_negative:
- When the input is a bulk (non-single-cell) isoform table — the beta-binomial phi
  metric requires per-cell counts to model inter- vs intra-cell variation.
- When genes are not highly expressed — sites failing the gene count >= 10 in >30
  cells and mean psi 0.1–0.9 filters yield unreliable bootstrap intervals and should
  not be kept.
- For intra-cell-only heterogeneity measures or differential alternative splicing
  testing, which use different Longcell functionality.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Longcell
  role: Provides genesSitesPhi with method = "beta" (mle_bb maximum likelihood fitting
    of the beta-binomial model) and bootstrap re-estimation of alpha/beta for CI(phi);
    phiPlot renders the filtered phi-versus-psi scatter
  repo: github.com/yuntianf/Longcell
- name: LongcellPre
  role: Supplies the gene strand annotation (gene_bed.rds) needed to build meta-splicing
    sites before phi estimation
- name: dplyr
  role: Used in the vignette workflow for filtering/loading the isoform count data
- name: future
  role: Loaded for parallelized computation in the phi estimation workflow
- name: future.apply
  role: Loaded for parallelized computation in the phi estimation workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-60902-2
    title: Single cell and spatial alternative splicing analysis with Nanopore long
      read sequencing
  source_content_hash: sha256:625a5486de8dbe0de18953a32c87595c748472e9cd0765105725be6e72ab8636
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/yuntianf/Longcell
  tools:
  - Longcell
  - LongcellPre
  - Seurat
  - dplyr
  - future
  - future.apply
  - ggplot2
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# bootstrap-confidence-interval-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantify uncertainty in Beta-Binomial splicing heterogeneity estimates by re-estimating alpha and beta on resampled data in Longcell's genesSitesPhi workflow, keeping only meta-splicing sites whose phi confidence interval is narrow enough (CI(phi) < 0.2) to be called confident.

## When to use

Use when estimating inter- vs intra-cell splicing heterogeneity (phi) from single-cell Nanopore isoform counts and you need to distinguish statistically confident phi estimates from unstable ones before plotting or interpretation.

## When NOT to use

- When the input is a bulk (non-single-cell) isoform table — the beta-binomial phi metric requires per-cell counts to model inter- vs intra-cell variation.
- When genes are not highly expressed — sites failing the gene count >= 10 in >30 cells and mean psi 0.1–0.9 filters yield unreliable bootstrap intervals and should not be kept.
- For intra-cell-only heterogeneity measures or differential alternative splicing testing, which use different Longcell functionality.

## Inputs

- Splice object with merged meta-splicing sites (from creatSplice_from_df, HighExprsGene thresh=500, geneSiteTable eps=0.05)
- Gene strand annotation (gene_bed.rds from LongcellPre)
- Single-cell isoform count table (sc_iso_count.rds)

## Outputs

- Phi estimation table with alpha, beta, phi = 1/(alpha+beta+1), and bootstrap CI(phi) per meta-splice site (written to CSV)
- Filtered set of confident phi estimates (CI(phi) < 0.2) suitable for phi-versus-psi scatter plotting via phiPlot

## How to apply

After building a Splice object (creatSplice_from_df), restricting to highly expressed genes with HighExprsGene (thresh = 500, yielding ~752 genes in the CRCLM vignette), and merging sites with geneSiteTable (eps = 0.05), call genesSitesPhi with method = "beta". This fits the beta-binomial model by maximum likelihood via mle_bb and computes phi = 1/(alpha+beta+1) per meta-splice site; confidence intervals are obtained by re-estimating alpha and beta via bootstrap resampling. Retain only meta-sites with gene count >= 10 in over 30 cells, mean psi between 0.1 and 0.9, and bootstrap CI width CI(phi) < 0.2. Rationale: the CI filter removes low-coverage or extreme-psi sites where phi is unstable, so the surviving confident estimates support downstream claims (e.g., the CRCLM result of 88 valid meta-splicing sites in 57 genes dominated by low inter-cell heterogeneity).

## Related tools

- **Longcell** (Provides genesSitesPhi with method = "beta" (mle_bb maximum likelihood fitting of the beta-binomial model) and bootstrap re-estimation of alpha/beta for CI(phi); phiPlot renders the filtered phi-versus-psi scatter) — github.com/yuntianf/Longcell
- **LongcellPre** (Supplies the gene strand annotation (gene_bed.rds) needed to build meta-splicing sites before phi estimation)
- **dplyr** (Used in the vignette workflow for filtering/loading the isoform count data)
- **future** (Loaded for parallelized computation in the phi estimation workflow)
- **future.apply** (Loaded for parallelized computation in the phi estimation workflow)

## Examples

```
crc_phi = genesSitesPhi(crc_splice, genes = heg, method = "beta", verbose = FALSE)
```

## Evaluation signals

- Every retained meta-splice site has bootstrap CI(phi) width < 0.2, mean psi in [0.1, 0.9], and gene count >= 10 in more than 30 cells.
- Running on the CRCLM highly expressed gene set (thresh = 500, ~752 genes) should yield a small set of confident phi estimates (~88 sites in 57 genes), dominated by low-phi sites.
- Each phi value equals 1/(alpha+beta+1) for the mle_bb-fitted beta-binomial parameters of that meta-site.
- The filtered phi table should render cleanly with phiPlot(annot_col = "gene") as a phi-versus-psi scatter, with most points at low phi and psi strictly between 0.1 and 0.9.
- Replicate bootstrap runs should produce consistent confident-site membership; sites near the CI boundary (width ~0.2) flag instability.

## Limitations

- Only meta-splicing sites in highly expressed genes (HighExprsGene thresh = 500) have sufficient coverage for stable bootstrap intervals; low-expression sites are dropped by the coverage and CI filters.
- Sites with extreme psi (outside 0.1–0.9) are excluded, so phi cannot be assessed for near-constitutive or very rare splicing events.
- Bootstrap CI validity depends on the beta-binomial (mle_bb) fit; sequencing errors, read truncation, and mapping errors in Nanopore reads upstream can bias the underlying counts.

## Evidence

- [methods] keep only meta-sites with gene count >= 10 in over 30 cells, mean psi between 0.1 and 0.9, and bootstrap confidence interval width CI(phi) < 0.2, obtained by re-estimating alpha and beta via bootstrap: [evidence span withheld — anchor status: not_found]
- [methods] fitting the beta-binomial model by maximum likelihood (mle_bb) and computing phi = 1/(alpha+beta+1) for each meta-splice site: [evidence span withheld — anchor status: not_found]
- [methods] crc_phi = genesSitesPhi(crc_splice,genes = heg,method = "beta",verbose = FALSE): "crc_phi = genesSitesPhi(crc_splice,genes = heg,method = "beta",verbose = FALSE)"
- [abstract] Longcell efficiently recovers cell barcodes and UMIs, corrects sequencing errors, and models splicing diversity within and between cells or spots: "Longcell efficiently recovers cell barcodes and UMIs, corrects sequencing errors, and models splicing diversity within and between cells or spots."
- [discussion] we proposed a novel metric to evaluate the exon- and splice-site usage heterogeneity within versus across single cells: "we proposed a novel metric to evaluate the exon- and splice-site usage heterogeneity within versus across single cells"
