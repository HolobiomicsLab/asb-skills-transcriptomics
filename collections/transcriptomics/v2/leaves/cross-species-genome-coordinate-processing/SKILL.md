---
name: cross-species-genome-coordinate-processing
description: Use when building training data for RNA modification site prediction
  from RMBase v2.0 BED/GTF annotations across multiple species, requiring genome-coordinate
  extraction of flanking sequence windows.
when_to_use_negative:
- Your input is already extracted, center-aligned RNA fragments rather than genome
  coordinates
- Your modification annotations are species-specific with no matching reference genome
  available (e.g. assemblies other than hg19/mm10/sacCer3)
- You need modification types other than pseudouridine (Ψ), which this extraction
  scheme was built around
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0099
tools:
- name: BEDTools
  role: Extracts the Ψ site plus 25 bases upstream and downstream from species reference
    genomes using coordinate intervals; also intersects the hg19 genome with a GENCODE
    gtf file
- name: MU-PseUDeep
  role: 'Downstream consumer: the extracted 51-nt fragments are one-of-K encoded and
    fed into its two CNN input branches'
  repo: https://github.com/smk5g5/MU-PseUDeep
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2020.07.010
    title: 'MU-PseUDeep: A deep learning method for prediction of pseudouridine sites'
  source_content_hash: sha256:acdd5c88c1a658bb9b471e20c84c24ee21a02bd842ec4e4e66c645f159cabb9b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/smk5g5/MU-PseUDeep
  tools:
  - BEDTools
  - MU-PseUDeep
  - Needleman-Wunsch
  - Scikit-learn
  - cd-hit-est
  - cd-hit-est-2d
  - pandas
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 20
---

# Cross-species genome coordinate processing

## Summary

Extracts 51-nt RNA fragments centered on experimentally validated Ψ sites from RMBase v2.0 annotations by lifting BED coordinate intervals against species-specific reference genomes (hg19 human, mm10 mouse, sacCer3 yeast) using BEDTools.

## When to use

Use when building training data for RNA modification site prediction from RMBase v2.0 BED/GTF annotations across multiple species, requiring genome-coordinate extraction of flanking sequence windows.

## When NOT to use

- Your input is already extracted, center-aligned RNA fragments rather than genome coordinates
- Your modification annotations are species-specific with no matching reference genome available (e.g. assemblies other than hg19/mm10/sacCer3)
- You need modification types other than pseudouridine (Ψ), which this extraction scheme was built around

## Inputs

- RMBase v2.0 experimentally validated Ψ site annotations for human, mouse, and yeast
- Reference genome FASTA files: hg19 (human), mm10 (mouse), sacCer3 (yeast)
- GENCODE gtf file for transcript/gene region annotation
- BED-formatted intervals of Ψ positions ± 25 nt

## Outputs

- 51-nt sequence fragments centered on the Ψ site, one set per species
- Negative-set coordinate intervals (RNA regions with no validated Ψ sites)
- Genomic region metadata for transcriptome-wide scanning (supplemental table)

## How to apply

Download experimentally validated Ψ site annotations from RMBase v2.0 for human, mouse, and yeast, then convert each modification site into a BED interval covering the Ψ position plus 25 bases upstream and downstream (51-bp fragment). Run BEDTools against the species-matched reference genomes — hg19 for human, mm10 for mouse, sacCer3 for yeast — to extract the nucleotide sequence underlying each interval, verifying the fragment has the modified uridine at its center. For transcriptome-context work (e.g. defining protein-coding gene regions), intersect genome FASTA with a GENCODE gtf file using BedTools. Collect negative regions from RNA segments lacking any experimentally validated Ψ site so downstream classifiers receive coordinate-consistent positives and negatives across species. The rationale is that all three species share the same 51-nt window convention, enabling one model to train on consistently encoded multi-species inputs.

## Related tools

- **BEDTools** (Extracts the Ψ site plus 25 bases upstream and downstream from species reference genomes using coordinate intervals; also intersects the hg19 genome with a GENCODE gtf file)
- **MU-PseUDeep** (Downstream consumer: the extracted 51-nt fragments are one-of-K encoded and fed into its two CNN input branches) — https://github.com/smk5g5/MU-PseUDeep

## Examples

```
bedtools getfasta -fi hg19.fa -bed psi_sites_hg19.bed -fo fragments_hg19.fa; bedtools getfasta -fi mm10.fa -bed psi_sites_mm10.bed -fo fragments_mm10.fa; bedtools getfasta -fi sacCer3.fa -bed psi_sites_sacCer3.bed -fo fragments_sacCer3.fa
```

## Evaluation signals

- Each extracted fragment is exactly 51 nt with the modified uridine at the center position
- Species-specific fragment sets align to the correct reference build: hg19 for human, mm10 for mouse, sacCer3 for yeast
- Positive Ψ positions match RMBase v2.0 experimentally validated site coordinates
- Negative-region intervals contain no experimentally validated Ψ sites
- Coordinate extraction is reproducible: re-running BEDTools getfasta/intersect on the same BED + reference FASTA yields identical fragments

## Limitations

- Bound to three specific reference builds (hg19, mm10, sacCer3); other assemblies would require liftover and revalidation of fragment centering
- Only fragments with a uridine at their center were considered for further prediction, so non-uridine-centered extractions are discarded
- Ψ site annotations are limited to experimentally validated entries in RMBase v2.0, restricting coverage to measured modifications

## Evidence

- [methods] RMBase v2.0 annotation download: "The Ψ site information was downloaded from RMBase v2.0"
- [methods] Fragment extraction with BEDTools: "we extracted the Ψ and surrounding 25 bases upstream and downstream nucleotides using BEDTools"
- [methods] Species reference files: "with reference files of three species, hg19 (human), mm10 (mouse), and sacCer3 (yeast)"
- [methods] GENCODE region intersection: "obtained using BedTools from the hg19 genome and gencode gtf file"
- [other] Workflow task card: cross-species extraction procedure: [evidence span withheld — anchor status: not_found]
