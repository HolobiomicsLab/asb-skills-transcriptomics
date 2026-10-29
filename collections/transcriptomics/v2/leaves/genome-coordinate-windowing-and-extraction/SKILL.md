---
name: genome-coordinate-windowing-and-extraction
description: Use when a genome annotation (GTF) and reference genome (e.g., hg19 FASTA)
  must be turned into fixed-length, strand-corrected sequence fragments centered on
  candidate residues for downstream model prediction.
when_to_use_negative:
- When fragments must be centered on residues other than uridine — the center-uridine
  filter would discard the candidates of interest
- When the prediction target is experimentally validated Ψ sites themselves (training
  data), since this workflow masks known RMBase sites ±25 bp to scan for novel sites
edam_operation: http://edamontology.org/operation_0369
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3512
tools:
- name: BEDTools
  role: Extracts coding sequences from hg19 + GENCODE gtf with strand-aware DNA-to-RNA
    conversion and masks RMBase Ψ sites with 25 bp flanks on both sides
- name: SeqKit
  role: Generates running windows of 51 bp fragments across the masked transcriptome
- name: hg19
  role: Human reference genome used as the sequence source for coordinate-based extraction
- name: RMBase v2.0
  role: Provides known Ψ site coordinates that are masked with 25 bp flanking sequences
    before novel-site scanning
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2020.07.010
    title: 'MU-PseUDeep: A deep learning method for prediction of pseudouridine sites'
  source_content_hash: sha256:94e336f95e3bcf066956272fc787ab0cc391bfa1d74c5d03aaa90705d749ad71
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BEDTools
  - MU-PseUDeep
  - RMBase v2.0
  - SeqKit
  - hg19
  edam_operation: http://edamontology.org/operation_0369
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# Genome coordinate windowing and extraction

## Summary

Converts genome-wide coordinate annotations into strand-aware, uridine-centered sequence fragments for ML scoring, using BEDTools to extract transcriptome coding sequences from hg19 + GENCODE and SeqKit to generate 51 bp running windows.

## When to use

Use when a genome annotation (GTF) and reference genome (e.g., hg19 FASTA) must be turned into fixed-length, strand-corrected sequence fragments centered on candidate residues for downstream model prediction.

## When NOT to use

- When fragments must be centered on residues other than uridine — the center-uridine filter would discard the candidates of interest
- When the prediction target is experimentally validated Ψ sites themselves (training data), since this workflow masks known RMBase sites ±25 bp to scan for novel sites

## Inputs

- Reference genome FASTA (hg19)
- Gene annotation GTF (GENCODE)
- Known modification site coordinates (RMBase v2.0 Ψ sites, for masking)
- Window length parameter (51 bp) and center-residue constraint (uridine)

## Outputs

- Strand-corrected RNA coding sequences per gene (FASTA)
- Masked transcriptome with known Ψ sites plus 25 bp flanks removed
- Set of 51 bp uridine-centered fragments ready for model scoring
- Genomic coordinates of candidate fragments (recorded in Supplemental_table2.csv)

## How to apply

Extract coding sequences for each transcriptome gene with BEDTools from the hg19 genome and the GENCODE gtf file, converting DNA to RNA according to each gene's strand so motifs are read in transcriptional orientation. Mask known RMBase pseudouridine sites together with 25 bases of flanking sequence on both sides using BEDTools, so that a subsequent scan targets only novel candidate sites. Generate running windows of 51 base pair fragments across the transcriptome with SeqKit, then retain only fragments carrying a uridine at their center — this matches the 51 × 4 one-of-K input encoding expected by MU-PseUDeep. The strand-aware RNA conversion and center-uridine filter are the key correctness decisions: they determine which positions the predictor is ever allowed to score.

## Related tools

- **BEDTools** (Extracts coding sequences from hg19 + GENCODE gtf with strand-aware DNA-to-RNA conversion and masks RMBase Ψ sites with 25 bp flanks on both sides)
- **SeqKit** (Generates running windows of 51 bp fragments across the masked transcriptome)
- **hg19** (Human reference genome used as the sequence source for coordinate-based extraction)
- **RMBase v2.0** (Provides known Ψ site coordinates that are masked with 25 bp flanking sequences before novel-site scanning)

## Examples

```
bedtools getfasta -fi hg19.fa -bed gencode_cds.bed -s -name | seqkit subseq -r 1:51 --bed uridine_windows.bed
```

## Evaluation signals

- Every output fragment is exactly 51 bp long with a uridine at position 26 (the center)
- Sequences read as RNA (U present, T absent) and match the orientation implied by each gene's strand in the GENCODE gtf
- No extracted fragment overlaps a known RMBase Ψ site or its 25 bp flanks
- Genomic coordinates of fragments can be mapped back to hg19 and cross-checked against the gene set recorded in Supplemental_table2.csv

## Limitations

- Known RMBase sites plus 25 bp flanks are masked, so this workflow cannot re-detect or validate already documented Ψ sites — it targets novel sites only
- The 51 bp window matches MU-PseUDeep's expected input shape (51 × 4 one-of-K encoding); other window lengths would be incompatible with the trained model
- Only uridine-centered fragments are retained, so any non-uridine candidate positions are excluded by construction

## Evidence

- [methods] Transcriptome extraction from hg19 with strand conversion: "obtained using BedTools from the hg19 genome and gencode gtf file"
- [methods] Masking of known Ψ sites with flanks: "we extracted the Ψ and surrounding 25 bases upstream and downstream nucleotides using BEDTools"
- [methods] Uridine-centered fragment filter: "Only those fragments with uridine at their center were considered for further prediction"
- [results] Genomic regions recorded in supplement: "details about genomic region can be found in the supplemental Excel file, “Supplemental_table2.csv”"
- [other] 51 bp fragment windows and masking in the scan workflow: [evidence span withheld — anchor status: not_found]
