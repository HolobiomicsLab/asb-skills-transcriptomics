---
name: locus-level-annotation-extraction-with-flanking-genes
description: Use when a locus-level TE quantification matrix (e.g., MATES output with
  loci named chr|start|end) must be annotated with coordinates and flanking genes
  to interpret cell-type-specific TE markers such as RLTR44B in macrophages.
when_to_use_negative:
- Working from subfamily-level TE counts (e.g., scTE-style subfamily matrices), which
  lack the locus identifiers and coordinates needed for flanking-gene extraction
- Long autonomously expressed TEs such as L1HS spanning several thousand bases, where
  the 2001 bp flanking window and multi-mapping rate estimates are unreliable
- TE copies with high sequence similarity and no uniquely mappable flanking regions
  (e.g., HERVK isoforms), where locus-specific annotation cannot be distinguished
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3295
tools:
- name: MATES
  role: Produces the locus-level TE quantification and locus identifiers (chromosome|start|end)
    that the annotation extraction operates on; TE reference and gene reference built
    via build_reference.py
  repo: https://github.com/mcgilldinglab/MATES
- name: data_processor
  role: MATES module used for downstream analysis (clustering, marker identification)
    whose outputs feed the locus annotation step
  repo: https://github.com/mcgilldinglab/MATES
- name: bedtools
  role: Used to build/trim the TE reference against the gene reference alignment,
    supporting TE-gene overlap and flanking relationships
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-53114-7
    title: 'MATES: a deep learning-based model for locus-specific quantification of
      transposable elements in single cell'
  source_content_hash: sha256:d27d3ea3b4886d2ad7d9eea40e29eeba66c014a9e5ac32994ab56fd2b2ba2450
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/mcgilldinglab/MATES
  repo_url: https://github.com/mcgilldinglab/MATES
  tools:
  - MATES
  - MATES_model
  - TE_quantifier
  - bam_processor
  - bedtools
  - data_processor
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# Locus-level annotation extraction with flanking genes

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extract specific transposable element loci from MATES locus-level quantification together with their genomic coordinates and flanking genes, providing locus-level annotation (e.g., chr13|89384655|89385083 adjacent to Edil3/Hapln1) that links cell-type-specific TE activity to nearby regulatory context.

## When to use

Use when a locus-level TE quantification matrix (e.g., MATES output with loci named chr|start|end) must be annotated with coordinates and flanking genes to interpret cell-type-specific TE markers such as RLTR44B in macrophages.

## When NOT to use

- Working from subfamily-level TE counts (e.g., scTE-style subfamily matrices), which lack the locus identifiers and coordinates needed for flanking-gene extraction
- Long autonomously expressed TEs such as L1HS spanning several thousand bases, where the 2001 bp flanking window and multi-mapping rate estimates are unreliable
- TE copies with high sequence similarity and no uniquely mappable flanking regions (e.g., HERVK isoforms), where locus-specific annotation cannot be distinguished

## Inputs

- MATES locus-level TE count matrix with locus identifiers in chromosome|start|end format
- Cell cluster assignments for the marker TE differential analysis
- TE reference bed/CSV file (e.g., TE_full.bed or TE_nooverlap.bed from build_reference.py)
- Gene annotation GTF used to build the reference genome

## Outputs

- Locus-level annotation table mapping TE loci (e.g., chr13|89384655|89385083) to genomic coordinates
- Flanking-gene associations for marker loci (e.g., Edil3/Hapln1 adjacent to RLTR44B)
- Cell-type-specific TE marker list (RMER16_Mm/RLTR44B, MamRep434/MER124, MURVY-LTR/MamRep1527)

## How to apply

After MATES produces a locus-level TE count matrix (loci keyed as chromosome|start|end), identify cell-type-specific TE markers by differential testing across the clusters (e.g., RMER16_Mm/RLTR44B in macrophages, MamRep434/MER124 in astrocytes, MURVY-LTR/MamRep1527 in oligodendrocytes in the 10x adult mouse brain scATAC-seq data). For each marker locus, parse its identifier into genomic coordinates and query the reference annotation (TE reference bed/CSV built via build_reference.py against RepeatMasker, plus gene GTF) to retrieve flanking genes within the MATES coverage-vector window (1000 bp upstream and downstream of the TE start, i.e., the 2001 bp context used for Vu/Vm vectors). Report the locus with its flanking genes — for RLTR44B, locus chr13|89384655|89385083 flanked by Edil3 and Hapln1 — to connect TE activity with neighboring gene regulation. Validate that the extracted coordinates match the locus identifier and that flanking-gene lookup uses the same reference files used during quantification.

## Related tools

- **MATES** (Produces the locus-level TE quantification and locus identifiers (chromosome|start|end) that the annotation extraction operates on; TE reference and gene reference built via build_reference.py) — https://github.com/mcgilldinglab/MATES
- **data_processor** (MATES module used for downstream analysis (clustering, marker identification) whose outputs feed the locus annotation step) — https://github.com/mcgilldinglab/MATES
- **bedtools** (Used to build/trim the TE reference against the gene reference alignment, supporting TE-gene overlap and flanking relationships)

## Examples

```
python build_reference.py --species Mouse; locus = 'chr13|89384655|89385083'; chr, start, end = locus.split('|')
```

## Evaluation signals

- Locus identifier chr13|89384655|89385083 parses into consistent coordinates (chr13, 89384655-89385083, 429 bp locus) matching the TE reference entries
- Flanking-gene lookup returns Edil3 and Hapln1 for the RLTR44B locus, consistent with the published annotation
- Marker TE assignments match the reported cell-type associations: RMER16_Mm/RLTR44B in macrophages, MamRep434/MER124 in astrocytes, MURVY-LTR/MamRep1527 in oligodendrocytes
- Extracted loci are within the 2001 bp MATES coverage-vector span (1000 bp up/downstream of TE start) so flanking context is well defined

## Limitations

- Locus-specific annotation cannot distinguish identical TE copies such as HERVK isoforms due to high sequence similarity and lack of uniquely mappable flanking regions
- Multi-mapping rate estimates and flanking context may be inaccurate for TEs longer than 2001 bp (e.g., L1HS) because coverage vectors incompletely span the locus
- Trimming TE-gene overlaps (exclusive TE_mode) can unintentionally distort quantification at TEs intersecting neighboring genes, affecting which loci/flanking genes are annotated

## Evidence

- [results] Marker loci and flanking genes: [evidence span withheld — anchor status: not_found]
- [readme] Reference building for species: [evidence span withheld — anchor status: external_doc]
