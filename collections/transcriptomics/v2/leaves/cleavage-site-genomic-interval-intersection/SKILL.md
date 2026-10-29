---
name: cleavage-site-genomic-interval-intersection
description: Use when you have de novo-called scRNA-seq cleavage site coordinates
  and need to classify them as GENCODE-validated, externally supported (PolyASite),
  or novel by genomic interval overlap.
when_to_use_negative:
- Input reads come from 10x Genomics data where most reads do not span cleavage sites,
  preventing exact de novo CS mapping — use the prebuilt CS atlas as scUTRquant targets
  instead
- The CS catalog already exists (e.g. scUTRquant-inputs resources); re-intersection
  is redundant for isoform quantification
- You need gene/3'UTR expression quantification rather than CS classification — use
  scUTRquant directly
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: cleanUpdTSeq
  role: Scores candidate cleavage sites for internal priming from local upstream/downstream
    nucleotide context; high-scoring artifacts are removed before/alongside classification
- name: PolyASite 2.0
  role: Provides independently curated poly(A) cluster coordinates used to tag candidate
    sites not in GENCODE as supported
- name: scUTRquant
  role: 'Downstream consumer: uses the CS annotations produced by this classification
    to quantify gene and 3''UTR isoform expression from scRNA-seq data'
  repo: https://github.com/Mayrlab/scUTRquant
- name: Snakemake
  role: Workflow manager for the companion CS-calling pipelines (hcl-utrome, mca-utrome)
    whose output sites are classified here
  repo: https://github.com/Mayrlab/mca-utrome
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-48254-9
    title: Quantifying 3′UTR length from scRNA-seq data reveals changes independent
      of gene expression
  source_content_hash: sha256:f6f784b0f12e1191a81813c8d0e46fef441417a18afbf6c98e1a77147d16b90f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mayrlab/scUTRquant
  tools:
  - PolyASite 2.0
  - Snakemake
  - cleanUpdTSeq
  - scUTRquant
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# cleavage-site-genomic-interval-intersection

## Summary

Classifies candidate mRNA 3' end cleavage sites (CS) de novo-called from Microwell-seq (MWS) data by coordinate-based interval intersection against GENCODE 3' end windows and PolyASite 2.0 clusters, sorting sites into GENCODE-annotated, PolyASite-supported, and novel categories before exporting a CS atlas (mws_cs_annotation.gtf/.bed).

## When to use

Use when you have de novo-called scRNA-seq cleavage site coordinates and need to classify them as GENCODE-validated, externally supported (PolyASite), or novel by genomic interval overlap.

## When NOT to use

- Input reads come from 10x Genomics data where most reads do not span cleavage sites, preventing exact de novo CS mapping — use the prebuilt CS atlas as scUTRquant targets instead
- The CS catalog already exists (e.g. scUTRquant-inputs resources); re-intersection is redundant for isoform quantification
- You need gene/3'UTR expression quantification rather than CS classification — use scUTRquant directly

## Inputs

- Candidate MWS cleavage site coordinate lists (BED-like) from Human Cell Landscape / Mouse Cell Atlas reprocessing
- GENCODE vM25 (mouse) and v39 (human) GTF annotation files
- PolyASite 2.0 poly(A) cluster coordinates
- Local genomic sequence around candidate sites (upstream/downstream nucleotide context) for cleanUpdTSeq internal-priming scoring

## Outputs

- Classified CS catalog with categories: GENCODE-overlapping (common), PolyASite-supported, and novel
- mws_cs_annotation.gtf
- mws_cs_annotation.bed

## How to apply

Parse GENCODE GTF annotations (vM25 mouse, v39 human used in the paper; earlier quantification work used vM21/v39 for transcripts with verified 3' ends, excluding mRNA_end_NF transcripts) and construct 40 nt windows centered on annotated transcript 3' ends. Intersect candidate MWS cleavage site coordinates with these windows via coordinate-based interval intersection, labeling sites falling inside existing annotations; overlap remaining sites with PolyASite 2.0 poly(A) cluster coordinates to tag independently supported sites, leaving the rest as novel. Before or after classification, score sites for internal priming with cleanUpdTSeq on local upstream/downstream nucleotide context and remove high-scoring internal-priming artifacts, since the paper reports rigorous internal-priming QC for MWS CS. Export the categorized catalog as mws_cs_annotation.gtf and mws_cs_annotation.bed for downstream use in scUTRquant 3'UTR isoform quantification.

## Related tools

- **cleanUpdTSeq** (Scores candidate cleavage sites for internal priming from local upstream/downstream nucleotide context; high-scoring artifacts are removed before/alongside classification)
- **PolyASite 2.0** (Provides independently curated poly(A) cluster coordinates used to tag candidate sites not in GENCODE as supported)
- **scUTRquant** (Downstream consumer: uses the CS annotations produced by this classification to quantify gene and 3'UTR isoform expression from scRNA-seq data) — https://github.com/Mayrlab/scUTRquant
- **Snakemake** (Workflow manager for the companion CS-calling pipelines (hcl-utrome, mca-utrome) whose output sites are classified here) — https://github.com/Mayrlab/mca-utrome

## Evaluation signals

- Merged human and mouse CS catalog extends GENCODE CS annotations by the reported ~40%
- Every exported site carries exactly one category label (GENCODE-overlapping/common, PolyASite-supported, or novel)
- GENCODE windows are 40 nt and centered on annotated transcript 3' ends; sanity-check overlap rate of MWS sites against these windows
- High-scoring internal-priming sites flagged by cleanUpdTSeq are absent from the final mws_cs_annotation.gtf/.bed
- Output files parse as valid GTF/BED with coordinates consistent with the input candidate site set (no shifted loci)

## Limitations

- 10x Genomics reads mostly do not span cleavage sites, so this classification must be done on MWS-derived CS; exact de novo mapping is not possible from 10x data
- Categories depend on the chosen annotation versions (GENCODE vM25/v39, PolyASite 2.0); novel sites may simply be unannotated rather than false
- Internal-priming filtering can remove genuine sites whose local sequence context mimics priming artifacts

## Evidence

- [discussion] MWS CS classification via interval intersection: "By integrating our MWS-derived CS with available GENCODE annotations, we provide the most comprehensive CS catalog for human and mouse to date"
- [discussion] Internal priming QC of MWS CS: "We performed rigorous quality control and filtering for internal priming and show that MWS CS are of high quality"
- [abstract] CS catalog extends GENCODE by 40%: "We map CS in more than 200 primary human and mouse cell types and increase CS annotations relative to the GENCODE database by 40%."
- [readme] de novo CS calling pipelines feeding the atlas: [evidence span withheld — anchor status: external_doc]
- [methods] high-confidence CS source datasets: "high-confidence cleavage sites called from the Human Cell Landscape and Mouse Cell Atlas datasets"
