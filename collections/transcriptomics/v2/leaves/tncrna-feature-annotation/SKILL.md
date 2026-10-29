---
name: tncrna-feature-annotation
description: Use when small RNA-seq reads have been mapped to a tRNA-masked artificial
  genome (BAM + reference FASTA available) and per-tncRNA nucleoside modification
  annotations (e.g. m3C, pseudouridine, dihydrouridine, m2G) are needed.
when_to_use_negative:
- When the goal is to detect or quantify tncRNAs themselves rather than annotate their
  modification sites — use the upstream tncRNA prediction step of the tncRNA Toolkit
  (tRNAscan-SE + bowtie1 artificial-genome mapping) instead.
- When reads have not been mapped to the tRNA-masked artificial genome with 50 nt
  flanks; modification calls would not be restricted to tRNA plus leader/trailer contexts.
- When sequencing data is conventional RNA-seq rather than small RNA-seq — the article
  notes switching to tRNA-seq is needed to overcome modification biases, so calls
  from biased RNA-seq would be unreliable.
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3170
tools:
- name: HAMR
  role: Predicts modified nucleosides (e.g. m3C, pseudouridine, dihydrouridine, m2G)
    in high-throughput sequencing data using the eukaryotic tRNA modification model
    with depth 30, 10% mismatch, FDR 0.05, H4 model
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: samtools
  role: Processes and sorts the bowtie1 BAM alignments of tncRNA-derived reads that
    serve as HAMR input
  repo: https://github.com/skbinfo/tncRNA-Toolkit
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:592ff9771d051aabddede90761d725038d2130de9c94310f02c191a230361626
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
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# tncrna-feature-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Predicts modified nucleoside sites on tRNA-derived non-coding RNAs (tncRNAs) using HAMR v1.2 with the eukaryotic tRNA modification model, retaining only calls on mature tRNA bodies and 50 bp leader/trailer flanks.

## When to use

Use when small RNA-seq reads have been mapped to a tRNA-masked artificial genome (BAM + reference FASTA available) and per-tncRNA nucleoside modification annotations (e.g. m3C, pseudouridine, dihydrouridine, m2G) are needed.

## When NOT to use

- When the goal is to detect or quantify tncRNAs themselves rather than annotate their modification sites — use the upstream tncRNA prediction step of the tncRNA Toolkit (tRNAscan-SE + bowtie1 artificial-genome mapping) instead.
- When reads have not been mapped to the tRNA-masked artificial genome with 50 nt flanks; modification calls would not be restricted to tRNA plus leader/trailer contexts.
- When sequencing data is conventional RNA-seq rather than small RNA-seq — the article notes switching to tRNA-seq is needed to overcome modification biases, so calls from biased RNA-seq would be unreliable.

## Inputs

- Sorted BAM alignment of tncRNA-derived small RNA reads to the tRNA-masked artificial genome (bowtie1 output, processed via samtools)
- Reference FASTA of the artificial genome (tRNA genes plus 50 nt upstream/downstream flanks as artificial chromosomes)
- HAMR eukaryotic tRNA modification model file (models/euk_trna_mods.Rdata, shipped with HAMR v1.2)

## Outputs

- HAMR modification-site predictions per tncRNA (modified nucleoside type and position)
- Per-tncRNA annotation table listing each tncRNA with its detected modification sites and modification types, restricted to mature tRNA and 50 bp leader/trailer regions

## How to apply

Load the sorted BAM of tncRNA-derived small RNA reads aligned to the artificial genome (produced upstream with bowtie1 '-v2 --best' and samtools) plus the corresponding reference FASTA. Run HAMR v1.2 with models/euk_trna_mods.Rdata and the parameters: minimum read depth 30, minimum 10% mismatch, FDR 0.05, model H4, followed by 0.01 0.05 0.05, exactly as in 'python hamr.py IN.bam REF.fa models/euk_trna_mods.Rdata OUT ath 30 10 0.05 H4 0.01 0.05 0.05'. Restrict reported modification sites to positions falling on mature tRNA or the 50 bp leader/trailer flank regions, discarding calls elsewhere per the pipeline's counting rule. Compile the retained calls into a per-tncRNA annotation table listing each tncRNA with its modification sites and modification types. Note HAMR-1.2 requires python2 and is bundled in the toolkit's util/ directory.

## Related tools

- **HAMR** (Predicts modified nucleosides (e.g. m3C, pseudouridine, dihydrouridine, m2G) in high-throughput sequencing data using the eukaryotic tRNA modification model with depth 30, 10% mismatch, FDR 0.05, H4 model) — https://github.com/skbinfo/tncRNA-Toolkit
- **samtools** (Processes and sorts the bowtie1 BAM alignments of tncRNA-derived reads that serve as HAMR input) — https://github.com/skbinfo/tncRNA-Toolkit

## Examples

```
python hamr.py IN.bam REF.fa models/euk_trna_mods.Rdata OUT ath 30 10 0.05 H4 0.01 0.05 0.05
```

## Evaluation signals

- The HAMR command completes with the euk_trna_mods.Rdata model and the exact parameter string 'ath 30 10 0.05 H4 0.01 0.05 0.05' without error
- The output annotation table exists and contains one entry per tncRNA with modification annotations
- All retained modification sites map to mature tRNA or 50 bp leader/trailer regions per the HAMR output and the pipeline's counting rule
- Predicted modification types are restricted to types in the eukaryotic tRNA model (e.g. m3C, pseudouridine, dihydrouridine, m2G)
- Most predicted modifications fall in the 20th-40th and 50th-60th positions of the parental tRNA, consistent with reported findings

## Limitations

- Allowing a single mismatch during alignment to tolerate sequencing errors may cause base misidentification and inflate false negatives
- HAMR-1.2 requires python2 to run, despite the toolkit's python3-based pipeline
- Conventional RNA-seq carries modification biases; the article recommends switching to tRNA-seq to overcome them
- Modification calls outside the mature tRNA body and 50 bp leader/trailer flanks are discarded, so flanks beyond 50 nt are unannotated

## Evidence

- [methods] HAMR was applied to the identified tncRNAs to predict modified nucleoside sites (e.g., m3C, pseudouridine, dihydrouridine, m2G): [evidence span withheld — anchor status: not_found]
