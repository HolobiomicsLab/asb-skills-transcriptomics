---
name: polya-site-rud-index-computation
description: Use when a PACdataset contains multiple poly(A) sites per gene (3'UTR
  APA) and per-gene/per-cell RUD scores are needed to quantify 3'UTR shortening or
  lengthening across cells or cell types.
when_to_use_negative:
- The PACdataset contains genes with only a single poly(A) site — RUD requires at
  least a proximal–distal pA pair, so get3UTRAPAds/get3UTRAPApd must exclude them
  first.
- The goal is genome-browser-style visualization of raw poly(A) site usage per site
  (vizTracks/vizStats) rather than a gene-level shortening index.
- The pA sites are not annotated to 3'UTR features (e.g., intronic or intergenic pAs
  without prior annotatePAC/ext3UTRPACds).
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: vizAPA
  role: Provides getAPAindexPACds (choose2PA='PD') to compute the per-gene, per-cell
    RUD index from a 3'UTR-subset PACdataset
  repo: https://github.com/BMILAB/vizAPA
- name: movAPA
  role: Supplies get3UTRAPAds and get3UTRAPApd (minDist=50, maxDist=5000, minRatio=0.05)
    to select genes and proximal–distal pA pairs for RUD/smartRUD calculation
- name: scAPAtrap
  role: Alternative upstream tool whose poly(A) site counts (converted to PACdataset
    via readPACds) serve as RUD input
  repo: https://github.com/BMILAB/scAPAtrap/
- name: Sierra
  role: Alternative upstream tool whose peak counts are converted into a PACdataset
    with readPACds before RUD computation
  repo: https://github.com/VCCRI/Sierra/wiki/Sierra-Vignette
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae099
    title: 'vizAPA: visualizing dynamics of alternative polyadenylation from bulk
      and single-cell data'
  source_content_hash: sha256:a3d5b946eee4286d040161ed56111d0ed0a0feda577e5257b813909cee527d81
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BMILAB/vizAPA
  tools:
  - R
  - Sierra
  - movAPA
  - scAPAtrap
  - vizAPA
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# polya-site-rud-index-computation

## Summary

Compute per-gene, per-cell Relative Usage of Distal poly(A) site (RUD) indices from a PACdataset of poly(A) site counts, quantifying 3'UTR shortening/lengthening at single-cell resolution. This is the core quantitative step in vizAPA for turning proximal–distal site usage ratios into a cell-level APA score overlayable on UMAP embeddings.

## When to use

