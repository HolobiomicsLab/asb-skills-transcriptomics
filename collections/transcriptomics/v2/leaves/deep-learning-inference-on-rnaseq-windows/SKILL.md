---
name: deep-learning-inference-on-rnaseq-windows
description: Use when RNAseq BAM files have been processed into tabix-indexed REDItools
  outTables and per-site classification of A-to-G substitutions as editing vs SNP
  is needed without paired DNA sequencing data.
when_to_use_negative:
- The analysis goal is de novo variant calling on genomic (DNAseq) data rather than
  A-to-G classification in transcriptome data — REDInet consumes RNAseq nucleotide-frequency
  windows only
- The input is raw FASTQ or unaligned BAM — REDItools pileup outTables must be generated
  and tabix-indexed first, per the README pipeline
- Sites fall outside regions with adequate window coverage and the genome-build imputation
  path cannot fill missing flanking nucleotides — sites with missing nucleotides in
  the 101-nt window are filtered out by design
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3474
tools:
- name: REDInet
  role: Temporal convolutional network package whose REDInet.h5 model and REDInet_Inference.py
    / REDInet_Inference_light_ver.py scripts perform the per-site classification from
    8 × 101 nucleotide-frequency windows
  repo: https://github.com/BioinfoUNIBA/REDInet.git
- name: REDItools
  role: Generates the RNAseq/DNAseq pileup outTables (REDItoolDnaRna.py at low stringency)
    that are the required tabix-indexed input to REDInet inference
  repo: https://github.com/BioinfoUNIBA/REDItools
- name: REDItools3
  role: Faster REDItools implementation (<60 min vs ~9 h for v1) that can produce
    the pileup outTables feeding REDInet inference
  repo: https://github.com/BioinfoUNIBA/REDItools3
- name: pysam
  role: Python SAM/BAM interface (wrapping samtools/tabix C APIs) used by the REDInet
    package to read alignments and indexed outTables during feature extraction
  repo: https://github.com/pysam-developers/pysam
provenance:
  source_task_ids:
  - task_006
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbaf107
    title: 'REDInet: a temporal convolutional network-based classifier for A-to-I
      RNA editing detection harnessing million known events'
  source_content_hash: sha256:6956329fe9129abcfcc71328f960a9aab38a6d563f481c542efa2af7f2ad48c6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/BioinfoUNIBA/REDInet
  repo_url: https://github.com/BioinfoUNIBA/REDInet.git
  tools:
  - Python
  - REDInet
  - REDItools
  - REDItools3
  - TensorFlow
  - numpy
  - pandas
  - pysam
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# deep-learning-inference-on-rnaseq-windows

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run the pretrained REDInet temporal convolutional network (TensorFlow, REDInet.h5) on tabix-indexed REDItools outTables to classify A-to-G substitutions as bona fide A-to-I RNA editing events using 8 × 101 nucleotide-frequency matrices from 101-base RNAseq windows, without requiring coupled WGS/WES data.

## When to use

Use when RNAseq BAM files have been processed into tabix-indexed REDItools outTables and per-site classification of A-to-G substitutions as editing vs SNP is needed without paired DNA sequencing data.

## When NOT to use

- The analysis goal is de novo variant calling on genomic (DNAseq) data rather than A-to-G classification in transcriptome data — REDInet consumes RNAseq nucleotide-frequency windows only
- The input is raw FASTQ or unaligned BAM — REDItools pileup outTables must be generated and tabix-indexed first, per the README pipeline
- Sites fall outside regions with adequate window coverage and the genome-build imputation path cannot fill missing flanking nucleotides — sites with missing nucleotides in the 101-nt window are filtered out by design

## Inputs

- Tabix-indexed, bgzip-compressed REDItools outTable (from REDItoolDnaRna.py or REDItools3) of variant/invariant sites
- Reference genome FASTA with samtools faidx index (GRCh37.primary_assembly.genome.fa or GRCh38.primary_assembly.genome.fa from GENCODE release 46)
- Assembly build specifier (GRCh37 or GRCh38)
- Pretrained REDInet temporal convolutional network model (REDInet.h5, bundled in the REDInet package)

## Outputs

