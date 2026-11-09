---
name: pseudodinucleotide-physicochemical-encoding
description: Use when converting 41 bp RNA m5C sequence segments into numerical features
  via dinucleotide physicochemical properties, e.g. to reproduce Staem5's PCPseDNC
  feature block before fusing with PSP, k-mer, and PseEIIP encodings.
when_to_use_negative:
- Input is already a numeric feature matrix (encoding has already been performed)
- The task requires feature blocks other than dinucleotide physicochemical composition,
  e.g. only PSP or only PseEIIP features are needed
- Sequences are not RNA m5C segment data of the expected 41 bp window length
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0099
tools:
- name: iLearn
  role: Used to compute PCPseDNC (parallel correlation pseudo dinucleotide composition)
    features encoding 38 dinucleotide physicochemical properties from the RNA sequence
    segments
- name: Staem5
  role: Downstream stacked ensemble model that consumes the fused feature matrix containing
    the PCPseDNC block as one of its four input feature types
  repo: https://github.com/Cxd-626/Staem5.git
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.omtn.2021.10.012
    title: 'Staem5: A novel computational approach for accurate prediction of m5C
      site'
  source_content_hash: sha256:f7f6775c2a718ad090cad3e7ba93e46eb8d9e482ff74aa6dc33f44b71de6c6b4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Cxd-626/Staem5.git
  tools:
  - Staem5
  - iLearn
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# pseudodinucleotide-physicochemical-encoding

## Summary

Compute parallel correlation pseudo dinucleotide composition (PCPseDNC) features from RNA sequence segments, encoding 38 dinucleotide physicochemical properties. In Staem5 this is one of four feature blocks (PSP, k-mer, PCPseDNC, PseEIIP) fused into the model input matrix.

## When to use

Use when converting 41 bp RNA m5C sequence segments into numerical features via dinucleotide physicochemical properties, e.g. to reproduce Staem5's PCPseDNC feature block before fusing with PSP, k-mer, and PseEIIP encodings.

## When NOT to use

- Input is already a numeric feature matrix (encoding has already been performed)
- The task requires feature blocks other than dinucleotide physicochemical composition, e.g. only PSP or only PseEIIP features are needed
- Sequences are not RNA m5C segment data of the expected 41 bp window length

## Inputs

- Labeled 41 bp m5C positive/negative RNA sequence segments (FASTA or sequence lists) for A. thaliana and M. musculus
- iLearn PCPseDNC encoding module (38 dinucleotide physicochemical properties)

## Outputs

- PCPseDNC feature columns per sequence segment
- Fused feature matrix (fused_features.csv) combining PCPseDNC with PSP, k-mer, and PseEIIP blocks, one row per training sample (e.g., 5,298 positive + 5,298 negative for A. thaliana; 4,563 + 4,563 for M. musculus)

## How to apply

Load the labeled 41 bp m5C training and independent test sequence segments for A. thaliana (GSE94065 benchmark data) and M. musculus. Using iLearn, compute PCPseDNC over each sequence segment: the parallel correlation scheme encodes dinucleotide composition together with the 38 dinucleotide physicochemical properties into pseudo components. Concatenate the resulting PCPseDNC columns with the PSP, k-mer (k = 1, 2, 3), and PseEIIP blocks into one fused input matrix written to fused_features.csv. PCPseDNC is specifically listed in the paper's methods as one of the four encoding schemes integrated by the model, and its inclusion is a prerequisite for the downstream F-score feature selection and stacked ensemble training.

## Related tools

- **iLearn** (Used to compute PCPseDNC (parallel correlation pseudo dinucleotide composition) features encoding 38 dinucleotide physicochemical properties from the RNA sequence segments)
- **Staem5** (Downstream stacked ensemble model that consumes the fused feature matrix containing the PCPseDNC block as one of its four input feature types) — https://github.com/Cxd-626/Staem5.git

## Examples

```
python -m ilearn.pse.pcpse_dnc --seq segments.fa --lambda 3 --w 1.0 --out pcpsednc.csv; python fuse_features.py --blocks psp.csv kmer.csv pcpsednc.csv pseeiip.csv --out fused_features.csv
```

## Evaluation signals

- fused_features.csv exists and contains one row per training sample with the expected class balance (5,298 positive + 5,298 negative for A. thaliana; 4,563 + 4,563 for M. musculus)
- All four encoding blocks (PSP, k-mer, PCPseDNC, PseEIIP) are present as distinct column groups in the fused matrix
- PCPseDNC columns correspond to the 38 dinucleotide physicochemical properties encoded by iLearn
- Downstream 10-fold cross-validation accuracy of the stacked model is in the reported range (e.g. 73.89% accuracy and 0.479 MCC for A. thaliana), indicating the encoding was computed correctly

## Limitations

- PCPseDNC is one of four required feature blocks; using it alone does not reproduce Staem5 performance, since the model was developed based on four types of sequence features
- Species-specific models are required: the paper notes it could be difficult to develop a general model for cross-species prediction
- The Staem5 pipeline applies F-score feature selection afterward, so raw PCPseDNC dimensions are reduced rather than used directly

## Evidence

- [methods] PCPseDNC encoding via iLearn: [evidence span withheld — anchor status: not_found]
