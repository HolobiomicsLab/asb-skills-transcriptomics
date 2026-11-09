---
name: one-hot-genomic-sequence-encoding
description: Use when a sequence-based classifier (e.g., the InPACT CNN sequence module)
  requires fixed-length genomic windows converted to binary one-hot nucleotide tensors,
  and you need reproducible positive/negative training and held-out test sets for
  polyA site prediction.
when_to_use_negative:
- The candidate polyA site annotation file (InPACT_polyAsites.hg38.saf) is already
  provided — InPACT's site-identification and read modules can consume it directly
  without retraining the sequence module
- The goal is transcript abundance quantification or IPA usage estimation, which is
  handled by gffread/Salmon and InPACT_quantify, not by sequence encoding
- Held-out test sites overlap the RefSeq training data — evaluation would be contaminated
  and AUROCs inflated
edam_operation: http://edamontology.org/operation_3937
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: InPACT
  role: Hosts the CNN sequence module whose input is the one-hot encoded 201-nt windows;
    repo provides environment.yml, IPA site annotation, and pretrained workflow
  repo: https://github.com/YY-TMU/InPACT
- name: DeepPASS
  role: Benchmark method evaluated on the same one-hot-encoded constructed test sets
    for AUROC comparison
- name: APARENT
  role: Benchmark polyA prediction method evaluated on the same held-out test sets
- name: DeepPASTA
  role: Benchmark polyA prediction method evaluated on the same held-out test sets
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-46875-8
    title: 'InPACT: a computational method for accurate characterization of intronic
      polyadenylation from RNA sequencing data'
  source_content_hash: sha256:2d58655d2d12f6dc7ac059daa293575fcc0750b60a57f95ab904daff746cddb8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YY-TMU/InPACT
  tools:
  - APARENT
  - DeepPASS
  - DeepPASTA
  - InPACT
  edam_operation: http://edamontology.org/operation_3937
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# one-hot-genomic-sequence-encoding

## Summary

Encodes fixed-length genomic windows as binary nucleotide feature tensors (A=[1,0,0,0], T=[0,1,0,0], G=[0,0,1,0], C=[0,0,0,1]) so they can be fed into the InPACT CNN sequence module for polyA site prediction, benchmarked by AUROC on held-out GENCODE, PolyA_DB 3, and PolyASite 2.0 test sets.

## When to use

Use when a sequence-based classifier (e.g., the InPACT CNN sequence module) requires fixed-length genomic windows converted to binary one-hot nucleotide tensors, and you need reproducible positive/negative training and held-out test sets for polyA site prediction.

## When NOT to use

- The candidate polyA site annotation file (InPACT_polyAsites.hg38.saf) is already provided — InPACT's site-identification and read modules can consume it directly without retraining the sequence module
- The goal is transcript abundance quantification or IPA usage estimation, which is handled by gffread/Salmon and InPACT_quantify, not by sequence encoding
- Held-out test sites overlap the RefSeq training data — evaluation would be contaminated and AUROCs inflated

## Inputs

- RefSeq annotation GTF (e.g., hg38.109.20211119.ncbiRefSeq.gtf.gz from UCSC goldenPath) for polyA site positives
- Human reference genome (GRCh38) FASTA for extracting 201-nt windows
- PolyA site catalogs for held-out test sets: GENCODE, PolyA_DB 3, PolyASite 2.0

## Outputs

- One-hot encoded nucleotide tensors of shape 201x4 for each genomic window
- Trained CNN sequence module scoring candidate polyA sites
- AUROC values per held-out test database (0.954 GENCODE, 0.920 PolyA_DB 3, 0.794 PolyASite 2.0) recorded in sequence_module_auroc_results.csv

## How to apply

Extract 201-nt genomic windows centered on RefSeq-annotated polyA sites as positives and an equal number of 201-nt windows randomly sampled from intergenic regions as negatives, then one-hot encode each nucleotide (A=[1,0,0,0], T=[0,1,0,0], G=[0,0,1,0], C=[0,0,0,1]). Feed the encoded tensors into a CNN built from two convolution-ReLU-max-pooling pairs and one fully connected layer with dropout, tuning hyper-parameters on a validation set by random sampling. Construct separate held-out test sets from GENCODE, PolyA_DB 3, and PolyASite 2.0 polyA sites, explicitly excluding any sites that overlap the training data to avoid leakage. Evaluate on each test set with AUROC and benchmark against DeepPASS, APARENT, and DeepPASTA on the same constructed test sets. The rationale is that the CNN learns contextual sequence motifs around cleavage sites directly from the binary encoding without hand-crafted features.

## Related tools

- **InPACT** (Hosts the CNN sequence module whose input is the one-hot encoded 201-nt windows; repo provides environment.yml, IPA site annotation, and pretrained workflow) — https://github.com/YY-TMU/InPACT
- **DeepPASS** (Benchmark method evaluated on the same one-hot-encoded constructed test sets for AUROC comparison)
- **APARENT** (Benchmark polyA prediction method evaluated on the same held-out test sets)
- **DeepPASTA** (Benchmark polyA prediction method evaluated on the same held-out test sets)

## Examples

```
import numpy as np; onehot = lambda s: np.array([[{'A':[1,0,0,0],'T':[0,1,0,0],'G':[0,0,1,0],'C':[0,0,0,1]}[c] for c in w] for w in windows201nt])
```

## Evaluation signals

- AUROC on held-out GENCODE test set reaches approximately 0.954; PolyA_DB 3 approximately 0.920; PolyASite 2.0 approximately 0.794
- Class balance invariant: number of negative intergenic windows equals number of positive RefSeq polyA site windows
- Overlap check: no GENCODE, PolyA_DB 3, or PolyASite 2.0 test site appears in the RefSeq-derived training set
- Every encoded window has exactly one '1' per position (valid one-hot row sums across 201 positions)
- CNN architecture check: two convolution-ReLU-max-pooling pairs plus one fully connected layer with dropout, selected by random-sampling hyper-parameter tuning on a validation set

## Limitations

- Performance is annotation-dependent: AUROC degrades from 0.954 (GENCODE) to 0.794 (PolyASite 2.0), reflecting differences in how polyA sites are cataloged across databases
- No version numbers were given for InPACT or any compared tool (DeepPASS, APARENT, DeepPASTA), complicating exact reproduction of the AUROC benchmarks
- No parameters or thresholds were reported for the comparative benchmarks in some sections, so exact training configuration may need to be recovered from the repository code

## Evidence

- [methods] CNN architecture and encoding: "The sequence module incorporates a convolutional neural network (CNN) with multiple convolutional and pooling layers"
- [methods] Held-out test databases: "evaluated on the polyA sites of GENCODE, PolyA_DB 3 and PolyASite 2.0"
- [other] One-hot encoding scheme: [evidence span withheld — anchor status: not_found]
- [other] AUROC results on held-out sets: "achieving AUROC values of 0.954 for GENCODE, 0.920 for PolyA_DB 3, and 0.794 for PolyASite 2.0"
- [readme] Readme: candidate site annotation consumed directly: "we provided an annotation of potential IPA sites predicted from the sequence module that could be used directly"
