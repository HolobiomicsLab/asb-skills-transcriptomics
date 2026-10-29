---
name: multi-species-transcript-feature-engineering
description: Use when raw transcript sequences (FASTA, >=200 nt, CD-HIT filtered at
  80%) from one or more species must be converted into a numeric feature matrix for
  coding-potential classification, especially when sORF-containing lncRNAs must be
  distinguished from PCTs.
when_to_use_negative:
- Input is already a precomputed five-type feature matrix — the encoding step is complete
  and the next step is CatBoost classification
- Transcripts are shorter than 200 nt — the article explicitly eliminates ncRNAs shorter
  than 200 nt before feature extraction
- Sequences have not been redundancy-reduced — applying feature encoding on unfiltered
  sets will not reproduce the CD-HIT 80% benchmark conditions
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3474
tools:
- name: Orfipy
  role: Extracts ORFs from transcript sequences using Standard Code transl_table=1
    (start codons ATG, TTG, CTG) as input to the ORF-attention features
- name: LncCat
  role: The full pipeline that fuses the five feature types and trains the CatBoost
    prediction model
  repo: https://github.com/a525076133/LncCat
- name: CatBoost
  role: Gradient-boosting classifier trained on the concatenated five-type feature
    matrices
  repo: https://github.com/a525076133/LncCat
- name: RNAsamba
  role: Competing neural-network coding-potential method used as a benchmark, not
    part of the feature engineering itself
  repo: http://github.com/apcamargo/RNAsamba
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2023.02.012
    title: 'LncCat: An ORF attention model to identify LncRNA based on ensemble learning
      strategy and fused sequence information'
  source_content_hash: sha256:dcbab46041deea5bbdbfc8ff73ef8186c9d00ef76a76f312e212a6ca01da6637
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/a525076133/LncCat
  repo_url: https://github.com/a525076133/LncCat
  tools:
  - CatBoost
  - LncCat
  - Orfipy
  - RNAsamba
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# multi-species-transcript-feature-engineering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Encoding transcript FASTA sequences from multiple species (human, mouse, zebrafish, wheat, chicken) into five fused feature types — codon-related, GC-related, sequence-related, peptide-related, and ORF-attention features — to build CatBoost-based lncRNA/PCT classifiers like LncCat.

## When to use

Use when raw transcript sequences (FASTA, >=200 nt, CD-HIT filtered at 80%) from one or more species must be converted into a numeric feature matrix for coding-potential classification, especially when sORF-containing lncRNAs must be distinguished from PCTs.

## When NOT to use

- Input is already a precomputed five-type feature matrix — the encoding step is complete and the next step is CatBoost classification
- Transcripts are shorter than 200 nt — the article explicitly eliminates ncRNAs shorter than 200 nt before feature extraction
- Sequences have not been redundancy-reduced — applying feature encoding on unfiltered sets will not reproduce the CD-HIT 80% benchmark conditions

## Inputs

- Transcript FASTA files per species (human, mouse, zebrafish, wheat, chicken) with lncRNA and PCT labels
- Pre-trained BERT model for peptide sequence embedding
- Orfipy (Standard Code transl_table=1, ATG/TTG/CTG start codons)
- CD-HIT (80% similarity threshold)

## Outputs

- Per-transcript feature matrix combining codon-related, GC-related, sequence-related, peptide-related, and ORF-attention features
- Trained CatBoost classifier per species or on a merged multi-species training matrix
- Per-species evaluation metrics (ACC, SEN, SPE, F1, PRE, MCC, AUC), e.g. lnccat_species_metrics.csv
- UMAP visualization of lncRNA vs. PCT feature distributions

## How to apply

Collect transcripts (≥200 nt) from NCBI RefSeq, GENCODE, and Ensembl per species and remove redundancy with CD-HIT at an 80% similarity threshold before splitting 70/10/20 into train/validation/test. Compute codon-related features (stop-codon count/frequency, Fickett score), GC-related features (GC1/GC2/GC3 and frame scores), sequence-related features (length, k-mer, CTD, hexamer score), and peptide-related features (Mw, PI, Gravy, instability index). Extract ORFs with Orfipy using Standard Code transl_table=1 with start codons ATG/TTG/CTG and stop codons TAA/TAG/TGA, then encode the ORF peptide sequences as ORF-attention features using a pre-trained BERT model. Concatenate the five feature blocks per transcript and feed the matrix to a CatBoost gradient-boosting classifier, applying a fixed 0.5 prediction-score cutoff (score ≥ 0.5 = lncRNA, < 0.5 = PCT). The rationale is that ORF-attention features make lncRNAs and PCTs separable in UMAP visualization and lift MCC on sORF-rich test sets where k-mer-only methods fail.

## Related tools

- **Orfipy** (Extracts ORFs from transcript sequences using Standard Code transl_table=1 (start codons ATG, TTG, CTG) as input to the ORF-attention features)
- **LncCat** (The full pipeline that fuses the five feature types and trains the CatBoost prediction model) — https://github.com/a525076133/LncCat
- **CatBoost** (Gradient-boosting classifier trained on the concatenated five-type feature matrices) — https://github.com/a525076133/LncCat
- **RNAsamba** (Competing neural-network coding-potential method used as a benchmark, not part of the feature engineering itself) — http://github.com/apcamargo/RNAsamba

## Examples

```
orfipy transcripts.fa --table 1 --start ATG,TTG,CTG --stop TAA,TAG,TGA --pep orfs.pep -o orfs.bed
```

## Evaluation signals

- The output feature matrix contains all five feature blocks: codon-related, GC-related, sequence-related, peptide-related, and ORF-attention
- Downstream CatBoost MCC on the human test set reproduces 0.9503 (F1 0.9742); mouse 0.9219; zebrafish 0.8591; wheat 0.8672; chicken 0.9047
- Integrated-dataset MCC matches 0.9236 (F1 0.9537) and sORF-integrated MCC matches 0.8346 (F1 0.9245) with the 0.5 cutoff
- All input transcripts are ≥200 nt and CD-HIT-filtered at 80% similarity; dataset split is 70/10/20
- UMAP visualization shows clear separation between lncRNAs and PCTs in ORF-attention feature space

## Limitations

- Cross-species transfer is weak: human, mouse, and zebrafish models perform poorly on wheat (MCC 0.67, 0.63, 0.66), so same-species models are preferred
- CatBoost and BERT versions and hyperparameter tuning settings are not reported, limiting exact reproducibility
- sORF-translated peptide sequences are not yet collected in this study, limiting functional annotation of sORF peptides

## Evidence
