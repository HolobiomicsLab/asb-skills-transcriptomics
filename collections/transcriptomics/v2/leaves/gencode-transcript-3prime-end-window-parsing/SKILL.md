---
name: gencode-transcript-3prime-end-window-parsing
description: Use when building or validating a cleavage site (CS) catalog from 3′-end
  scRNA-seq data and you need GENCODE-annotated transcript 3′ end coordinates as reference
  intervals for overlap-based classification.
when_to_use_negative:
- You need exact 3′ end matching rather than tolerance-window overlap — the 40 nt
  window deliberately allows coordinate slack around annotated ends
- Your reference annotation is not GENCODE (e.g., you are validating against PolyASite
  2.0, which is a separate downstream overlap step)
- Input reads are 10x Genomics data that do not span cleavage sites, so exact de novo
  3′ end mapping is not possible and window-based validation of de novo CS is inapplicable
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_0114
- http://edamontology.org/topic_3170
tools:
- name: GENCODE
  role: Source of annotated transcript 3′ end coordinates (vM25 mouse, v39 human)
    used to build the 40 nt reference windows
- name: scUTRquant
  role: Downstream pipeline whose CS catalog (mws_cs_annotation.gtf/.bed) is built
    from the GENCODE-window intersection results
  repo: https://github.com/Mayrlab/scUTRquant
- name: Mayrlab/mca-utrome
  role: Snakemake pipeline calling 3′ UTR cleavage sites de novo from the Mouse Cell
    Atlas, whose outputs are classified against GENCODE windows
  repo: https://github.com/Mayrlab/mca-utrome
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-48254-9
    title: Quantifying 3′UTR length from scRNA-seq data reveals changes independent
      of gene expression
  source_content_hash: sha256:eec45ce3843ee855ac8a1ed000b9a20b1663398583edb8e69639815198fcf82b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mayrlab/scUTRquant
  tools:
  - GENCODE
  - Mayrlab/mca-utrome
  - PolyASite 2.0
  - cleanUpdTSeq
  - scUTRquant
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_0114
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# gencode-transcript-3prime-end-window-parsing

## Summary

Parses GENCODE GTF annotations (vM25 mouse, v39 human) to construct 40 nt windows centered on annotated transcript 3′ ends, providing reference coordinate intervals against which candidate Microwell-seq cleavage sites are intersected to classify them as GENCODE-annotated.

## When to use

Use when building or validating a cleavage site (CS) catalog from 3′-end scRNA-seq data and you need GENCODE-annotated transcript 3′ end coordinates as reference intervals for overlap-based classification.

## When NOT to use

- You need exact 3′ end matching rather than tolerance-window overlap — the 40 nt window deliberately allows coordinate slack around annotated ends
- Your reference annotation is not GENCODE (e.g., you are validating against PolyASite 2.0, which is a separate downstream overlap step)
- Input reads are 10x Genomics data that do not span cleavage sites, so exact de novo 3′ end mapping is not possible and window-based validation of de novo CS is inapplicable

## Inputs

- GENCODE vM25 GTF (mouse) annotation file
- GENCODE v39 GTF (human) annotation file
- Candidate cleavage site coordinates from Microwell-seq processing (e.g., BED/GTF of MWS CS)

## Outputs

- Interval file of 40 nt windows centered on GENCODE-annotated transcript 3′ ends
- Labeling of MWS candidate CS as GENCODE-overlapping ('common') vs. non-overlapping via coordinate intersection
- Per-annotation flag in the merged mws_cs_annotation.gtf / mws_cs_annotation.bed catalog

## How to apply

Download the appropriate GENCODE GTF (vM25 for mouse, v39 for human, per the MWS CS atlas workflow) and extract transcript records with verified 3′ ends, filtering to protein-coding transcripts and excluding those tagged mRNA_end_NF. Compute the transcript 3′ end coordinate for each record and construct a 40 nt window centered on that coordinate, exporting the windows as a BED/GTF interval file. Use coordinate-based interval intersection (e.g., bedtools-style overlap) between candidate MWS cleavage site coordinates and these windows, labeling any candidate site falling inside a window as a GENCODE-annotated ('common') CS; sites outside windows proceed to PolyASite 2.0 overlap or are labeled novel. The rationale is that exact 3′ end calls from MWS reads may differ by a few nucleotides from curated annotations, so a tolerance window captures true annotated sites without inflating novelty.

## Related tools

- **GENCODE** (Source of annotated transcript 3′ end coordinates (vM25 mouse, v39 human) used to build the 40 nt reference windows)
- **scUTRquant** (Downstream pipeline whose CS catalog (mws_cs_annotation.gtf/.bed) is built from the GENCODE-window intersection results) — https://github.com/Mayrlab/scUTRquant
- **Mayrlab/mca-utrome** (Snakemake pipeline calling 3′ UTR cleavage sites de novo from the Mouse Cell Atlas, whose outputs are classified against GENCODE windows) — https://github.com/Mayrlab/mca-utrome

## Examples

```
grep -P "\tmRNA\t" gencode.vM25.annotation.gtf | awk -F'\t' '{end=($7=="+")?$5:$4; print $1"\t"end-20"\t"end+20"\t"$9"\t."\t$7}' > gencode_3end_40nt_windows.bed
```

## Evaluation signals

- The merged human and mouse CS catalog extends GENCODE CS annotations by the reported ~40% — a substantially larger or smaller extension indicates mis-sized windows or wrong GTF parsing
- Every candidate CS overlapping a window must be labeled as GENCODE-annotated ('common') in the output GTF/BED; spot-check a few annotated genes for correct strand and coordinate matching
- Windows must be exactly 40 nt and centered on the transcript 3′ end coordinate, honoring strand so 'upstream'/'downstream' context remains correct for later internal-priming scoring
- Transcripts tagged mRNA_end_NF must be excluded from the window set before intersection

## Limitations

- The 40 nt window introduces coordinate tolerance, so candidates just outside a true annotated end may be misclassified as novel
- Only protein-coding transcripts with verified 3′ ends are used (mRNA_end_NF excluded), so non-coding transcript ends are not represented in the reference windows
- GENCODE versions matter: the atlas used vM25 (mouse) and v39 (human) in the workflow, while the scUTRquant manuscript methods cite v39 and vM21 — version mismatches change overlap results

## Evidence

- [methods] Parse the GENCODE vM25 (mouse) and v39 (human) GTF annotations and construct 40 nt windows centered on annotated transcript 3' ends.: [evidence span withheld — anchor status: not_found]
- [methods] intersect windows with candidate CS: [evidence span withheld — anchor status: not_found]
- [methods] transcript filtering rule: "protein coding transcripts that have verified 3' ends in the GENCODE v39 and vM21 annotations"
- [discussion] catalog extension finding: "By integrating our MWS-derived CS with available GENCODE annotations, we provide the most comprehensive CS catalog for human and mouse to date, extending current GENCODE CS annotations by 40%"
- [readme] mca-utrome pipeline context: [evidence span withheld — anchor status: external_doc]
