---
name: cell-type-te-marker-identification
description: Use when a locus-level TE quantification matrix (cell × TE counts) has
  been produced by MATES and the goal is to find TEs that mark specific cell populations
  recovered by clustering or by a gene-expression reference clustering.
when_to_use_negative:
- Only subfamily-level TE counts are available — subfamily-level aggregation cannot
  resolve individual TE copies (e.g. distinguish the RLTR44B locus from other RLTR44B
  copies) and reduces marker specificity
- Identical TE copies such as HERVK isoforms with high sequence similarity and no
  uniquely mappable flanking regions — MATES cannot reliably distinguish these loci,
  so locus-specific marker calls are not trustworthy
- Marker candidate is a long autonomously expressed TE such as L1HS spanning several
  thousand bases — MATES's coverage-vector design limits accurate quantification of
  such TEs
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MATES
  role: Produces the locus-specific TE count matrices that marker identification is
    performed on; TE_quantifier and data_processor modules are used for quantification
    and downstream clustering/marker analysis
  repo: github.com/mcgilldinglab/MATES
- name: scTE
  role: Benchmark single-cell TE quantification method whose marker-TE recovery is
    compared against MATES in Arabidopsis/Drosophila
- name: SoloTE
  role: Benchmark single-cell TE quantification method whose marker-TE recovery is
    compared against MATES
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-53114-7
    title: 'MATES: a deep learning-based model for locus-specific quantification of
      transposable elements in single cell'
  source_content_hash: sha256:9c9f6e08cc053da3340824c183e1abbb5232ecd8c89c7110b0c63ead06395863
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/mcgilldinglab/MATES
  tools:
  - MATES
  - MATES_model
  - SoloTE
  - TE_quantifier
  - bam_processor
  - data_processor
  - scTE
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# cell-type-te-marker-identification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identifies cell-type-specific marker transposable elements from MATES locus-level TE count matrices by clustering cells on TE (optionally gene+TE) expression and testing which TEs are enriched within gene-defined cell populations, e.g. RMER16_Mm/RLTR44B in macrophages and MT2/MERVL-int near Zscan4 genes in 2CLC cells.

## When to use

Use when a locus-level TE quantification matrix (cell × TE counts) has been produced by MATES and the goal is to find TEs that mark specific cell populations recovered by clustering or by a gene-expression reference clustering.

## When NOT to use

- Only subfamily-level TE counts are available — subfamily-level aggregation cannot resolve individual TE copies (e.g. distinguish the RLTR44B locus from other RLTR44B copies) and reduces marker specificity
- Identical TE copies such as HERVK isoforms with high sequence similarity and no uniquely mappable flanking regions — MATES cannot reliably distinguish these loci, so locus-specific marker calls are not trustworthy
- Marker candidate is a long autonomously expressed TE such as L1HS spanning several thousand bases — MATES's coverage-vector design limits accurate quantification of such TEs

## Inputs

- MATES locus-level cell × TE count matrix (per-cell URi + ceil(alpha_i × MRi) counts)
- Gene-expression-based reference cell clustering or cell-type labels
- Cell-by-gene count matrix for gene+TE combined clustering
- TE locus coordinates (e.g. BED/CSV reference with chr|start|end identifiers)
- Optionally gene annotation (GTF) to link marker TE loci to flanking genes

## Outputs

- Per-cell-type marker TE list (e.g. RMER16_Mm, RLTR44B, MamRep434, MER124, MURVY-LTR, MamRep1527, ATCopia66LTR, ATCopia41LTR, GYPSY12_LTR)
- Locus-specific coordinates of marker TE insertions (e.g. chr13|89384655|89385083 for RLTR44B) with flanking genes (Edil3/Hapln1)
- Enrichment statistics / marker scores per TE per cell cluster

## How to apply

Quantify TEs at locus-specific level with the MATES TE_quantifier module (total counts computed as URi + ceil(alpha_i × MRi)), then cluster cells with the data_processor utilities using TE-only or combined gene+TE count matrices. Assign cell identities by agreement with a gene-expression-based reference clustering, then test for TEs enriched per cell type; in the adult mouse brain scATAC-seq data this yields RMER16_Mm/RLTR44B in macrophages, MamRep434/MER124 in astrocytes, and MURVY-LTR/MamRep1527 in oligodendrocytes, with the RLTR44B locus chr13|89384655|89385083 flanked by Edil3/Hapln1. In Arabidopsis and Drosophila scRNA-seq, verify markers against gene-defined populations (ATCopia66LTR, ATCopia41LTR, GYPSY12_LTR). Rationale: locus-level resolution (rather than subfamily-level) is what allows individual TE copies to be linked to specific cell types and nearby flanking genes.

## Related tools

- **MATES** (Produces the locus-specific TE count matrices that marker identification is performed on; TE_quantifier and data_processor modules are used for quantification and downstream clustering/marker analysis) — github.com/mcgilldinglab/MATES
- **scTE** (Benchmark single-cell TE quantification method whose marker-TE recovery is compared against MATES in Arabidopsis/Drosophila)
- **SoloTE** (Benchmark single-cell TE quantification method whose marker-TE recovery is compared against MATES)

## Examples

```
from MATES import MATES_pipeline; mates = MATES_pipeline(TE_mode, data_mode, sample_list_file, bam_path_file); mates.run() # train model and quantify both subfamily and locus-level TE expression
```

## Evaluation signals

- Recovered marker TEs must match the published list: RMER16_Mm/RLTR44B in macrophages, MamRep434/MER124 in astrocytes, MURVY-LTR/MamRep1527 in oligodendrocytes (mouse brain scATAC-seq)
- Marker TEs must be consistent with gene-defined cell populations: ATCopia66LTR and ATCopia41LTR in Arabidopsis and GYPSY12_LTR in Drosophila
- TE-based clustering should agree with the gene-expression reference clustering at reported ARI values (e.g. Arabidopsis Gene+TE ARI 0.6668; Drosophila TE-only ARI 0.3188), since markers are called within those clusters
- Marker TE loci should map to a specific locus identifier (e.g. RLTR44B at chr13|89384655|89385083 near Edil3/Hapln1), confirming locus-level rather than subfamily-level resolution
- Biological coherence check: marker TEs such as MT2 and MERVL-int should be enriched near biologically associated genes (Zscan4c/Zscan4d in 2CLC cells)

## Limitations

- Biological discoveries from MATES-derived marker TEs warrant further experimental validation
- MATES has limited ability to quantify long autonomously expressed TEs such as L1HS spanning several thousand bases, restricting marker discovery for such elements
- Identical TE copies (e.g. HERVK isoforms) with high sequence similarity and lack of uniquely mappable flanking regions cannot be distinguished, so marker assignments to individual copies may be ambiguous

## Evidence

- [readme] MATES downstream analysis tutorial on 10X scATAC data: [evidence span withheld — anchor status: external_doc]
- [full_text] locus-level TE clustering against gene-based clustering benchmark: [evidence span withheld — anchor status: not_found]
