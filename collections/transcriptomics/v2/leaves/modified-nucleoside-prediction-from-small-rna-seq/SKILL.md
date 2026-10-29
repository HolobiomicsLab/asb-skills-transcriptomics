---
name: modified-nucleoside-prediction-from-small-rna-seq
description: Use when quality-filtered, adapter-trimmed small RNA-seq reads have already
  been classified as tncRNAs (mapped exclusively to artificial tRNA chromosomes with
  50 nt leader/trailer) and you need to annotate modified nucleoside sites per tncRNA
  from the mismatch patterns.
when_to_use_negative:
- Upstream tncRNA identification and classification (tRF-5s, tRF-3s, tRF-1, tRNA halves)
  has not yet been performed — this skill operates on already-classified tncRNA alignments
- Input data is not small RNA-seq aligned to a tRNA-centric artificial genome; HAMR's
  tRNA modification model (euk_trna_mods.Rdata) assumes mismatch patterns on tRNA
  positions
- Read depth at candidate sites is below the minimum read depth of 30, since calls
  are not reliable under the H4 model parameters used
edam_operation: http://edamontology.org/operation_0294
edam_topics:
- http://edamontology.org/topic_0099
tools:
- name: HAMR
  role: Predicts modified nucleosides (m3C, pseudouridine, dihydrouridine, m2G, etc.)
    from mismatch patterns in the tncRNA-derived small RNA read alignments; run as
    hamr.py with the euk_trna_mods.Rdata model, min depth 30, 10% mismatch, FDR 0.05,
    H4 model
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: samtools
  role: Provides the sorted BAM alignment of tncRNA-derived small RNA reads to the
    artificial genome that serves as HAMR's input
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:85f54e366053744dea988bfbc22f47e69b6dcad5e4a8d1abb73666592ce77309
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
  edam_operation: http://edamontology.org/operation_0294
  edam_topics:
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# modified-nucleoside-prediction-from-small-rna-seq

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Predicts modified nucleosides (e.g., m3C, pseudouridine, dihydrouridine, m2G) on tRNA-derived non-coding RNAs (tncRNAs) by running HAMR on the sorted BAM of tncRNA-derived small RNA reads aligned to an artificial tRNA genome, then compiling per-tncRNA modification site annotations.

## When to use

Use when quality-filtered, adapter-trimmed small RNA-seq reads have already been classified as tncRNAs (mapped exclusively to artificial tRNA chromosomes with 50 nt leader/trailer) and you need to annotate modified nucleoside sites per tncRNA from the mismatch patterns.

## When NOT to use

- Upstream tncRNA identification and classification (tRF-5s, tRF-3s, tRF-1, tRNA halves) has not yet been performed — this skill operates on already-classified tncRNA alignments
- Input data is not small RNA-seq aligned to a tRNA-centric artificial genome; HAMR's tRNA modification model (euk_trna_mods.Rdata) assumes mismatch patterns on tRNA positions
- Read depth at candidate sites is below the minimum read depth of 30, since calls are not reliable under the H4 model parameters used

## Inputs

- Sorted BAM of tncRNA-derived small RNA reads aligned to the artificial genome (from bowtie/samtools upstream)
- Reference FASTA of the artificial genome (tRNA gene regions with 50 nt leader and trailer flanks)
- HAMR eukaryotic tRNA modification model file (models/euk_trna_mods.Rdata, shipped with HAMR v1.2)

## Outputs

- Per-tncRNA annotation table listing each tncRNA with its detected modification sites and modification types (e.g., m3C, pseudouridine, dihydrouridine, m2G)
- HAMR prediction output with modified-nucleoside calls at specific positions on the parental tRNAs

## How to apply

First generate the upstream inputs: align processed single-end sRNA reads to the bowtie index of the artificial genome (tRNA genes masked with 50 nt upstream/downstream) and produce a sorted BAM plus reference FASTA with bowtie and samtools. Then run HAMR v1.2 with the eukaryotic tRNA modification model file (models/euk_trna_mods.Rdata) using the parameters: minimum read depth 30, minimum 10% mismatch, FDR 0.05, H4 model, 0.01, 0.05, 0.05. Restrict reported modification sites to positions falling on mature tRNA and 50 bp leader/trailer flank regions, discarding calls elsewhere. Finally compile a per-tncRNA annotation table listing each tncRNA with its detected modification sites and modification types; note HAMR-1.2 requires python2 to run and is executed from the 'util' directory of the tncRNA Toolkit.

## Related tools

- **HAMR** (Predicts modified nucleosides (m3C, pseudouridine, dihydrouridine, m2G, etc.) from mismatch patterns in the tncRNA-derived small RNA read alignments; run as hamr.py with the euk_trna_mods.Rdata model, min depth 30, 10% mismatch, FDR 0.05, H4 model) — https://github.com/skbinfo/tncRNA-Toolkit
- **samtools** (Provides the sorted BAM alignment of tncRNA-derived small RNA reads to the artificial genome that serves as HAMR's input)

## Examples

```
python hamr.py IN.bam REF.fa models/euk_trna_mods.Rdata OUT ath 30 10 0.05 H4 0.01 0.05 0.05
```

## Evaluation signals

- Output annotation table exists and contains one entry per tncRNA with modification annotations
- All retained modification sites map to mature tRNA or 50 bp leader/trailer regions per the HAMR output and the pipeline's counting rule
- Every retained call satisfies the HAMR parameter thresholds: minimum read depth 30, minimum 10% mismatch, FDR 0.05, H4 model
- Predicted modifications predominantly fall in the 20th-40th and 50th-60th positions of the parental tRNA, matching the paper's reported distribution
- Detected modification types are consistent with the eukaryotic tRNA modification model (e.g., m3C, pseudouridine, dihydrouridine, m2G)

## Limitations

- HAMR-1.2 requires python2 for running, which may conflict with the python3-based pipeline environment
- Allowing a single mismatch during alignment to overcome sequencing errors may lead to base misidentification and inflation of false negatives, affecting modification calling
- Conventional RNA-seq rather than tRNA-seq introduces modification biases in tncRNA identification; switching to tRNA-seq would be advantageous
- How tRNA post-transcriptional modifications affect regulation of tRNA cleavage remains unclear; predicted sites are not experimentally validated
- Modification sites outside the mature tRNA and 50 bp leader/trailer flanks are discarded and not reported

## Evidence

- [readme] HAMR version and runtime requirement from the toolkit dependencies: [evidence span withheld — anchor status: not_found]
