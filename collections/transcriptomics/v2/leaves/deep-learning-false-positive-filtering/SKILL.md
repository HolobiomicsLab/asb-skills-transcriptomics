---
name: deep-learning-false-positive-filtering
description: Use when running Anchored-fusion on paired-end RNA-seq data and candidate
  chimeric reads must be classified as natural chromosome fusions versus artificial
  cDNA fusions from PCR/sequencing artifacts.
when_to_use_negative:
- The sequencing library is single-end RNA-seq or whole-genome/whole-exome data —
  Anchored-fusion and its HVLD filter operate only on paired-end RNA-seq data
- You need a quick unfiltered candidate list and cannot spare training time — use
  '--not_filter_false_positive' instead, accepting more false positives
- The fusion events of interest do not include the user-provided anchored target gene
  — the whole anchored-search workflow, including its filter, will not detect them
edam_operation: http://edamontology.org/operation_3675
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: Anchored-fusion
  role: Hosts the HVLD deep-learning module that classifies candidate fusion genes
    and filters those with probability < 0.1
  repo: https://github.com/ShenLab-Genomics/Anchored-Fusion
- name: BLAT
  role: Produces the candidate chimeric read alignments and precise fusion locations
    that feed into HVLD filtering
- name: BWA-MEM
  role: Upstream aligner whose anchored-gene/whole-genome mapping generates the paired-end
    chimeric reads evaluated by HVLD
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100733
    title: Anchored-fusion enables targeted fusion search in bulk and single-cell
      RNA sequencing data
  source_content_hash: sha256:425cdc6d91b7a5b9aee821f344fe388a2bacd5e4a360a90bb2e3b67d67ec1dc9
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ShenLab-Genomics/Anchored-Fusion
  repo_url: https://github.com/ShenLab-Genomics/Anchored-Fusion
  tools:
  - Anchored-fusion
  - Arriba
  - BLAT
  - BWA-MEM
  - FusionCatcher
  - FusionInspector
  - scFusion
  edam_operation: http://edamontology.org/operation_3675
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# deep-learning-false-positive-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train and apply the HVLD (hierarchical view learning and distillation) deep-learning module from Anchored-fusion to discard artificial chimeric-fragment false positives generated during sequencing while retaining true fusion gene calls, e.g. NSD2-IGH in multiple myeloma scRNA-seq.

## When to use

Use when running Anchored-fusion on paired-end RNA-seq data and candidate chimeric reads must be classified as natural chromosome fusions versus artificial cDNA fusions from PCR/sequencing artifacts.

## When NOT to use

- The sequencing library is single-end RNA-seq or whole-genome/whole-exome data — Anchored-fusion and its HVLD filter operate only on paired-end RNA-seq data
- You need a quick unfiltered candidate list and cannot spare training time — use '--not_filter_false_positive' instead, accepting more false positives
- The fusion events of interest do not include the user-provided anchored target gene — the whole anchored-search workflow, including its filter, will not detect them

## Inputs

- Candidate fusion gene calls with spanning and split read support from Anchored-fusion's BLAT-based breakpoint step
- Positive training samples: 3540 fusion genes from Pan-Cancer Analysis of Whole Genomes (PCAWG)
- Homologous gene file from the program's 'data' folder
- Pre-trained HVLD model file (optional, for --not_train_filter_model)
- Current paired-end RNA-seq dataset used to generate dataset-specific artificial negative chimeric reads

## Outputs

- HVLD probability score per candidate fusion gene
- Filtered fusion gene call set with HVLD probability >= 0.1

## How to apply

After BLAT-based breakpoint prediction on candidate chimeric reads, pass candidates to the HVLD decision model. By default, retrain a per-dataset filter using the positive training samples (PCAWG fusion genes) and homologous gene file shipped in the program's 'data' folder, removing negative training samples that contain the anchored gene X and filtering reads aligned to homologous genes during negative sample generation. Candidates with an HVLD probability below 0.1 are filtered out. If the input dataset is too small to retrain, use the '--not_train_filter_model' option with a pre-trained model specified via '--model_file' (defaulted to the 'data' folder), or skip filtering entirely with '--not_filter_false_positive' to save time. Training on the current dataset is recommended because artificial fusion gene distributions are dataset-specific, but deviations may occur with the pre-trained model.

## Related tools

- **Anchored-fusion** (Hosts the HVLD deep-learning module that classifies candidate fusion genes and filters those with probability < 0.1) — https://github.com/ShenLab-Genomics/Anchored-Fusion
- **BLAT** (Produces the candidate chimeric read alignments and precise fusion locations that feed into HVLD filtering)
- **BWA-MEM** (Upstream aligner whose anchored-gene/whole-genome mapping generates the paired-end chimeric reads evaluated by HVLD)

## Examples

```
python Anchored_Fusion.py --file_anchored_cds=test/target_gene.fasta --fastq1=test/test_sample_1.fastq.gz --fastq2=test/test_sample_1.fastq.gz --out_folder=output_dir --file_ref_seq=reference.genome.fa --file_ref_ann=genome.annotation.gtf --not_train_filter_model --model_file=data/model.pt
```

## Evaluation signals

- All retained fusion calls have HVLD probability >= 0.1; candidates below this cutoff are removed
- Negative training samples containing the anchored gene X are absent from the retrained model's training set
- In the MM benchmark, the filtered pipeline recovers 99 NSD2-IGH-positive cells from the three t(4;14) patients (RRMM1, RRMM2, SMM0), i.e. 92.5% of positive cells, with breakpoints at chr4:1,902,353 and chr4:1,905,943
- Running with '--not_filter_false_positive' should yield a strictly larger candidate set than the HVLD-filtered run, with the extra calls enriched for artificial chimeric fragments
- If '--not_train_filter_model' is used, a '--model_file' path is provided and defaults to the 'data' folder of the program

## Limitations

- For datasets too small to retrain, the default pre-trained model on GitHub must be used, and results may deviate since it was not trained on the user's input data
- Skipping the filter ('--not_filter_false_positive') saves substantial time but leaves artificial chimeric-fragment false positives in the output
- Strict hard filtering to remove false positives can further reduce sensitivity for detecting true fusions; HVLD trades this off via learned probabilities rather than hard read-count rules
- The loss-balancing hyperparameter alpha in the HVLD loss function is not specified in the article

## Evidence
