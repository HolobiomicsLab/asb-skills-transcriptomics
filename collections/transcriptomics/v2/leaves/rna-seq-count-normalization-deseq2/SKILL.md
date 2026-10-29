---
name: rna-seq-count-normalization-deseq2
description: Use when starting from raw RNA-seq gene count data across contrasting
  conditions (e.g. perturbation vs. vehicle control) that must be put on a comparable
  scale before downstream signature or pathway-activity scoring.
when_to_use_negative:
- Input is already normalized (e.g. TPM/FPKM/log-CPM) — reapplying DESeq2 size factors
  is invalid since they assume raw counts
- Single-cell UMI data where DESeq2 size-factor assumptions break down; the progeny
  README points to decoupleR-style scoring on scRNA-seq instead
- Proteomics or phosphoprotein measurements, which the article notes directly capture
  post-translational signaling rather than being inferred from transcript counts
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: DESeq2
  role: Median-of-ratios size-factor normalization and variance-stabilizing transformation
    of raw RNA-seq counts prior to pathway scoring
- name: progeny
  role: Bioconductor package supplying the PROGENy pathway weight matrix applied to
    the normalized/vst expression matrix
  repo: github.com/saezlab/progeny
- name: decoupler
  role: Statistical framework for inferring pathway activities from the PROGENy signatures
    (R and Python)
  repo: github.com/scverse/decoupler
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-017-02391-6
    title: Perturbation-response genes reveal signaling footprints in cancer gene
      expression
  source_content_hash: sha256:2aba2149320a49c49330289929475e497bcc63063f4517748d7b0c78362ba40d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DESeq2
  - decoupler
  - progeny
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- small-rna-seq-count-normalization
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# rna-seq-count-normalization-deseq2

## Summary

Normalizes raw RNA-seq count matrices with DESeq2 size factors (plus variance-stabilizing transformation) so that samples can be scored with pathway signatures such as PROGENy in perturbation experiments (e.g. GSE97979 HEK293-ER).

## When to use

Use when starting from raw RNA-seq gene count data across contrasting conditions (e.g. perturbation vs. vehicle control) that must be put on a comparable scale before downstream signature or pathway-activity scoring.

## When NOT to use

- Input is already normalized (e.g. TPM/FPKM/log-CPM) — reapplying DESeq2 size factors is invalid since they assume raw counts
- Single-cell UMI data where DESeq2 size-factor assumptions break down; the progeny README points to decoupleR-style scoring on scRNA-seq instead
- Proteomics or phosphoprotein measurements, which the article notes directly capture post-translational signaling rather than being inferred from transcript counts

## Inputs

- Raw RNA-seq gene count matrix (e.g. GEO accession GSE97979 HEK293-ER perturbation counts for 4OHT, Ly294002, TNF-alpha, TGF-beta, IFN-gamma and BSA control)
- Sample metadata table mapping samples to condition groups
- Gene annotation for identifier mapping if the count matrix lacks matching gene symbols

## Outputs

- DESeq2 size-factor-normalized count matrix
- Variance-stabilized (vst) expression matrix suitable for pathway scoring
- Per-sample pathway activity scores, e.g. pathway_score_deltas.csv with fields pathway, condition, delta_score

## How to apply

Build a DESeqDataSet from the raw count matrix and sample metadata, then apply DESeq2's median-of-ratios size-factor normalization to remove library-size/composition bias between samples. For signature-based scoring (e.g. PROGENy pathway activity as a weighted linear combination of gene expression), use variance-stabilized expression values rather than raw or log-CPM values. In the PROGENy HEK293-ER use case, normalized/vst expression is multiplied by the progeny package weight matrix to compute per-sample pathway scores, which are then compared against the BSA vehicle control to obtain delta scores per pathway (4OHT/MAPK, Ly294002/PI3K, TNF-alpha/NFkB, TGFb, IFN-gamma/JAK-STAT). The rationale, per the article, is that only consistently normalized expression lets pathway scores reflect true signaling activity rather than technical library variation.

## Related tools

- **DESeq2** (Median-of-ratios size-factor normalization and variance-stabilizing transformation of raw RNA-seq counts prior to pathway scoring)
- **progeny** (Bioconductor package supplying the PROGENy pathway weight matrix applied to the normalized/vst expression matrix) — github.com/saezlab/progeny
- **decoupler** (Statistical framework for inferring pathway activities from the PROGENy signatures (R and Python)) — github.com/scverse/decoupler

## Examples

```
dds <- DESeqDataSetFromMatrix(countData = counts, colData = coldata, design = ~ condition); dds <- estimateSizeFactors(dds); vst_expr <- assay(vst(dds))
```

## Evaluation signals

- Output file pathway_score_deltas.csv exists and contains the required fields pathway, condition, delta_score
- Sign of each delta_score matches the direction of change reported for the stimulated pathways in Fig. 2d of the article
- PROGENy scores agree with measured phosphorylation changes of key pathway proteins in the HEK293-ER perturbation data
- Each perturbation delta is computed relative to its corresponding BSA vehicle control score, not to another perturbation
- Scores are computed on variance-stabilized (not raw) expression, consistent with weighted linear combination of normalized expression

## Limitations

- Pathway scores from gene expression cannot account for post-translational modifications, so they may not always reflect true signaling activity
- Tissue- or context-specific signatures may more closely reflect pathway activation status than the consensus PROGENy signatures in a given context
- Expression level of a pathway's members should not be interpreted as pathway activity; perturbation-derived signatures like PROGENy are needed for activity inference

## Evidence

- [other] DESeq2 size-factor normalization with vst for scoring: [evidence span withheld — anchor status: not_found]
- [other] Scores must match phosphorylation and Fig 2d: [evidence span withheld — anchor status: not_found]
- [intro] Scores do not account for PTMs: "do not take into account the effect of post-translational modifications (Fig. 1a). It is therefore unclear if and under what circumstances the pathway scores obtained by these methods reflect"
- [abstract] PROGENy accurately infers pathway activity: "these results show that PROGENy accurately infers pathway activity from gene expression in a wide range of conditions"
- [readme] Normalization scripts in the paper repo: [evidence span withheld — anchor status: external_doc]
