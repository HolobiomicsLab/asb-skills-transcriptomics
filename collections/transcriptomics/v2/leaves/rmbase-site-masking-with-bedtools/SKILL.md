---
name: rmbase-site-masking-with-bedtools
description: Use when preparing a human (or other species) transcriptome scan for
  novel RNA modification sites and known experimentally validated sites from RMBase
  must be excluded from prediction, e.g. before scoring uridine-centered windows.
when_to_use_negative:
- When the goal is to validate or re-detect known RMBase Ψ sites rather than discover
  novel ones — masking would remove exactly the positive signal
- When working with a species or modification type not covered by RMBase v2.0 annotations,
  since no reliable mask intervals can be constructed
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_0659
tools:
- name: BEDTools
  role: Extracts coding sequences from hg19 with the GENCODE gtf and masks known RMBase
    Ψ sites together with 25 bases of flanking sequence on both sides
- name: RMBase v2.0
  role: Source of experimentally validated Ψ site coordinates used to construct the
    mask intervals
- name: SeqKit
  role: Generates running 51-base-pair windows across the masked transcriptome; only
    fragments with a uridine at the center are retained for prediction
- name: MU-PseUDeep
  role: Downstream deep learning model that scores the masked, uridine-centered fragments
    for potential Ψ sites at a >0.99 precision threshold
  repo: https://github.com/smk5g5/MU-PseUDeep
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2020.07.010
    title: 'MU-PseUDeep: A deep learning method for prediction of pseudouridine sites'
  source_content_hash: sha256:7fa175b8667f7b6336221d2ceab0cf8ebbb9f7e9b7818b8e96c885e31518b08e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/smk5g5/MU-PseUDeep
  tools:
  - BEDTools
  - MU-PseUDeep
  - RMBase v2.0
  - SeqKit
  - hg19
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# rmbase-site-masking-with-bedtools

## Summary

Masks known RMBase v2.0 pseudouridine (Ψ) sites plus 25 bases of flanking sequence on each side using BEDTools, so that a downstream transcriptome-wide scan (e.g., with MU-PseUDeep) targets only novel candidate Ψ sites.

## When to use

Use when preparing a human (or other species) transcriptome scan for novel RNA modification sites and known experimentally validated sites from RMBase must be excluded from prediction, e.g. before scoring uridine-centered windows.

## When NOT to use

- When the goal is to validate or re-detect known RMBase Ψ sites rather than discover novel ones — masking would remove exactly the positive signal
- When working with a species or modification type not covered by RMBase v2.0 annotations, since no reliable mask intervals can be constructed

## Inputs

- RMBase v2.0 Ψ site coordinate table (genomic positions of experimentally validated pseudouridine sites)
- GENCODE gtf annotation file
- Reference genome FASTA (hg19 for the human transcriptome scan)
- Flanking window size parameter (25 bases on each side)

## Outputs

- BED interval file of masked genomic regions (known Ψ sites ± 25 bp flanks)
- Masked transcriptome coding sequences with known sites excluded
- Cleaned uridine-centered 51-mer fragment set (via SeqKit) free of documented Ψ sites

## How to apply

Download Ψ site coordinates from RMBase v2.0 and the GENCODE gtf annotation, then use BEDTools to expand each known Ψ site by 25 bases upstream and 25 bases downstream so that any 51-base fragment containing a known site is masked. Apply the same BEDTools-based extraction workflow to the reference genome (hg19 for human), respecting each gene's strand when converting DNA to RNA. The rationale is that the subsequent uridine-centered 51-mer scan (generated with SeqKit) should only score fragments without documented Ψ sites, so predicted sites can be interpreted as novel. After prediction, cross-check the predicted gene set back against RMBase to quantify how many predictions recover known sites (e.g., 284 of 2441 predicted genes in the human transcriptome scan).

## Related tools

- **BEDTools** (Extracts coding sequences from hg19 with the GENCODE gtf and masks known RMBase Ψ sites together with 25 bases of flanking sequence on both sides)
- **RMBase v2.0** (Source of experimentally validated Ψ site coordinates used to construct the mask intervals)
- **SeqKit** (Generates running 51-base-pair windows across the masked transcriptome; only fragments with a uridine at the center are retained for prediction)
- **MU-PseUDeep** (Downstream deep learning model that scores the masked, uridine-centered fragments for potential Ψ sites at a >0.99 precision threshold) — https://github.com/smk5g5/MU-PseUDeep

## Examples

```
bedtools intersect -a uridine_centered_51mers.bed -b rmbase_psi_sites.bed -v -s > novel_fragments.bed
```

## Evaluation signals

- Masked regions should equal the number of RMBase Ψ sites extended by exactly 25 bp on each side (51 bp total mask per site)
- After masking and 51-mer generation with SeqKit, no retained uridine-centered fragment should overlap a known RMBase Ψ site ± 25 bp
- Cross-checking predicted genes against RMBase should recover the expected overlap (284 of 2441 predicted genes already have a documented RMBase Ψ site in the published human scan)
- Total predicted genes with one or more Ψ sites at the >0.99 precision threshold should be in the same range as the reported 2441 protein-coding genes
- Strand correctness: extracted RNA sequences should match each gene's orientation as defined in the GENCODE gtf

## Limitations

- Only sites documented in RMBase v2.0 can be masked; known Ψ sites absent from RMBase will remain in the scan and may be reported as false 'novel' predictions
- Masking with a fixed 25-base flank is tuned for the 51-base fragment size used by MU-PseUDeep; other fragment lengths would require adjusting the flank width
- Results depend on the hg19 genome build and GENCODE annotation version; coordinates are not transferable to other builds without liftOver

## Evidence

- [methods] Mask known RMBase pseudouridine sites together with 25 bases of flanking sequence on both sides using BEDTools so the scan targets novel sites: [evidence span withheld — anchor status: not_found]
- [methods] Extraction via BEDTools: "we extracted the Ψ and surrounding 25 bases upstream and downstream nucleotides using BEDTools"
- [results] RMBase as validation reference: "284 genes already had one or more known Ψ site as documented in the RMBase"
- [methods] Genome and annotation source: "obtained using BedTools from the hg19 genome and gencode gtf file"
- [methods] Uridine-centered fragment filter: "Only those fragments with uridine at their center were considered for further prediction"
