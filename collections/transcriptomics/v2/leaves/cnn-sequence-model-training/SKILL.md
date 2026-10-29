---
name: cnn-sequence-model-training
description: Use when a candidate list of polyA/IPA sites must be scored purely from
  nucleotide sequence context, or when a CNN classifier for polyA sites must be trained
  and benchmarked with AUROC.
when_to_use_negative:
- Candidate sites must be distinguished using RNA-seq read alignment evidence (spliced/spanning
  read counts, coverage patterns) — that is InPACT's read module with its random-forest
  classifier, not the CNN sequence module
- The task is quantifying IPA isoform usage or transcript abundance — use the Salmon-based
  quantification step instead
- The data are 3'-end or long-read sequencing benchmarks for comparing IPA-identification
  tools like IPAFinder or APAIQ, rather than training a sequence-based classifier
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: InPACT
  role: Hosts the CNN sequence module; its predicted polyA-site annotation (InPACT_polyAsites.hg38.saf)
    is the downstream consumer of the trained model
  repo: https://github.com/YY-TMU/InPACT
- name: DeepPASS
  role: Benchmark polyA-site predictor evaluated on the same constructed held-out
    test sets
- name: APARENT
  role: Benchmark polyA-site predictor evaluated on the same constructed held-out
    test sets
- name: DeepPASTA
  role: Benchmark polyA-site predictor evaluated on the same constructed held-out
    test sets
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-46875-8
    title: 'InPACT: a computational method for accurate characterization of intronic
      polyadenylation from RNA sequencing data'
  source_content_hash: sha256:0f160b5d70851d2395b5d30838cbc12bbeb7727af3bba22544aa74148627276c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YY-TMU/InPACT
  tools:
  - APARENT
  - DeepPASS
  - DeepPASTA
  - InPACT
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# cnn-sequence-model-training

## Summary

Train a convolutional neural network (CNN) on one-hot-encoded 201-nt genomic windows to score candidate polyA sites, as done in InPACT's sequence module, and benchmark it by AUROC on held-out GENCODE, PolyA_DB 3, and PolyASite 2.0 test sets.

## When to use

Use when a candidate list of polyA/IPA sites must be scored purely from nucleotide sequence context, or when a CNN classifier for polyA sites must be trained and benchmarked with AUROC.

## When NOT to use

- Candidate sites must be distinguished using RNA-seq read alignment evidence (spliced/spanning read counts, coverage patterns) — that is InPACT's read module with its random-forest classifier, not the CNN sequence module
- The task is quantifying IPA isoform usage or transcript abundance — use the Salmon-based quantification step instead
- The data are 3'-end or long-read sequencing benchmarks for comparing IPA-identification tools like IPAFinder or APAIQ, rather than training a sequence-based classifier

## Inputs

- RefSeq gene annotation (GTF, e.g. hg38.109.20211119.ncbiRefSeq.gtf) with annotated polyA sites as positive windows
- Human reference genome (GRCh38/hg38) FASTA for extracting 201-nt windows
- PolyA site lists from GENCODE, PolyA_DB 3, and PolyASite 2.0 for constructing held-out test sets

## Outputs

- Trained CNN sequence-module classifier that scores 201-nt windows as polyA sites
- Predicted polyA site annotation (e.g. InPACT_polyAsites.hg38.saf: gene, chromosome, start, end, strand)
- sequence_module_auroc_results.csv with AUROC values per test database (0.954 GENCODE, 0.920 PolyA_DB 3, 0.794 PolyASite 2.0)

## How to apply

Extract 201-nt genomic windows centered on RefSeq-annotated polyA sites as positives and an equal number of 201-nt windows randomly sampled from intergenic regions as negatives, one-hot encoding nucleotides as A=[1,0,0,0], T=[0,1,0,0], G=[0,0,1,0], C=[0,0,0,1]. Build a CNN with two convolution-ReLU-max-pooling pairs and one fully connected layer with dropout, fit it on the training set, and tune hyper-parameters on a validation set by random sampling. Construct separate held-out test sets from GENCODE, PolyA_DB 3, and PolyASite 2.0 polyA sites, excluding any sites overlapping the training data. Evaluate the trained CNN on each test set by AUROC and benchmark against DeepPASS, APARENT, and DeepPASTA on the same constructed test sets. The resulting site-level scores feed InPACT's downstream read module, which annotates potential IPA sites from 20 nt upstream to each predicted polyA site.

## Related tools

- **InPACT** (Hosts the CNN sequence module; its predicted polyA-site annotation (InPACT_polyAsites.hg38.saf) is the downstream consumer of the trained model) — https://github.com/YY-TMU/InPACT
- **DeepPASS** (Benchmark polyA-site predictor evaluated on the same constructed held-out test sets)
- **APARENT** (Benchmark polyA-site predictor evaluated on the same constructed held-out test sets)
- **DeepPASTA** (Benchmark polyA-site predictor evaluated on the same constructed held-out test sets)

## Evaluation signals

- AUROC of 0.954 on the GENCODE held-out test set, 0.920 on PolyA_DB 3, and 0.794 on PolyASite 2.0; large deviations indicate training or test-construction errors
- Test-set sites overlap none of the training data (explicit exclusion step before evaluation)
- Positive and negative windows are balanced 1:1 (equal number of randomly sampled intergenic 201-nt windows)
- Input tensors are 201-nt one-hot encoded with the exact channel mapping A=[1,0,0,0], T=[0,1,0,0], G=[0,0,1,0], C=[0,0,0,1]
- Benchmark methods (DeepPASS, APARENT, DeepPASTA) are scored on identical constructed test sets for a fair comparison

## Limitations

- Generalization drops on experimentally supported sites: AUROC is lowest on PolyASite 2.0 (0.794) versus annotation-derived sets (0.954 GENCODE, 0.920 PolyA_DB 3)
- No version numbers reported for InPACT or the benchmarked tools, complicating exact reproduction
- Hyper-parameter tuning relies on random sampling on a validation set, so results may vary with the sampled configuration

## Evidence

- [methods] The sequence module incorporates a convolutional neural network (CNN) with multiple convolutional and pooling layers: "The sequence module incorporates a convolutional neural network (CNN) with multiple convolutional and pooling layers"
- [results] achieving AUROC values of 0.954 for GENCODE, 0.920 for PolyA_DB 3, and 0.794 for PolyASite 2.0: "achieving AUROC values of 0.954 for GENCODE, 0.920 for PolyA_DB 3, and 0.794 for PolyASite 2.0"
- [intro] we present a new computational method, InPACT (Intronic PolyAdenylation Characterization Tool), which incorporates a sequence module and a read module to enable precise sample-wise characterization: "we present a new computational method, InPACT (Intronic PolyAdenylation Characterization Tool), which incorporates a sequence module and a read module to enable precise sample-wise characterization"
