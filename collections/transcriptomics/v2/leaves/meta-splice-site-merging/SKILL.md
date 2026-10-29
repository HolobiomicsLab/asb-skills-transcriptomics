---
name: meta-splice-site-merging
description: Use when a single-cell Nanopore isoform quantification (Splice object
  from sc_iso_count.rds) contains multiple alternative exons or splice sites per gene,
  and downstream Beta-Binomial inter-cell heterogeneity (phi) estimation requires
  them merged into meta-splicing sites.
when_to_use_negative:
- The gene strand annotation (gene_bed.rds) is unavailable, since gene strand information
  is necessary to build meta splicing sites
- The Splice object has not been filtered with HighExprsGene, so low-expression genes
  with too few junction reads would produce unstable meta-sites
- The goal is per-exon psi visualization on a single well-defined site (psiHist/psiCellPlot)
  rather than gene-level heterogeneity modeling, where merging is unnecessary
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Longcell
  role: Provides geneSiteTable/getMetaSites functions that merge coexisting or mutually
    exclusive splicing sites into meta-splicing sites with eps = 0.05
  repo: github.com/yuntianf/Longcell
- name: LongcellPre
  role: Supplies the gene strand annotation gene_bed.rds required as input to build
    meta splicing sites
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-60902-2
    title: Single cell and spatial alternative splicing analysis with Nanopore long
      read sequencing
  source_content_hash: sha256:fc451481e55f31e020ab0cedc667010387866936bb0c63bed86a335627ce0b6a
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# meta-splice-site-merging

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Merges coexisting or mutually exclusive alternative splicing sites of the same gene into meta-splicing sites using Longcell's geneSiteTable (eps = 0.05), so that downstream Beta-Binomial psi/phi heterogeneity estimation operates on well-supported, non-redundant splicing units.

## When to use

Use when a single-cell Nanopore isoform quantification (Splice object from sc_iso_count.rds) contains multiple alternative exons or splice sites per gene, and downstream Beta-Binomial inter-cell heterogeneity (phi) estimation requires them merged into meta-splicing sites.

## When NOT to use

- The gene strand annotation (gene_bed.rds) is unavailable, since gene strand information is necessary to build meta splicing sites
- The Splice object has not been filtered with HighExprsGene, so low-expression genes with too few junction reads would produce unstable meta-sites
- The goal is per-exon psi visualization on a single well-defined site (psiHist/psiCellPlot) rather than gene-level heterogeneity modeling, where merging is unnecessary

## Inputs

- Splice object built from a single-cell isoform count table (sc_iso_count.rds) via creatSplice_from_df
- Gene strand annotation file gene_bed.rds from the LongcellPre package
- List of highly expressed genes from HighExprsGene(crc_splice, thresh = 500)

## Outputs

- Updated Splice object carrying merged splicing sites per gene
- metaSite object (via getMetaSites()) enumerating meta-splicing sites for downstream phi estimation (e.g. 88 sites in 57 genes for CRCLM highly expressed genes)

## How to apply

Start from a Splice object built with creatSplice_from_df and restrict to highly expressed genes via HighExprsGene (thresh = 500; 752 genes in the CRCLM vignette run), since sparse low-expression genes cannot support robust site grouping. Load the gene strand annotation gene_bed.rds from the LongcellPre package, because strand information is required to define meta splicing sites on the correct transcript strand. Call geneSiteTable on the Splice object with genes = heg and eps = 0.05 so that coexisting or mutually exclusive splicing sites within each gene are merged; the merged sites are stored in a metaSite object retrievable with getMetaSites(). Rationale: individual splice sites on the same isoform set share the same junction-informative reads, so merging them into meta-sites increases read support and stabilizes the psi and Beta-Binomial phi (phi = 1/(alpha+beta+1)) estimates computed afterwards by genesSitesPhi. Verify the merge by inspecting the metaSite object before proceeding to genesSitesPhi(method = "beta") and phiPlot.

## Related tools

- **Longcell** (Provides geneSiteTable/getMetaSites functions that merge coexisting or mutually exclusive splicing sites into meta-splicing sites with eps = 0.05) — github.com/yuntianf/Longcell
- **LongcellPre** (Supplies the gene strand annotation gene_bed.rds required as input to build meta splicing sites)

## Examples

```
crc_splice = geneSiteTable(crc_splice,gene_bed,genes = heg,eps = 0.05,verbose = FALSE)
```

## Evaluation signals

- getMetaSites() returns a populated metaSite object; for the CRCLM highly expressed run, confident phi estimates correspond to 88 meta-splicing sites in 57 genes
- Only coexisting or mutually exclusive splicing sites within the same gene are merged; no cross-gene merges should appear in the metaSite table
- The call used eps = 0.05 and genes = heg; sites from genes outside the HighExprsGene (thresh = 500) set should not be present
- Downstream genesSitesPhi(method = "beta") and phiPlot run without errors on the merged meta-sites, confirming the merge produced valid splicing units

## Limitations

- Gene strand information from LongcellPre is mandatory; without gene_bed.rds the meta-site construction cannot be performed
- The merge granularity depends on the eps parameter (0.05 in the vignette); different eps values change which sites are grouped
- The Longcell repository provides no changelog, so version differences in merging behavior cannot be tracked

## Evidence
