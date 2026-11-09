---
name: polya-cluster-overlap-annotation
description: Use when you have candidate 3' end cleavage site coordinates from 3'-tag
  scRNA-seq (e.g., Microwell-seq) and need to label them against GENCODE 3' ends,
  PolyASite 2.0 clusters, and internal-priming risk.
when_to_use_negative:
- Input is 10x Genomics data where reads do not span the mRNA 3' end cleavage site,
  preventing exact de novo CS mapping and thus reliable overlap-based classification
- The goal is quantifying 3'UTR isoform abundance from scRNA-seq counts rather than
  building or extending a cleavage site annotation catalog (use scUTRquant quantification
  instead)
- Candidate sites have not yet passed quality control for internal priming — classification
  must not be run before the cleanUpdTSeq filtering step
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0769
- http://edamontology.org/topic_0622
tools:
- name: cleanUpdTSeq
  role: Scores each candidate cleavage site for internal priming using local upstream/downstream
    nucleotide context; high-scoring artifacts are removed prior to annotation classes
- name: PolyASite 2.0
  role: Provides independently curated poly(A) cluster coordinates used to tag candidate
    sites as independently supported
- name: GENCODE
  role: Reference annotation (vM25 mouse / v39 human) whose transcript 3' ends define
    40 nt windows for the coordinate-based intersection
- name: Snakemake
  role: Workflow engine used in the companion de novo cleavage-site calling repositories
    (mca-utrome, hcl-utrome) that produce the candidate CS input
  repo: https://github.com/Mayrlab/mca-utrome
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-48254-9
    title: Quantifying 3′UTR length from scRNA-seq data reveals changes independent
      of gene expression
  source_content_hash: sha256:8ce7eca75285d3969fbb38574134f5f6e98740ff478f526d5ab14ee5c2c9b100
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mayrlab/mca-utrome
  tools:
  - GENCODE
  - PolyASite 2.0
  - Snakemake
  - cleanUpdTSeq
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0769
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# polya-cluster-overlap-annotation

## Summary

Classifies candidate mRNA 3' end cleavage sites (CS) called from Microwell-seq (MWS) data by coordinate-based overlap with GENCODE-annotated 3' ends and PolyASite 2.0 poly(A) clusters, after removing internal-priming artifacts with cleanUpdTSeq, to build the validated/supported/likely CS catalog.

## When to use

Use when you have candidate 3' end cleavage site coordinates from 3'-tag scRNA-seq (e.g., Microwell-seq) and need to label them against GENCODE 3' ends, PolyASite 2.0 clusters, and internal-priming risk.

## When NOT to use

- Input is 10x Genomics data where reads do not span the mRNA 3' end cleavage site, preventing exact de novo CS mapping and thus reliable overlap-based classification
- The goal is quantifying 3'UTR isoform abundance from scRNA-seq counts rather than building or extending a cleavage site annotation catalog (use scUTRquant quantification instead)
- Candidate sites have not yet passed quality control for internal priming — classification must not be run before the cleanUpdTSeq filtering step

## Inputs

- Candidate MWS cleavage site coordinates (e.g., BED from de novo CS calling on Human Cell Landscape / Mouse Cell Atlas MWS data)
- GENCODE GTF annotations (vM25 mouse, v39 human) with protein-coding transcripts having verified 3' ends
- PolyASite 2.0 poly(A) cluster coordinate file
- Local genomic nucleotide sequence upstream/downstream of each candidate CS for cleanUpdTSeq scoring

## Outputs

- mws_cs_annotation.gtf (classified CS catalog)
- mws_cs_annotation.bed (classified CS catalog)
- Per-site labels: GENCODE-overlapping (common), PolyASite-supported, or novel

## How to apply

Parse GENCODE GTF annotations (vM25 for mouse, v39 for human) and build 40 nt windows centered on annotated transcript 3' ends. Intersect candidate MWS CS coordinates with these windows via coordinate-based interval intersection to label sites that fall inside existing annotations; overlap remaining sites with PolyASite 2.0 poly(A) cluster coordinates to tag independently supported sites. Score every candidate site for internal priming with cleanUpdTSeq using the local upstream/downstream nucleotide context, and remove high-scoring internal-priming artifacts before final categorization. Categorize the surviving sites into GENCODE-overlapping (common/validated), PolyASite-supported, or novel classes, and export the merged human and mouse catalog as mws_cs_annotation.gtf and mws_cs_annotation.bed. The rationale is that each CS receives independent annotation support rather than being accepted de novo, which underpins the claim that the atlas extends GENCODE CS annotations by ~40%.

## Related tools

- **cleanUpdTSeq** (Scores each candidate cleavage site for internal priming using local upstream/downstream nucleotide context; high-scoring artifacts are removed prior to annotation classes)
- **PolyASite 2.0** (Provides independently curated poly(A) cluster coordinates used to tag candidate sites as independently supported)
- **GENCODE** (Reference annotation (vM25 mouse / v39 human) whose transcript 3' ends define 40 nt windows for the coordinate-based intersection)
- **Snakemake** (Workflow engine used in the companion de novo cleavage-site calling repositories (mca-utrome, hcl-utrome) that produce the candidate CS input) — https://github.com/Mayrlab/mca-utrome

## Evaluation signals

- The merged human and mouse CS catalog extends GENCODE CS annotations by the reported ~40%
- Sites classified as 'common' are exactly those GENCODE-annotated 3' ends also observed in the MWS data
- No high-scoring internal-priming sites remain after cleanUpdTSeq filtering (QC demonstrated 'MWS CS are of high quality')
- Output files mws_cs_annotation.gtf and mws_cs_annotation.bed are present, internally consistent, and carry the three-way class labels
- Coordinates in the output fall within the 40 nt GENCODE 3' end windows or PolyASite 2.0 clusters as labeled

## Limitations

- Most reads from 10x Genomics data do not span mRNA 3' end cleavage sites, so this de novo mapping/classification approach is not directly transferable to 10x data
- cleanUpdTSeq internal-priming removal depends on local sequence context and may not capture all genomic A-rich priming-prone regions
- Sites observed in only few cell types make up roughly half of all CS, so support for rare sites is inherently weaker than for major 3' ends

## Evidence

- [methods] Intersect candidate MWS cleavage site coordinates with these GENCODE 3' end windows: [evidence span withheld — anchor status: not_found]
- [methods] Score all candidate sites for internal priming with cleanUpdTSeq: [evidence span withheld — anchor status: not_found]
- [methods] Overlap the remaining candidate sites with PolyASite 2.0: [evidence span withheld — anchor status: not_found]
- [discussion] extending current GENCODE CS annotations by 40%: "By integrating our MWS-derived CS with available GENCODE annotations, we provide the most comprehensive CS catalog for human and mouse to date, extending current GENCODE CS annotations by 40%"
- [discussion] rigorous quality control and filtering for internal priming: "We performed rigorous quality control and filtering for internal priming and show that MWS CS are of high quality"
