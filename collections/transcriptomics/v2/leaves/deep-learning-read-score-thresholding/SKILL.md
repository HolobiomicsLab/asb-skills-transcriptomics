---
name: deep-learning-read-score-thresholding
description: Use when unmapped RNA-seq FASTQ reads must be filtered into viral candidates
  using a pretrained deep learning model, with scores above 0.7 as assembly seeds
  and average scores below 0.5 as extension stop criteria.
when_to_use_negative:
- Input reads are still mapped/aligned data or raw FASTQ including human-mapped reads
  — viRNAtrap expects unmapped reads extracted by the user beforehand
- The goal is read-level viral classification only, not contig assembly — the seed
  threshold alone suffices but the extension gating and contig output are unnecessary
- Sequences come from organisms other than human/viral — model predictions for such
  sequences are not defined per the paper's discussion
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3474
tools:
- name: viRNAtrap
  role: Alignment-free pipeline whose deep learning model scores unmapped reads to
    select viral assembly seeds and to gate contig extension
  repo: https://github.com/AuslanderLab/virnatrap.git
- name: TensorFlow
  role: Backend for the precomputed 48 bp read-scoring model applied to unmapped FASTQ
    reads
  repo: https://github.com/AuslanderLab/virnatrap
- name: keras
  role: Model format/framework in which the pretrained scoring model (model_lr_0.005_pool_5_emb_25_l2_0.02_64.hdf5)
    is loaded
  repo: https://github.com/fchollet/keras
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-36336-z
    title: A deep learning approach reveals unexplored landscape of viral expression
      in cancer
  source_content_hash: sha256:918f31154e03c30c9c06df16b5dafa053f7b3eab802d466c6d4041e21168928f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AuslanderLab/virnatrap.git
  tools:
  - TensorFlow
  - keras
  - viRNAtrap
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# deep-learning-read-score-thresholding

## Summary

Threshold deep-learning model scores assigned to fixed-length (48 bp) RNA-seq reads to select candidate viral reads for assembly seeds, and re-score extended contig regions to gate extension during contig growth in viRNAtrap.

## When to use

Use when unmapped RNA-seq FASTQ reads must be filtered into viral candidates using a pretrained deep learning model, with scores above 0.7 as assembly seeds and average scores below 0.5 as extension stop criteria.

## When NOT to use

- Input reads are still mapped/aligned data or raw FASTQ including human-mapped reads — viRNAtrap expects unmapped reads extracted by the user beforehand
- The goal is read-level viral classification only, not contig assembly — the seed threshold alone suffices but the extension gating and contig output are unnecessary
- Sequences come from organisms other than human/viral — model predictions for such sequences are not defined per the paper's discussion

## Inputs

- FASTQ file(s) of RNA-seq reads not mapped to the human genome (hg19) or phiX, named *_unmapped.fastq
- Precomputed Keras/TensorFlow model file (model/model_lr_0.005_pool_5_emb_25_l2_0.02_64.hdf5)

## Outputs

- FASTA-format contig files with .txt suffix, one per input FASTQ, with model score embedded in the header (e.g. >contig2[0.8009399])
- Set of seed reads selected by score > 0.7

## How to apply

Score each 48 bp read in the unmapped FASTQ with the precomputed Keras/TensorFlow model (model/model_lr_0.005_pool_5_emb_25_l2_0.02_64.hdf5) and keep reads with score above 0.7 as seeds for the iterative 24 bp k-mer overlap-extension assembly. During assembly, at every extension step recompute model scores over the extended region and stop extending in a direction once the average model score falls below 0.5. Concatenate left and right extensions into the final contig and embed the score in the FASTA header in brackets (e.g. >contig2[0.8009399]). The dual-threshold design concentrates expensive extension on high-confidence viral reads while preventing assembly from drifting into human or contaminant sequence as the model confidence decays.

## Related tools

- **viRNAtrap** (Alignment-free pipeline whose deep learning model scores unmapped reads to select viral assembly seeds and to gate contig extension) — https://github.com/AuslanderLab/virnatrap.git
- **TensorFlow** (Backend for the precomputed 48 bp read-scoring model applied to unmapped FASTQ reads) — https://github.com/AuslanderLab/virnatrap
- **keras** (Model format/framework in which the pretrained scoring model (model_lr_0.005_pool_5_emb_25_l2_0.02_64.hdf5) is loaded) — https://github.com/fchollet/keras

## Examples

```
virnatrap-predict --input input_fastq/ --output output_contigs/ --fastmode 1
```

## Evaluation signals

- diff of output_contigs/example_contigs.txt against expected_output/output_py.txt returns no differences, or only small differences in the less significant digits of bracketed scores (e.g. >contig2[0.8009399] vs >contig2[0.8009398])
- All emitted contig headers embed a score in brackets with a plausible value in [0,1]
- No seed read has model score ≤ 0.7 and no extension direction continued after its average score dropped below 0.5
- If an input FASTQ yields zero predicted viral reads, the corresponding output .txt file is created but empty (one-to-one input/output correspondence)
- Fast (C, compiled with gcc -O3 -shared -fPIC and called via ctypes) and slow (native Python) modes reproduce identical contigs

## Limitations

- virnatrap-predict will not overwrite pre-existing output files; previous outputs must be removed before rerunning
- Paired-end files must be concatenated into one FASTQ because viRNAtrap treats each input file as a distinct sample
- Model predictions for sequences from organisms other than human or viral are not defined; sensitivity is only guaranteed when more than five viral reads are present
- It is not possible to model all laboratory contaminants, so contaminant reads may receive high scores

## Evidence

- [readme] seed threshold: "There is a one-to-one correspondence between input files in directory input_fastq/ and output files in directory output_contigs (or whatever subdirectories the user specifies)."
- [readme] no-overwrite behavior: "If one wants to rerun the command with the same  input files and the same output_contigs/ output directory, one should first remove the previous output files."
- [readme] model architecture filename: "Building a Tensorflow model to predict whether reads of a fixed length come from viruses or not; this has been precomputed based on 48bp reads."
- [readme] paired-read concatenation: "the user is advised to concatenate the two files into one combined file for input to viRNAtrap because viRNAtrap treats each distinct input file as if it comes from a distinct sample."
- [intro] scores drive assembly: "we develop a framework, named viRNAtrap, that employs a deep learning model to accurately distinguish viral reads from RNA sequencing, and utilizes the model scores to assemble viral contigs"
- [intro] unmapped-read precondition: "The computational framework, named viRNAtrap, was then applied to unaligned RNA reads (to reduce the running time of viRNAtrap), to detect viral reads and assemble predicted viral contigs"
- [intro] training read length: "positive (viral) and negative (human) transcripts that were segmented into 48 bp fragments and divided into training and test sets"
- [discussion] sensitivity guarantee: "while not compromising sensitivity if more than five viral reads are present"
- [discussion] undefined predictions for other organisms: "Future work could train models to identify viruses from a variety of other organisms, and, with the viRNAtrap framework, achieve higher sensitivity for viral detection"
