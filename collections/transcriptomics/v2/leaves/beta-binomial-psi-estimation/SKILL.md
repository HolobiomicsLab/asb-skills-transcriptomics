---
name: beta-binomial-psi-estimation
description: Use when you have a cell-by-isoform count matrix from single-cell Nanopore
  long reads and need to estimate inter-cell splicing heterogeneity (phi) versus psi
  at exon/splice sites, distinguishing isoform co-expression within cells.
when_to_use_negative:
- Input is raw Nanopore FASTQ without UMI-corrected isoform quantification — run LongcellPre/isoform
  quantification first
- Genes with very low expression that fail the HighExprsGene threshold of 500 — phi
  estimates will not pass the >=10 gene-count-in-30-cells filter
- Bulk or tissue-level splicing analysis where cell-level (inter vs intra) heterogeneity
  decomposition is not the goal
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: Longcell
  role: Provides creatSplice_from_df, HighExprsGene, geneSiteTable, genesSitesPhi
    (beta-binomial MLE via mle_bb) and phiPlot for phi/psi estimation on single-cell
    Nanopore isoform counts
  repo: github.com/yuntianf/Longcell
- name: LongcellPre
  role: Supplies the gene strand information (gene_bed.rds) needed to build meta splicing
    sites downstream
- name: Seurat
  role: R environment dependency used alongside Longcell in the phi-estimation workflow
- name: dplyr
  role: Data manipulation of the Splice object and filtered phi estimation table
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-60902-2
    title: Single cell and spatial alternative splicing analysis with Nanopore long
      read sequencing
  source_content_hash: sha256:4d50383ccb1faf74dbac9e84f857f3bcc119dd091e230474fa0896a46078b983
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# beta-binomial-psi-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fits a Beta-Binomial model (phi = 1/(alpha+beta+1)) to meta-splicing sites in single-cell Nanopore isoform data to quantify inter-cell splicing heterogeneity separately from intra-cell heterogeneity, using Longcell's Splice object workflow on highly expressed genes (e.g., the CRCLM sample).

## When to use

Use when you have a cell-by-isoform count matrix from single-cell Nanopore long reads and need to estimate inter-cell splicing heterogeneity (phi) versus psi at exon/splice sites, distinguishing isoform co-expression within cells.

## When NOT to use

- Input is raw Nanopore FASTQ without UMI-corrected isoform quantification — run LongcellPre/isoform quantification first
- Genes with very low expression that fail the HighExprsGene threshold of 500 — phi estimates will not pass the >=10 gene-count-in-30-cells filter
- Bulk or tissue-level splicing analysis where cell-level (inter vs intra) heterogeneity decomposition is not the goal

## Inputs

- Cell-by-isoform count matrix from single-cell Nanopore reads (sc_iso_count.rds)
- Gene strand annotation bed file from LongcellPre (gene_bed.rds)

## Outputs

- Filtered phi estimation table (per meta-splice-site phi with bootstrap confidence intervals) written to CSV
- Phi-versus-psi scatter plot from phiPlot annotated by gene
- metaSite object of merged meta-splicing sites

## How to apply

Load Longcell, Seurat, dplyr, future and future.apply, then read the single-cell isoform count table (sc_iso_count.rds) and gene strand annotation (gene_bed.rds from LongcellPre). Build a Splice object with creatSplice_from_df, restrict to highly expressed genes via HighExprsGene (threshold 500; 752 genes in the vignette run), and merge coexisting/mutually exclusive splicing sites into meta-splicing sites with geneSiteTable (eps = 0.05). Estimate inter-cell heterogeneity with genesSitesPhi using method = "beta", which fits the beta-binomial by maximum likelihood (mle_bb) and computes phi = 1/(alpha+beta+1) per meta-site. Retain only confident estimates: gene count >= 10 in over 30 cells, mean psi between 0.1 and 0.9, and bootstrap CI(phi) width < 0.2. Visualize with phiPlot (annot_col = "gene") and export the filtered phi table to CSV.

## Related tools

- **Longcell** (Provides creatSplice_from_df, HighExprsGene, geneSiteTable, genesSitesPhi (beta-binomial MLE via mle_bb) and phiPlot for phi/psi estimation on single-cell Nanopore isoform counts) — github.com/yuntianf/Longcell
- **LongcellPre** (Supplies the gene strand information (gene_bed.rds) needed to build meta splicing sites downstream)
- **Seurat** (R environment dependency used alongside Longcell in the phi-estimation workflow)
- **dplyr** (Data manipulation of the Splice object and filtered phi estimation table)

## Examples

```
crc_phi = genesSitesPhi(crc_splice,genes = heg,method = "beta",verbose = FALSE)
```

## Evaluation signals

- The set of confident phi estimates should be dominated by low-phi sites, matching the paper's recovery of valid estimates for 88 meta-splicing sites in 57 genes from highly expressed CRCLM genes
- HighExprsGene with thresh = 500 should retain ~752 highly expressed genes in the vignette run
- All retained phi estimates must satisfy gene count >= 10 in over 30 cells, mean psi in [0.1, 0.9], and bootstrap CI(phi) width < 0.2
- phi values must fall in (0, 1] and correspond to phi = 1/(alpha+beta+1) from the MLE-fitted beta-binomial
- phiPlot renders a phi-versus-psi scatter with gene-level annotation (annot_col = "gene")

## Limitations

- Requires highly expressed genes; low-expression genes yield too few molecules per cell for confident beta-binomial fits
- Filtering removes most sites — only a small set of meta-splicing sites (88 sites in 57 genes) passed confidence filters in the vignette run
- Bootstrap-based CI(phi) is needed because phi estimates on small cell/sample counts can be unstable

## Evidence

- [methods] Threshold and function calls grounding: "heg = HighExprsGene(crc_splice,thresh = 500)"
- [methods] Beta method call: "crc_phi = genesSitesPhi(crc_splice,genes = heg,method = "beta",verbose = FALSE)"
- [methods] Gene strand need: "The gene strand information is necessary to build meta splicing sites in the downstream analysis, here we can use the `gene_bed.rds`"
- [abstract] Scope of skill: "Longcell also reveals widespread high intra-cell isoform heterogeneity for highly expressed genes."
- [readme] Installation and citation grounding: [evidence span withheld — anchor status: external_doc]