- Per-site SNP/editing probabilities and predicted classes (softmax class 0 = SNP, 1 = editing site)
- REDInet_Inference.py 3-step output files: <.gz name>_discarded.txt (positions filtered on coverage/AG criteria), a file for positions filtered for missing nucleotides in the 101-nt window, and the final per-site predictions
- Optional per-site feature vectors (8 × 101 nucleotide-frequency matrices) for downstream perturbation analyses

## How to apply

First generate low-stringency REDItools DnaRna tables (e.g. REDItoolDnaRna.py with -t 40, -c 0,1, -m 0,255, -v 1, -q 0,30, -n 0.0, -N 0.0, -S) so complete 101-nt intervals are retrieved, bgzip-compress each outTable and index it with 'tabix -s 1 -b 2 -e 2'. Then launch REDInet_Inference.py (multiprocessing, with optional imputation of missing values via GRCh37/GRCh38 reference genomes) or the lighter REDInet_Inference_light_ver.py, supplying the outTable folder, assembly (GRCh37 or GRCh38), and optional thresholds: minimum coverage 50, minimum 3 A-to-G substitutions, minimum A-to-G substitution frequency 0.01. The script filters positions on coverage/AG-rate criteria, further filters sites with missing nucleotides in the 101-nucleotide window, extracts 8 × 101 nucleotide-frequency matrices, and runs the pretrained TCN (REDInet.h5) to output softmax SNP/editing probabilities per site. Rationale: REDInet was trained on ~13 million examples from 8404 GTEx RNAseq experiments and achieves accuracy >99% on held-out sets and >95% on external ground truth sets, so inference alone suffices to detect editing without empirical WGS/WES filtering.

## Related tools

- **REDInet** (Temporal convolutional network package whose REDInet.h5 model and REDInet_Inference.py / REDInet_Inference_light_ver.py scripts perform the per-site classification from 8 × 101 nucleotide-frequency windows) — https://github.com/BioinfoUNIBA/REDInet.git
- **REDItools** (Generates the RNAseq/DNAseq pileup outTables (REDItoolDnaRna.py at low stringency) that are the required tabix-indexed input to REDInet inference) — https://github.com/BioinfoUNIBA/REDItools
- **REDItools3** (Faster REDItools implementation (<60 min vs ~9 h for v1) that can produce the pileup outTables feeding REDInet inference) — https://github.com/BioinfoUNIBA/REDItools3
- **pysam** (Python SAM/BAM interface (wrapping samtools/tabix C APIs) used by the REDInet package to read alignments and indexed outTables during feature extraction) — https://github.com/pysam-developers/pysam

## Examples

```
python3 ../REDInet/Package/Utilities/REDInet_Inference.py --I <REDItools_outTable_folderpath> --O <output_folder> --A GRCh37 --C 50 --M 3 --N <outTable_name>
```

## Evaluation signals

- On the HEK293T KO-WT ground truth set (>20,000 bona fide events), predictions should reach accuracy >95%, specificity >93%, and precision >96%
- On the A549 external test set (>13,000 sites), mean accuracy should exceed 96%
- Positions failing minimum coverage (50), minimum A-to-G count (3), or minimum AG frequency (0.01) must appear in the <.gz name>_discarded.txt file and be absent from final predictions
- Per-site outputs must be paired softmax probabilities over the two classes (SNP vs editing); ground-truth comparison via scikit-learn should give low FNR/FPR
- Simulated missing-data robustness check: replacing >30% of flanking RNAseq frequencies with genomic reference frequencies should keep false-positive and false-negative rates below 10% (Fig. S4)

## Limitations

- Sites whose 101-nucleotide window is not fully covered by RNAseq reads are filtered out; imputation of missing values via the reference genome (hg38/hg19) is explicitly flagged as a BETA feature in the multiprocessing script
- Reference genomes and assemblies must match the annotation (GRCh37 or GRCh38 only); mismatched build will corrupt window extraction
- The multiprocessing REDInet_Inference.py should not be assumed stable for all genome builds — the lighter version is recommended for HTC-cluster parallelization or when imputation on different builds is needed
- Complete REDInet architecture description and feature-rescaling details are deferred to Supplementary Material, so internal model behavior is not fully documented in the main text

## Evidence
