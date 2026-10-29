---
name: bam-alignment-processing
description: Use when a sorted BAM of small-RNA reads aligned to a tRNA-masked artificial
  genome and a matching reference FASTA are available and the goal is to predict modified
  nucleoside sites along tncRNAs.
when_to_use_negative:
- Input reads are unprocessed FASTQ or not yet adapter-trimmed and aligned to the
  artificial genome — run the upstream tncRNA detection workflow first
- The sequencing data is conventional RNA-seq rather than sRNA-seq; the article notes
  modification biases require tRNA-seq to overcome
- The goal is general small-RNA annotation (miRNAs, piRNAs, rRNAs) rather than modified
  nucleosides on tRNA-derived reads — a pipeline like SPORTS1.0 is more appropriate
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3170
tools:
- name: HAMR
  role: Predicts modified nucleoside sites from the BAM alignment using mismatch signatures
    and the eukaryotic tRNA modification model
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: samtools
  role: Produces and handles the sorted BAM alignment of tncRNA-derived reads fed
    to HAMR
- name: tncRNA Toolkit
  role: Wraps HAMR (v1.2, bundled in util/) to compile retained modification calls
    into the per-tncRNA annotation table
  repo: https://github.com/skbinfo/tncRNA-Toolkit
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:df1aa5032b058f1276fafc2ac435c14f67b1f3dab03318c5f213d82eba9e49ae
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
  - tncRNA Toolkit
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# bam-alignment-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs the HAMR-based modification site prediction step of the tncRNA Toolkit, which reads a sorted BAM of tncRNA-derived small RNA reads aligned to an artificial genome and calls modified nucleosides (e.g., m3C, pseudouridine, dihydrouridine, m2G) on the identified tncRNAs.

## When to use

Use when a sorted BAM of small-RNA reads aligned to a tRNA-masked artificial genome and a matching reference FASTA are available and the goal is to predict modified nucleoside sites along tncRNAs.

## When NOT to use

- Input reads are unprocessed FASTQ or not yet adapter-trimmed and aligned to the artificial genome — run the upstream tncRNA detection workflow first
- The sequencing data is conventional RNA-seq rather than sRNA-seq; the article notes modification biases require tRNA-seq to overcome
- The goal is general small-RNA annotation (miRNAs, piRNAs, rRNAs) rather than modified nucleosides on tRNA-derived reads — a pipeline like SPORTS1.0 is more appropriate

## Inputs

- Sorted BAM alignment of tncRNA-derived small RNA reads to the artificial genome (from bowtie/samtools)
- Reference FASTA of the artificial genome (tRNA gene regions masked with 50 nt upstream and downstream)
- HAMR v1.2 eukaryotic tRNA modification model file (models/euk_trna_mods.Rdata)

## Outputs

- HAMR modified-nucleoside call output per tncRNA
- Per-tncRNA annotation table listing each tncRNA with its detected modification sites and modification types

## How to apply

Load the sorted BAM produced upstream by bowtie ('-v2 --best' alignment to the artificial genome) plus the corresponding reference FASTA. Run HAMR v1.2 with the eukaryotic tRNA modification model file models/euk_trna_mods.Rdata and the pipeline parameters: minimum read depth 30, minimum 10% mismatch, FDR 0.05, H4 model, followed by 0.01 0.05 0.05, as in 'python hamr.py IN.bam REF.fa models/euk_trna_mods.Rdata OUT ath 30 10 0.05 H4 0.01 0.05 0.05' (note HAMR v1.2 requires python2). Restrict reported modification sites to positions on mature tRNA and the 50 bp leader/trailer flank regions, discarding calls elsewhere, then compile the retained calls into a per-tncRNA annotation table listing each tncRNA with its modification sites and modification types. The rationale is that mismatch patterns in high-depth tRNA alignments correspond to specific modified nucleosides, and most predicted modifications were found in the 20th-40th and 50th-60th positions of the parental tRNA.

## Related tools

- **HAMR** (Predicts modified nucleoside sites from the BAM alignment using mismatch signatures and the eukaryotic tRNA modification model) — https://github.com/skbinfo/tncRNA-Toolkit
- **samtools** (Produces and handles the sorted BAM alignment of tncRNA-derived reads fed to HAMR)
- **tncRNA Toolkit** (Wraps HAMR (v1.2, bundled in util/) to compile retained modification calls into the per-tncRNA annotation table) — https://github.com/skbinfo/tncRNA-Toolkit

## Examples

```
python hamr.py IN.bam REF.fa models/euk_trna_mods.Rdata OUT ath 30 10 0.05 H4 0.01 0.05 0.05
```

## Evaluation signals

- Output table exists and contains one entry per tncRNA with modification annotations
- All retained modification sites map to mature tRNA or the 50 bp leader/trailer regions per the HAMR output and the pipeline's counting rule
- No sites were called at read depths below 30 or mismatch fractions below 10% (HAMR parameter check)
- Predicted modification types are restricted to those in the euk_trna_mods.Rdata model (e.g., m3C, pseudouridine, dihydrouridine, m2G)
- Modification position distribution is consistent with the article's finding that most calls fall in the 20th-40th and 50th-60th positions of the parental tRNA

## Limitations

- HAMR v1.2 requires python2, which may conflict with the python3-based rest of the toolkit
- Single-mismatch-tolerant alignment to overcome sequencing errors may cause base misidentification and inflate false negatives
- Only sites on mature tRNA and 50 bp leader/trailer flanks are retained; calls elsewhere are discarded
- How tRNA post-transcriptional modifications affect regulation of tRNA cleavage remains unclear, limiting biological interpretation

## Evidence

- [other] HAMR applied with eukaryotic tRNA model and thresholds: "python hamr.py IN.bam REF.fa models/euk_trna_mods.Rdata OUT ath 30 10 0.05 H4 0.01 0.05 0.05"
- [results] HAMR as modification prediction tool: "we utilized HAMR [48], a high utility tool for predicting modified nucleosides in high throughput sequencing datasets"
- [readme] HAMR bundled in toolkit and python2 requirement: "tar -xf HAMR-1.2.tar.gz"
- [intro] Modification site annotation as workflow step: "we have also included information related to the modified nucleoside sites detected along the individual tncRNAs"
