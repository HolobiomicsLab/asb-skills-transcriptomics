---
name: genomic-flanking-sequence-retrieval
description: Use when you have coordinate-annotated modification sites (e.g., BED-format
  Ψ sites from RMBase) and need fixed-width sequence fragments centered on each site
  from a matching reference genome for downstream ML feature encoding.
when_to_use_negative:
- Your modification sites lack genomic coordinates, so BEDTools interval extraction
  cannot be applied
- The reference genome version does not match the annotation's coordinate system (e.g.,
  Ψ sites annotated on hg38 vs. hg19 extraction)
- You need transcript-level sequences rather than genome-derived fragments with fixed
  flanking windows
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0099
tools:
- name: BEDTools
  role: Extracts each Ψ site plus 25 upstream and 25 downstream nucleotides from hg19,
    mm10, and sacCer3 reference genomes to build 51-bp fragments
- name: RMBase v2.0
  role: Source of experimentally validated Ψ site annotations used as extraction coordinates
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2020.07.010
    title: 'MU-PseUDeep: A deep learning method for prediction of pseudouridine sites'
  source_content_hash: sha256:d1ed3e4afc41285d7bfaff541bca313b44456c4dadff26f1a1d4c217f4915c88
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BEDTools
  - MU-PseUDeep
  - Needleman-Wunsch
  - RMBase v2.0
  - Scikit-learn
  - cd-hit-est
  - cd-hit-est-2d
  - pandas
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# genomic-flanking-sequence-retrieval

## Summary

Extract fixed-width genomic fragments centered on annotated modification sites by fetching each site plus flanking bases from a reference genome using BEDTools. In MU-PseUDeep, this converts RMBase v2.0 pseudouridine annotations into 51-bp RNA fragments for human (hg19), mouse (mm10), and yeast (sacCer3).

## When to use

Use when you have coordinate-annotated modification sites (e.g., BED-format Ψ sites from RMBase) and need fixed-width sequence fragments centered on each site from a matching reference genome for downstream ML feature encoding.

## When NOT to use

- Your modification sites lack genomic coordinates, so BEDTools interval extraction cannot be applied
- The reference genome version does not match the annotation's coordinate system (e.g., Ψ sites annotated on hg38 vs. hg19 extraction)
- You need transcript-level sequences rather than genome-derived fragments with fixed flanking windows

## Inputs

- RMBase v2.0 experimentally validated Ψ site annotations for human, mouse, and yeast
- Reference genome FASTA files: hg19 (human), mm10 (mouse), sacCer3 (yeast)
- GENCODE GTF file for transcript/gene region extraction

## Outputs

- 51-bp RNA fragment sequences centered on each Ψ site
- Negative dataset of RNA regions without validated Ψ sites (10x positive count for imbalanced setting)
- BED-format interval files consumable by BEDTools

## How to apply

Download experimentally validated Ψ site annotations from RMBase v2.0 for human, mouse, and yeast, then extract the Ψ site plus 25 bases upstream and 25 bases downstream (a 51-bp fragment) using BEDTools against the appropriate reference genome: hg19 for human, mm10 for mouse, and sacCer3 for yeast. Only fragments with uridine at their center are retained for further prediction. For negatives, collect RNA regions containing no experimentally validated Ψ sites, sampling negatives at 10 times the number of positive samples for the imbalanced configuration. Validate that each species yields 51-nt fragments centered on the modified base and that imbalanced negative sets are 10x the positive class size before feeding fragments into one-of-K encoding.

## Related tools

- **BEDTools** (Extracts each Ψ site plus 25 upstream and 25 downstream nucleotides from hg19, mm10, and sacCer3 reference genomes to build 51-bp fragments)
- **RMBase v2.0** (Source of experimentally validated Ψ site annotations used as extraction coordinates)

## Examples

```
bedtools getfasta -fi hg19.fa -bed psite_sites.bed -s -fo fragments_51nt.fa  # psite_sites.bed holds 51-bp windows (site ± 25 nt) from RMBase v2.0
```

## Evaluation signals

- Each extracted fragment is exactly 51 nt long with the modified base (uridine) at the center position
- For the imbalanced configuration, the negative set is 10 times the size of the positive set
- Every retained fragment passes the uridine-centered filter before proceeding to prediction
- Species-to-genome mapping is correct: hg19 for human, mm10 for mouse, sacCer3 for yeast

## Limitations

- Fragment quality depends on reference genome version matching the annotation coordinate system (hg19/mm10/sacCer3 specifically in this workflow)
- Fragments lacking a uridine at the center must be discarded, which can shrink the usable dataset
- Only experimentally validated Ψ sites from RMBase v2.0 are covered; novel or unannotated sites are not extracted

## Evidence

- [methods] Extract Ψ site with 25-bp flanks via BEDTools: "we extracted the Ψ and surrounding 25 bases upstream and downstream nucleotides using BEDTools"
- [methods] Uridine-centered fragment filter: "Only those fragments with uridine at their center were considered for further prediction"
- [methods] Negative dataset construction without validated Ψ sites: "To create the negative dataset, we collected those regions of RNA that did not contain any experimentally validated Ψ sites"
- [methods] Download Ψ sites from RMBase v2.0: "The Ψ site information was downloaded from RMBase v2.0"
- [methods] Reference genomes hg19, mm10, sacCer3: "with reference files of three species, hg19 (human)"
- [other] Validation: 51-bp fragments centered on modified base: [evidence span withheld — anchor status: not_found]
