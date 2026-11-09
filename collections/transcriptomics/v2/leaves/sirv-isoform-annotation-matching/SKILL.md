---
name: sirv-isoform-annotation-matching
description: Use when benchmarking isoform-detection tools on a long-read dataset
  with a known ground-truth isoform set (e.g., PacBio SIRV reads) and you need to
  count how many of the 68 known SIRV isoforms each tool correctly detects.
when_to_use_negative:
- When the sample has no known isoform ground truth (e.g., a novel transcriptome without
  a SIRV-like spike-in annotation) — there is no annotation to match against, so counts
  of correct detections cannot be computed.
- When the goal is differential expression or quantification accuracy rather than
  isoform detection benchmarking — this skill compares detected isoform structures
  against an annotation, not abundance estimates.
- When isoforms have already been curated/edited manually in VIsoQLR — comparing those
  edited isoforms against the original benchmark annotation would not reproduce the
  reported automatic-analysis figures.
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0196
tools:
- name: VIsoQLR
  role: Isoform detection and quantification tool whose output isoforms are compared
    against known SIRV isoform annotation; run via Docker with default parameters
    (2% CEC read threshold)
  repo: https://github.com/TBLabFJD/VIsoQLR
- name: FLAIR
  role: Isoform detection tool benchmarked against SIRV annotation on the same minimap2
    BAM alignment
- name: StringTie2
  role: Transcript assembly tool benchmarked against SIRV annotation; its detected
    isoforms are compared for boundary-extension artifacts
- name: Minimap2
  role: Sequence mapper producing the BAM alignment used by all three tools for isoform
    detection prior to annotation matching
  repo: https://github.com/lh3/minimap2
- name: VIsoQLR_benchmark
  role: Repository of scripts (isoform_analysis.sh, isoform_comparison.R) used for
    data preprocessing, isoform calling and isoform comparison
  repo: https://github.com/TBLabFJD/VIsoQLR_benchmark
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s00439-023-02539-z
    title: 'VIsoQLR: an interactive tool for the detection, quantification and fine-tuning
      of isoforms in selected genes using long-read sequencing'
  source_content_hash: sha256:578fe1631f79f124b78a671b5cc8034cdb17a8eed7e14249f353f1ef50f41e9c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TBLabFJD/VIsoQLR
  tools:
  - FLAIR
  - Minimap2
  - StringTie2
  - VIsoQLR
  - VIsoQLR_benchmark
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# sirv-isoform-annotation-matching

## Summary

Compares isoforms detected by long-read transcript tools (VIsoQLR, FLAIR, StringTie2) against the known SIRV isoform annotation, counting correct detections (e.g., VIsoQLR 48/68 = 71%, FLAIR 35 = 51%, StringTie2 15 = 22%) and verifying exon coordinate accuracy to benchmark tool performance.

## When to use

Use when benchmarking isoform-detection tools on a long-read dataset with a known ground-truth isoform set (e.g., PacBio SIRV reads) and you need to count how many of the 68 known SIRV isoforms each tool correctly detects.

## When NOT to use

- When the sample has no known isoform ground truth (e.g., a novel transcriptome without a SIRV-like spike-in annotation) — there is no annotation to match against, so counts of correct detections cannot be computed.
- When the goal is differential expression or quantification accuracy rather than isoform detection benchmarking — this skill compares detected isoform structures against an annotation, not abundance estimates.
- When isoforms have already been curated/edited manually in VIsoQLR — comparing those edited isoforms against the original benchmark annotation would not reproduce the reported automatic-analysis figures.

## Inputs

- Detected isoform sets per tool (exon-coordinate isoforms from VIsoQLR, FLAIR, StringTie2 runs on the same BAM alignment)
- Known SIRV isoform annotation (reference ground truth of 68 isoforms)
- PacBio SIRV reads aligned to reference (BAM, e.g., from Minimap2 '-ax splice:hq -uf --secondary=no' or GMAP)

## Outputs

- benchmark_detection_counts.csv — per-tool detection counts vs known SIRV isoforms (VIsoQLR 48/68 = 71%, FLAIR 35 = 51%, StringTie2 15 = 22%)
- Per-tool exon coordinate accuracy metrics (e.g., VIsoQLR 99% exon coordinate accuracy)
- Isoform comparison report distinguishing correctly detected, missed, and artifact isoforms (e.g., StringTie2 boundary extensions absent from BAM files)

## How to apply

