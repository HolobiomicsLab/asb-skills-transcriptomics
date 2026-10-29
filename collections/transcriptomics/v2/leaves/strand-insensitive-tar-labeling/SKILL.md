---
name: strand-insensitive-tar-labeling
description: Use when you have groHMM HMM bed features (chr*_HMM.bed) and a refFlat
  gene annotation, and need a per-TAR aTAR/uTAR label based on genomic start/stop
  overlap regardless of strand.
when_to_use_negative:
- Your downstream analysis requires antisense-overlapping TARs to be called uTARs
  — use the strand-aware variant described in the paper instead
- You do not yet have groHMM bed files (chr*_HMM.bed); the labeling rule operates
  on already-called TAR features, not raw reads or BAMs
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: groHMM
  role: Upstream producer of the TAR HMM bed features (chr*_HMM.bed) that this labeling
    rule annotates; must be installed in R before the workflow rules run
  repo: https://bioconductor.org/packages/release/bioc/html/groHMM.html
- name: rtracklayer
  role: R package used to load/overlap the HMM bed features with the refFlat gene
    annotation for aTAR/uTAR labeling
  repo: https://bioconductor.org/packages/release/bioc/html/rtracklayer.html
- name: Snakemake
  role: Workflow engine wrapping the labeling steps (e.g., rule calcHMMrefFlat) and
    emitting the labeled refFlat/GTF outputs
  repo: https://github.com/fw262/TAR-scRNA-seq
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-22496-3
    title: Uncovering transcriptional dark matter via gene annotation independent
      single-cell RNA sequencing analysis
  source_content_hash: sha256:61920d2e28b0083ef71db206665ea39a671880e3b89e115d8bbea96bbfa5929f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Snakemake
  - groHMM
  - rtracklayer
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# strand-insensitive-tar-labeling

## Summary

Labels each groHMM-derived TAR (chr*_HMM.bed feature) as aTAR or uTAR by testing genomic coordinate overlap against the refFlat gene annotation while deliberately ignoring strand, so antisense-overlapping TARs are still counted as annotated. This yields a binary annotation-overlap label used to separate annotated from unannotated (dark-matter) transcription in the TAR-scRNA-seq workflow.

## When to use

Use when you have groHMM HMM bed features (chr*_HMM.bed) and a refFlat gene annotation, and need a per-TAR aTAR/uTAR label based on genomic start/stop overlap regardless of strand.

## When NOT to use

- Your downstream analysis requires antisense-overlapping TARs to be called uTARs — use the strand-aware variant described in the paper instead
- You do not yet have groHMM bed files (chr*_HMM.bed); the labeling rule operates on already-called TAR features, not raw reads or BAMs

## Inputs

- groHMM-derived TAR feature bed files (chr*_HMM.bed) from the groHMM HMM step
- Reference gene annotation in refFlat format

## Outputs

- Labeled TAR annotation in refFlat format, with each TAR carrying exactly one of the labels 'aTAR' or 'uTAR'
- Labeled TAR annotation converted to GTF format using the custom script from github.com/fw262/TAR-scRNA-seq

## How to apply

Load the groHMM-derived TAR feature set (chr*_HMM.bed files produced by the groHMM step) and the reference gene annotation in refFlat format using R with rtracklayer. For each HMM feature, test for overlap with any existing gene annotation transcript strictly on genomic start/stop coordinates, deliberately ignoring strandedness so that antisense-overlapping features are still counted as overlapping (note the paper also describes an extended strand-aware variant that would treat antisense overlaps as uTARs — this skill is the strand-insensitive rule). Label features with any overlap as 'aTAR' and all remaining features as 'uTAR'. Write the labeled TAR annotation in refFlat format, then use the custom conversion script from the TAR-scRNA-seq repository to emit the labeled annotation in GTF format for downstream single-cell TAR expression analyses.

## Related tools

- **groHMM** (Upstream producer of the TAR HMM bed features (chr*_HMM.bed) that this labeling rule annotates; must be installed in R before the workflow rules run) — https://bioconductor.org/packages/release/bioc/html/groHMM.html
- **rtracklayer** (R package used to load/overlap the HMM bed features with the refFlat gene annotation for aTAR/uTAR labeling) — https://bioconductor.org/packages/release/bioc/html/rtracklayer.html
- **Snakemake** (Workflow engine wrapping the labeling steps (e.g., rule calcHMMrefFlat) and emitting the labeled refFlat/GTF outputs) — https://github.com/fw262/TAR-scRNA-seq

## Examples

```
snakemake --cores 1 calcHMMrefFlat
```

## Evaluation signals

- The labeled GTF file exists and parses as valid GTF
- Every TAR record in the labeled annotation carries exactly one of the two labels (aTAR or uTAR), never both or neither
- Each label is consistent with the overlap rule: TARs with any start/stop overlap against the refFlat annotation are aTARs, TARs entirely outside annotations are uTARs (antisense overlaps retained as aTAR under this strand-insensitive rule)
- Downstream 'cannot read: chr*_HMM.bed: No such file or directory' errors in SingleCellHMM_Run_combined_bam_HMM_features.log must be absent, confirming groHMM bed inputs were produced before labeling

## Limitations

- Requires groHMM and rtracklayer to be manually installed in R; failure to load them causes the rule calcHMMrefFlat to fail with 'no lines available in input' when chr*_HMM.bed is missing
- Ignoring strandedness means antisense-overlapping TARs are labeled aTAR; the paper's extended strand-aware definition would classify those as uTARs, so this rule may undercount unannotated transcription
- Labeling reflects only overlap with the existing annotation, so aTAR/uTAR calls are only as good as the (possibly incomplete) reference annotation

## Evidence

- [other] TARs completely outside gene annotations are labeled uTARs, while TARs overlapping annotations are labeled aTARs: [evidence span withheld — anchor status: not_found]
- [other] ignoring strandedness so that antisense-overlapping features are still counted: [evidence span withheld — anchor status: not_found]
- [other] strand-aware definition treats antisense as uTAR: [evidence span withheld — anchor status: not_found]
- [readme] README manual install note: "Please manually install [groHMM](https://bioconductor.org/packages/release/bioc/html/groHMM.html) and [rtracklayer](https://bioconductor.org/packages/release/bioc/html/rtracklayer.html) and make sure"
- [intro] groHMM HMM state division: "groHMM uses a hidden Markov model to divide the genome into two states (transcribed and untranscribed) based on read coverage in each bin"
