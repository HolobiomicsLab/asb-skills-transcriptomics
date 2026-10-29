---
name: cs-catalog-categorization-and-export
description: Use when you have candidate 3' end cleavage site coordinates from scRNA-seq
  (e.g. Microwell-seq) and need to annotate them against GENCODE 3' ends and PolyASite
  clusters while removing internal-priming artifacts.
when_to_use_negative:
- Input is 10x Genomics data where most reads do not span the cleavage site — exact
  de novo CS mapping is not possible and this cataloging approach does not apply
- You only need gene-level or 3'UTR isoform quantification with an existing CS annotation
  — use scUTRquant directly rather than rebuilding the catalog
- Input sites have already been categorized against GENCODE/PolyASite — re-annotating
  duplicates work
edam_operation: http://edamontology.org/operation_0361
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: cleanUpdTSeq
  role: Scores candidate CS for internal priming from local upstream/downstream nucleotide
    context; high-scoring sites are removed as artifacts
- name: PolyASite 2.0
  role: Independent poly(A) cluster coordinates used to tag candidate sites not found
    in GENCODE as externally supported
- name: Snakemake
  role: Workflow engine for the upstream hcl-utrome / mca-utrome CS-calling pipelines
    whose outputs feed this categorization
  repo: https://github.com/Mayrlab/mca-utrome
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-48254-9
    title: Quantifying 3′UTR length from scRNA-seq data reveals changes independent
      of gene expression
  source_content_hash: sha256:715fa582318fd1d3261ced374e8f7d6c1cb66ef278a4d8817b9f720dd5197f1c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mayrlab/mca-utrome
  tools:
  - PolyASite 2.0
  - Snakemake
  - cleanUpdTSeq
  edam_operation: http://edamontology.org/operation_0361
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# cs-catalog-categorization-and-export

## Summary

Classify de novo mRNA 3' end cleavage sites (CS) called from Microwell-seq (MWS) data into validated/supported/likely categories relative to GENCODE and PolyASite annotations, and export the merged catalog as GTF/BED files that extend GENCODE CS annotations by ~40%.

## When to use

Use when you have candidate 3' end cleavage site coordinates from scRNA-seq (e.g. Microwell-seq) and need to annotate them against GENCODE 3' ends and PolyASite clusters while removing internal-priming artifacts.

## When NOT to use

- Input is 10x Genomics data where most reads do not span the cleavage site — exact de novo CS mapping is not possible and this cataloging approach does not apply
- You only need gene-level or 3'UTR isoform quantification with an existing CS annotation — use scUTRquant directly rather than rebuilding the catalog
- Input sites have already been categorized against GENCODE/PolyASite — re-annotating duplicates work

## Inputs

- Candidate MWS cleavage site coordinates (BED-like intervals) from hcl-utrome / mca-utrome de novo CS calling
- GENCODE GTF annotations (vM25 mouse, v39 human) with verified 3' ends
- PolyASite 2.0 poly(A) cluster coordinate file
- Local upstream/downstream nucleotide sequences around each candidate CS for cleanUpdTSeq

## Outputs

- mws_cs_annotation.gtf — categorized CS catalog (GENCODE-overlapping, PolyASite-supported, novel)
- mws_cs_annotation.bed — same catalog in BED format
- Internal-priming artifact flags/removals per candidate site

## How to apply

Parse GENCODE GTF annotations (vM25 mouse / v39 human as used in the paper) and build 40 nt windows centered on annotated transcript 3' ends, then intersect candidate MWS CS coordinates with these windows via interval overlap to label GENCODE-overlapping (common) sites. Overlap the remaining sites with PolyASite 2.0 poly(A) cluster coordinates to tag independently supported sites. Score every candidate site for internal priming with cleanUpdTSeq using the local upstream/downstream nucleotide context and remove high-scoring internal-priming artifacts before categorization. Categorize surviving sites into GENCODE-overlapping, PolyASite-supported, and novel groups, and export the catalog as mws_cs_annotation.gtf and mws_cs_annotation.bed for downstream use with scUTRquant. The de novo CS calls feeding this step come from the Human Cell Landscape (hcl-utrome) and Mouse Cell Atlas (mca-utrome) Snakemake pipelines.

## Related tools

- **cleanUpdTSeq** (Scores candidate CS for internal priming from local upstream/downstream nucleotide context; high-scoring sites are removed as artifacts)
- **PolyASite 2.0** (Independent poly(A) cluster coordinates used to tag candidate sites not found in GENCODE as externally supported)
- **Snakemake** (Workflow engine for the upstream hcl-utrome / mca-utrome CS-calling pipelines whose outputs feed this categorization) — https://github.com/Mayrlab/mca-utrome

## Evaluation signals

- The merged human and mouse catalog extends GENCODE CS annotations by the reported ~40%
- All candidate CS falling inside a GENCODE 40 nt 3'-end window are labeled as the common/GENCODE-overlapping class
- No high cleanUpdTSeq internal-priming scores remain among the exported sites (rigorous internal-priming filtering was applied)
- Exports exist in both GTF and BED form (mws_cs_annotation.gtf / mws_cs_annotation.bed) with consistent site coordinates and category labels
- Every exported site belongs to exactly one of the three categories: GENCODE-overlapping, PolyASite-supported, or novel

## Limitations

- CS support depends on MWS data quality; rigorous internal-priming filtering is required because misprimed sites can mimic true 3' ends
- Only sites near annotated 3' ends (40 nt GENCODE windows) or PolyASite clusters can be labeled as supported; the remainder are novel and carry less evidence
- The catalog is specific to protein-coding transcripts with validated 3' ends in the chosen GENCODE release (vM25/vM21 mouse, v39 human)

## Evidence

- [abstract] catalog extends GENCODE by 40%: "We map CS in more than 200 primary human and mouse cell types and increase CS annotations relative to the GENCODE database by 40%."
- [discussion] integration of MWS CS with GENCODE: "By integrating our MWS-derived CS with available GENCODE annotations, we provide the most comprehensive CS catalog for human and mouse to date"
- [discussion] internal priming QC: "We performed rigorous quality control and filtering for internal priming and show that MWS CS are of high quality"
- [methods] GENCODE transcript filter: "protein coding transcripts that have verified 3' ends in the GENCODE v39 and vM21 annotations"
- [readme] MCA CS calling pipeline: [evidence span withheld — anchor status: external_doc]
