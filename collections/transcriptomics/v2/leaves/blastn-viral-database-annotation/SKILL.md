---
name: blastn-viral-database-annotation
description: Use when you have predicted viral contigs (FASTA output from virnatrap-predict)
  and need to assign viral taxonomy per sample, e.g. to confirm HR-αHPV, HBV, or HCV
  detections in TCGA tumor types.
when_to_use_negative:
- Input is raw unmapped FASTQ reads that have not yet been scored and assembled by
  virnatrap-predict (virnatrap-predict with model_lr_0.005_pool_5_emb_25_l2_0.02_64.hdf5
  and assemble_read_c.so must run first)
- Sample sequences may come from organisms other than human/viral — the article states
  model predictions for such sequences are undefined and models for other organisms
  are future work
- Very low-viral-read samples (≤5 viral reads) where the framework does not guarantee
  sensitivity
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: blastn
  role: Compares assembled viral contigs against three curated viral databases with
    word size 15 and E-value 0.01 to assign viral taxonomy
- name: viRNAtrap
  role: 'Upstream producer of predicted viral contigs: scores unmapped reads with
    the 48 bp TensorFlow model and assembles them in fast mode (assemble_read_c.so,
    --fastmode 1)'
  repo: https://github.com/AuslanderLab/virnatrap.git
- name: blastx
  role: Used for targeted protein-level verification that multiple viral reads support
    the classification of divergent viruses
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-36336-z
    title: A deep learning approach reveals unexplored landscape of viral expression
      in cancer
  source_content_hash: sha256:355d967d596295dcd142814d55bdce4e2e6e76d5d9c6281ac8861e02f915705e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AuslanderLab/virnatrap.git
  tools:
  - NumPy
  - blastn
  - blastx
  - viRNAtrap
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# blastn-viral-database-annotation

## Summary

Annotate viRNAtrap-assembled viral contigs by aligning them with blastn against three curated viral databases to assign viral taxonomy per TCGA sample, enabling validation of known cancer-associated virus detections against clinical annotations.

## When to use

Use when you have predicted viral contigs (FASTA output from virnatrap-predict) and need to assign viral taxonomy per sample, e.g. to confirm HR-αHPV, HBV, or HCV detections in TCGA tumor types.

## When NOT to use

- Input is raw unmapped FASTQ reads that have not yet been scored and assembled by virnatrap-predict (virnatrap-predict with model_lr_0.005_pool_5_emb_25_l2_0.02_64.hdf5 and assemble_read_c.so must run first)
- Sample sequences may come from organisms other than human/viral — the article states model predictions for such sequences are undefined and models for other organisms are future work
- Very low-viral-read samples (≤5 viral reads) where the framework does not guarantee sensitivity

## Inputs

- Per-sample FASTA contig files produced by virnatrap-predict (output suffix .txt in FASTA format), one per input *_unmapped.fastq file
- Three curated viral reference databases for blastn (including HR-αHPV, HBV, HCV references)
- TCGA clinical annotations for CESC, HNSC, and LIHC (for TPR validation)

## Outputs

- Per-sample viral taxonomy assignments from blastn hits
- Per-tumor-type detection counts for CESC, HNSC, and LIHC
- Computed true positive rate versus TCGA clinical annotations (expected > 95%)

## How to apply

After viRNAtrap (deep learning model + C-library fast-mode assembly) produces per-sample FASTA contig files, run blastn with word size 15 and E-value 0.01 to compare the assembled contigs against three curated viral databases. Aggregate the hits to assign viral taxonomy per sample and roll up per-sample calls into per-tumor-type detection counts (CESC, HNSC, LIHC). Match detected virus-sample calls against TCGA clinical annotations to compute the true positive rate; acceptance requires TPR > 95%, consistent with the Supplementary Fig. 1 pattern. Note the viRNAtrap README states viRNAtrap-predict will not overwrite pre-existing output files, so regenerate contigs before re-annotation.

## Related tools

- **blastn** (Compares assembled viral contigs against three curated viral databases with word size 15 and E-value 0.01 to assign viral taxonomy)
- **viRNAtrap** (Upstream producer of predicted viral contigs: scores unmapped reads with the 48 bp TensorFlow model and assembles them in fast mode (assemble_read_c.so, --fastmode 1)) — https://github.com/AuslanderLab/virnatrap.git
- **blastx** (Used for targeted protein-level verification that multiple viral reads support the classification of divergent viruses)

## Examples

```
blastn -query output_contigs/sample_contigs.txt -db curated_viral_dbs -word_size 15 -evalue 0.01
```

## Evaluation signals

- Generated per-sample FASTA contig files exist in the output directory for every input *_unmapped.fastq file
- Known virus counts reproduced: HR-αHPV in 288 CESC and 61 HNSC samples, HBV in 85 LIHC samples, HCV in 13 LIHC samples
- True positive rate exceeds 95% when matched against TCGA clinical annotations
- Detection pattern is consistent with Supplementary Fig. 1
- Divergent virus calls are supported by multiple viral reads verified via targeted blastn/blastx analyses

## Limitations

- Sensitivity is only stated for cases where more than five viral reads are present
- All laboratory contaminants that may have infected samples cannot be modeled
- blastn against the curated nucleotide databases assigns taxonomy from assembled contigs; divergent/divergent-classified viruses additionally require targeted blastn- and blastx-based verification

## Evidence

- [intro] blastn post-processing of contigs: "in post-processing analysis, we used blastn24 to compare the assembled viral contigs to three curated viral databases"
- [discussion] aligning contigs to viral databases of interest: "The contigs detected by viRNAtrap can be aligned to different viral databases, as we demonstrate in this study, to rapidly identify viral expressions of interest"
- [discussion] verification of divergent virus calls: "the presence and classification of multiple viral reads was verified by targeted blastn- and blastx-based sequence analyses in different samples"
- [readme] viRNAtrap output format and one-to-one file correspondence: "The output files are in FASTA format but have the suffix .txt"
- [other] blastn parameters for taxonomy assignment: [evidence span withheld — anchor status: not_found]
