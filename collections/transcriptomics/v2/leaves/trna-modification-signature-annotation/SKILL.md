---
name: trna-modification-signature-annotation
description: Use when sorted BAM alignments of tncRNA-derived small RNA reads to an
  artificial tRNA-plus-flank genome are available and per-tncRNA nucleoside modification
  calls and modification types are needed.
when_to_use_negative:
- Input reads have not been quality-filtered and adapter-trimmed and aligned to the
  artificial tRNA genome — HAMR needs mismatch-aware alignments of tncRNA reads, not
  raw FASTQ
- The goal is de novo tncRNA discovery or subtype classification (tRF-5s, tRF-3s,
  tRF-1, tRNA halves) rather than modification site annotation
- Sequencing depth at candidate sites is below the HAMR minimum read depth of 30,
  since modification calls will not be reliable
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_0659
tools:
- name: HAMR
  role: Predicts modified nucleoside sites on identified tncRNAs from mismatch patterns
    in BAM alignments using a eukaryotic tRNA modification model (v1.2, bundled in
    the toolkit tar package, requires python2)
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: samtools
  role: Produces/manages the sorted BAM alignment of small RNA reads to the artificial
    genome that serves as HAMR input
  repo: https://github.com/skbinfo/tncRNA-Toolkit
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:75b6e97134432a342b988ba398dde92e01cb6b8ccb28a9ebf4c052765ad4f57d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/skbinfo/tncRNA-Toolkit
  repo_url: https://github.com/skbinfo/tncRNA-Toolkit
  tools:
  - HAMR
  - samtools
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# trna-modification-signature-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates modified nucleoside sites (e.g., m3C, pseudouridine, dihydrouridine, m2G) on tRNA-derived non-coding RNAs using HAMR (v1.2) with a eukaryotic tRNA modification model, as done in the tncRNA Toolkit pipeline for six angiosperms.

## When to use

Use when sorted BAM alignments of tncRNA-derived small RNA reads to an artificial tRNA-plus-flank genome are available and per-tncRNA nucleoside modification calls and modification types are needed.

## When NOT to use

- Input reads have not been quality-filtered and adapter-trimmed and aligned to the artificial tRNA genome — HAMR needs mismatch-aware alignments of tncRNA reads, not raw FASTQ
- The goal is de novo tncRNA discovery or subtype classification (tRF-5s, tRF-3s, tRF-1, tRNA halves) rather than modification site annotation
- Sequencing depth at candidate sites is below the HAMR minimum read depth of 30, since modification calls will not be reliable

## Inputs

- Sorted BAM file of tncRNA-derived small RNA reads aligned to the artificial genome (tRNA regions plus 50 nt leader/trailer)
- Reference FASTA of the artificial genome used for the bowtie index
- HAMR eukaryotic tRNA modification model file (models/euk_trna_mods.Rdata)

## Outputs

- Per-tncRNA annotation table listing each tncRNA with detected modification sites and modification types (e.g., m3C, pseudouridine, dihydrouridine, m2G)
- HAMR raw output of predicted modified nucleoside calls

## How to apply

Load the sorted BAM alignment (produced upstream by bowtie with '-v2 --best' and samtools) together with the corresponding reference FASTA of the artificial genome. Run HAMR v1.2 with the eukaryotic tRNA modification model file (models/euk_trna_mods.Rdata) using parameters: minimum read depth 30, minimum 10% mismatch, FDR 0.05, H4 model, then 0.01, 0.05, 0.05. Restrict reported modification sites to positions on mature tRNA or the 50 bp leader/trailer flanks, discarding calls elsewhere. Compile retained calls into a per-tncRNA table listing each tncRNA with its modification sites and modification types; note HAMR 1.2 requires python2 while the surrounding toolkit runs on python3.

## Related tools

- **HAMR** (Predicts modified nucleoside sites on identified tncRNAs from mismatch patterns in BAM alignments using a eukaryotic tRNA modification model (v1.2, bundled in the toolkit tar package, requires python2)) — https://github.com/skbinfo/tncRNA-Toolkit
- **samtools** (Produces/manages the sorted BAM alignment of small RNA reads to the artificial genome that serves as HAMR input) — https://github.com/skbinfo/tncRNA-Toolkit

## Examples

```
python hamr.py IN.bam REF.fa models/euk_trna_mods.Rdata OUT ath 30 10 0.05 H4 0.01 0.05 0.05
```

## Evaluation signals

- Output table contains one entry per tncRNA with modification annotations
- All retained modification sites map to mature tRNA or the 50 bp leader/trailer flank regions per the HAMR output and the pipeline's counting rule
- Predicted modification calls meet HAMR thresholds: minimum read depth 30, minimum 10% mismatch, FDR 0.05
- Modification types are restricted to those in the eukaryotic tRNA model (e.g., m3C, pseudouridine, dihydrouridine, m2G)
- Most predicted modifications fall in the 20th-40th and 50th-60th positions of the parental tRNA, consistent with the paper's reported positional distribution

## Limitations

- HAMR 1.2 requires python2 while the tncRNA Toolkit scripts run on python3, so the two must be executed from the 'util' directory or with both environments available
- Allowing a single mismatch during alignment to overcome sequencing errors may lead to base misidentification and inflation of false negatives
- How tRNA post-transcriptional modifications affect the regulation of tRNA cleavage remains unclear; detailed mechanisms are unresolved
- Modification calls are only retained within mature tRNA plus 50 nt leader/trailer regions; calls elsewhere are discarded

## Evidence