Use when a PACdataset contains multiple poly(A) sites per gene (3'UTR APA) and per-gene/per-cell RUD scores are needed to quantify 3'UTR shortening or lengthening across cells or cell types.

## When NOT to use

- The PACdataset contains genes with only a single poly(A) site — RUD requires at least a proximal–distal pA pair, so get3UTRAPAds/get3UTRAPApd must exclude them first.
- The goal is genome-browser-style visualization of raw poly(A) site usage per site (vizTracks/vizStats) rather than a gene-level shortening index.
- The pA sites are not annotated to 3'UTR features (e.g., intronic or intergenic pAs without prior annotatePAC/ext3UTRPACds).

## Inputs

- PACdataset object of poly(A) site counts (e.g., from Sierra or scAPAtrap via readPACds), with @anno$ftr annotations
- Cell type annotation in PACdataset @colData (e.g., celltype factor releveled to SC/RS/ES)
- Genome annotation (gtf via annoHub/annotatePAC) and optional BSgenome for internal priming removal

## Outputs

- Per-gene, per-cell RUD index PACdataset from getAPAindexPACds (iPACds)
- Proximal–distal pA pair selection (pd) from movAPA::get3UTRAPApd for smartRUD
- Mean per-cell RUD scores for overlaying on vizUMAP embeddings or differential APA marker testing

## How to apply

Subset the PACdataset to 3'UTR features with scPACds[scPACds@anno$ftr=='3UTR'], because only genes with 3'UTR APA are usable for RUD calculation; alternatively preselect genes via movAPA::get3UTRAPAds or pick proximal–distal pairs with movAPA::get3UTRAPApd (minDist=50, maxDist=5000, minRatio=0.05) for smartRUD. Compute the index with vizAPA::getAPAindexPACds using choose2PA='PD', which selects a proximal–distal pA pair per gene and yields a per-gene, per-cell RUD PACdataset where decreased RUD scores indicate 3'UTR shortening. Optionally remove internal priming first (removePACdsIP, e.g. up=-140, dn=10, conA=6) and extend 3'UTRs with ext3UTRPACds(scPACds, 1000) so distal sites are correctly captured. Downstream, mean per-cell RUD scores can be overlaid as gradient colors on vizUMAP 2D embeddings or used as APA markers with avg_log2FC comparing cell groups (e.g., lower RUD in ES than SC spermatogenic cells).

## Related tools

- **vizAPA** (Provides getAPAindexPACds (choose2PA='PD') to compute the per-gene, per-cell RUD index from a 3'UTR-subset PACdataset) — https://github.com/BMILAB/vizAPA
- **movAPA** (Supplies get3UTRAPAds and get3UTRAPApd (minDist=50, maxDist=5000, minRatio=0.05) to select genes and proximal–distal pA pairs for RUD/smartRUD calculation)
- **scAPAtrap** (Alternative upstream tool whose poly(A) site counts (converted to PACdataset via readPACds) serve as RUD input) — https://github.com/BMILAB/scAPAtrap/
- **Sierra** (Alternative upstream tool whose peak counts are converted into a PACdataset with readPACds before RUD computation) — https://github.com/VCCRI/Sierra/wiki/Sierra-Vignette

## Examples

```
scPACds<-scPACds[scPACds@anno$ftr=="3UTR"]; iPACds=getAPAindexPACds(scPACds, choose2PA="PD")
```

## Evaluation signals

- The RUD PACdataset contains only genes with ≥2 3'UTR pAs (proximal–distal pair selected by choose2PA='PD').
- Overlaying mean per-cell RUD on vizUMAP shows a gradual transition of decreased RUD scores (3'UTR shortening) along SC -> RS -> ES in the mouse sperm demo.
- For known APA dynamics (e.g., Asrgl1), RUD scores decrease from SC to ES; APA markers between ES and SC show avg_log2FC<0 (lower RUD/shorter 3'UTR in ES).
- Internal priming candidates were removed (removePACdsIP) before index computation, so scores are not confounded by genomic A-stretches.
- Proximal–distal pair filters were applied: minDist=50, maxDist=5000, minRatio=0.05.

## Limitations

- Only genes with 3'UTR APA (multiple 3'UTR pAs) can be used for RUD calculation.
- RUD is computed after subsetting to 3'UTR features; pAs in other genomic features are excluded from the index.
- RUD depends on the quality of the upstream pA identification (Sierra or scAPAtrap); scAPAtrap recommends merging peak- and tail-based results and retaining tail-supported or sufficiently expressed pAs.
- scPACds demo data are sparse (974 pAs from 413 genes in mouse sperm cells), so per-cell mean RUD is noisy for lowly covered cells.

## Evidence

- [methods] Subset PACds to 3UTR features before RUD calculation: "scPACds<-scPACds[scPACds@anno$ftr=="3UTR"]"
- [methods] getAPAindexPACds choose2PA='PD' (proximal-distal RUD index): "iPACds=getAPAindexPACds(scPACds, choose2PA = "PD")"
- [methods] get3UTRAPApd (choose proximal and distal pAs for smartRUD): "pd=movAPA::get3UTRAPApd(pacds=scPACds, 
                        minDist=50, maxDist=5000, 
                        minRatio=0.05"
- [methods] 3'UTR shortening finding: "The plot shows transition of 3' UTR shortening (i.e., decreased RUD scores) during sperm cell differentiation (SC -> RS -> ES)."
- [readme] vizUMAP module description: [evidence span withheld — anchor status: external_doc]