After aligning SIRV long reads to the reference (with GMAP or Minimap2 using '-ax splice:hq -uf --secondary=no') and running each tool (VIsoQLR via its Docker container with default parameters including the 2% CEC read threshold; FLAIR and StringTie2 with default parameters on the same BAM), convert each tool's detected isoform set to exon-coordinate form and compare it against the known SIRV isoform annotation. Count a detection as correct when the tool's isoform matches a known SIRV isoform with matching exon coordinates (VIsoQLR achieves 99% exon coordinate accuracy); tabulate the per-tool detection counts (VIsoQLR 48/68 = 71%, FLAIR 35 = 51%, StringTie2 15 = 22%) into benchmark_detection_counts.csv. The comparison script is `isoform_comparison.R` from the TBLabFJD/VIsoQLR_benchmark repository. Validation: accept the result if reproduced counts match the reported benchmark figures — VIsoQLR detecting ~72% of known SIRV isoforms with ~99% exon coordinate accuracy and outperforming FLAIR and StringTie2.

## Related tools

- **VIsoQLR** (Isoform detection and quantification tool whose output isoforms are compared against known SIRV isoform annotation; run via Docker with default parameters (2% CEC read threshold)) — https://github.com/TBLabFJD/VIsoQLR
- **FLAIR** (Isoform detection tool benchmarked against SIRV annotation on the same minimap2 BAM alignment)
- **StringTie2** (Transcript assembly tool benchmarked against SIRV annotation; its detected isoforms are compared for boundary-extension artifacts)
- **Minimap2** (Sequence mapper producing the BAM alignment used by all three tools for isoform detection prior to annotation matching) — https://github.com/lh3/minimap2
- **VIsoQLR_benchmark** (Repository of scripts (isoform_analysis.sh, isoform_comparison.R) used for data preprocessing, isoform calling and isoform comparison) — https://github.com/TBLabFJD/VIsoQLR_benchmark

## Examples

```
Rscript isoform_comparison.R
```

## Evaluation signals

- Reproduced detection counts match reported benchmark: VIsoQLR 48/68 (71%), FLAIR 35/68 (51%), StringTie2 15/68 (22%)
- VIsoQLR achieves ~99% exon coordinate accuracy on detected isoforms
- GMAP-based and minimap2-based mappings yield similar per-tool detection counts (mapper-invariant result)
- Missed isoforms fall mostly below the default 2% CEC read threshold, consistent with VIsoQLR's discussion of threshold-driven misses
- No StringTie2 boundary-extension artifact is present in the input BAM files, so any such extension in StringTie2 output is attributable to the tool, not the alignment

## Limitations

- VIsoQLR's automatic analysis misses isoforms whose reads fall below the default 2% CEC read threshold; detecting them requires manual parameter tuning, which changes the comparison outcome
- StringTie2 extends the boundaries of some isoforms and does not call isoforms lacking certain exons; these artifacts must be accounted for when matching against the annotation
- Small-abundance isoforms (e.g., Iso5 at 4% and Iso8 at 1.7%) are hard to distinguish from noise in electropherograms, so detection counts for low-abundance isoforms are less reliable
- No software version numbers were reported for VIsoQLR, StringTie2, or FLAIR, limiting exact reproducibility of the comparison

## Evidence

- [results] VIsoQLR detected 48 out of 68 isoforms (71%), FLAIR 35 (51%), and StringTie2 15 (22%).: [evidence span withheld — anchor status: not_found]
- [discussion] VIsoQLR detects with 99% accuracy of exon coordinates 72% of the known isoforms using SIRVs: "VIsoQLR detects with 99% accuracy of exon coordinates 72% of the known isoforms using SIRVs, outperforming the other tools."
- [methods] docker run -it -p 8888:8888 tblabfjd/visoqlr:latest: "docker run -it -p 8888:8888 tblabfjd/visoqlr:latest"
- [discussion] most missing isoforms were below 2% (default value): "most missing isoforms were below 2% (default value). At the same time, lowering this threshold has the cost of detecting more isoforms"
- [discussion] it extends the boundaries of some isoforms, and it does not call isoforms without certain exons: "it extends the boundaries of some isoforms, and it does not call isoforms without certain exons"
- [discussion] This extension (also seen in the SIRVs analysis) seems to be an artifact as it is not present in BAM files: "This extension (also seen in the SIRVs analysis) seems to be an artifact as it is not present in BAM files used by StringTie2 and VIsoQLR"
- [readme] This repostory contains the scripts used for data preprocessing, isoform calling and isoform comparison of VIsoQLR, FLAIR and StringTie2.: [evidence span withheld — anchor status: external_doc]
- [results] Align the SIRV long reads to the reference FASTA with Minimap2 using the high-quality splice-aware preset: [evidence span withheld — anchor status: not_found]
