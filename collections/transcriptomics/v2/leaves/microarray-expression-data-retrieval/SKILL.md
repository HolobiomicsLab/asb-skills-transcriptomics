---
name: microarray-expression-data-retrieval
description: Use when an analysis requires baseline (non-perturbation) microarray
  expression profiles for GDSC cancer cell lines, e.g. to score pathway-responsive
  gene signatures against IC50 drug sensitivity, starting from ArrayExpress/cancerrxgene
  accessions.
when_to_use_negative:
- The goal is perturbation z-scores from perturbation experiments rather than baseline
  cell-line expression profiles — use the footprints z-score scripts on ArrayExpress
  perturbation submissions instead.
- The input is already a normalized, sample-by-gene expression matrix with cell-line
  IDs matched to drug response — retrieval can be skipped and pathway scoring can
  proceed directly.
- The data are proteomic or genomic (mutation/CNV) rather than microarray transcriptomics
  — the GDSC drug-association workflow in the article operates on expression profiles.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3272
tools:
- name: progeny
  role: Consumes the retrieved microarray expression profiles to score the 11 PROGENy
    pathway-responsive gene signatures per cell line prior to drug-association testing
  repo: github.com/saezlab/progeny
- name: footprints data scripts
  role: Normalize and QC each expression series as a whole (normalize_data.r) and
    assemble the relevant expression matrix (expr.r) after download
  repo: saezlab/footprints
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-017-02391-6
    title: Perturbation-response genes reveal signaling footprints in cancer gene
      expression
  source_content_hash: sha256:e4cb9b9ed36849019de77cfd27020b01b0670f33eacc213062d14627c4db27fb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - footprints data scripts
  - progeny
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# microarray-expression-data-retrieval

## Summary

Retrieves the GDSC v17a microarray gene-expression dataset (ArrayExpress accession E-MTAB-3610, 805 cancer cell lines) together with the matching GDSC IC50 drug-sensitivity table (265 drugs), providing the expression inputs on which PROGENy pathway scores are computed for drug-sensitivity association analyses.

## When to use

Use when an analysis requires baseline (non-perturbation) microarray expression profiles for GDSC cancer cell lines, e.g. to score pathway-responsive gene signatures against IC50 drug sensitivity, starting from ArrayExpress/cancerrxgene accessions.

## When NOT to use

- The goal is perturbation z-scores from perturbation experiments rather than baseline cell-line expression profiles — use the footprints z-score scripts on ArrayExpress perturbation submissions instead.
- The input is already a normalized, sample-by-gene expression matrix with cell-line IDs matched to drug response — retrieval can be skipped and pathway scoring can proceed directly.
- The data are proteomic or genomic (mutation/CNV) rather than microarray transcriptomics — the GDSC drug-association workflow in the article operates on expression profiles.

## Inputs

- ArrayExpress accession E-MTAB-3610 (GDSC v17a microarray expression dataset)
- GDSC IC50 drug-sensitivity table (265 drugs x 805 cell lines)
- Cell-line metadata (MSI status, tissue of origin)

## Outputs

- Normalized microarray expression matrix for 805 GDSC cell lines
- Cell-line matched expression-IC50 joined data frame ready for pathway scoring

## How to apply

Download the GDSC v17a microarray expression dataset from ArrayExpress under accession E-MTAB-3610, and in parallel the GDSC IC50 drug-sensitivity table covering 265 drugs across 805 cell lines. Each expression series must be normalized and QC'd as a whole before assembling the expression matrix, following the footprints repository's data scripts (normalize_data.r, expr.r). Match expression samples to cell-line identifiers used in the IC50 table so each cell line links one expression profile to 265 drug responses. The retrieved expression matrix is then handed to the progeny package to score the 11 pathway signatures (EGFR, Hypoxia, JAK-STAT, MAPK, NFkB, p53/DDR, PI3K, TGFb, Trail, VEGF/PDGF) per cell line. Keep track of cell-line annotations such as MSI status and tissue of origin, since these are used as covariates in the downstream ANOVA models.

## Related tools

- **progeny** (Consumes the retrieved microarray expression profiles to score the 11 PROGENy pathway-responsive gene signatures per cell line prior to drug-association testing) — github.com/saezlab/progeny
- **footprints data scripts** (Normalize and QC each expression series as a whole (normalize_data.r) and assemble the relevant expression matrix (expr.r) after download) — saezlab/footprints

## Examples

```
Rscript normalize_data.r
```

## Evaluation signals

- The retrieved dataset corresponds to ArrayExpress accession E-MTAB-3610 (GDSC v17a) and contains expression profiles for 805 cell lines.
- The matched IC50 table covers exactly 265 drugs and cell-line IDs align 1:1 with expression sample IDs.
- Each expression series has been normalized and QC'd as a whole before assembling the expression matrix (no raw/unnormalized CEL files passed downstream).
- Downstream sanity check: pathway scoring plus ANOVA on the retrieved data should reproduce 178 significant drug-pathway associations at 10% FDR with Nutlin-3a–p53 as the strongest hit.

## Limitations

- GDSC expression data must be paired with drug sensitivity from the same release (v17a); mismatched releases break cell-line/drug concordance.
- SPIA and Pathifier comparisons in the paper required per-tissue baselines (pathway scores computed against all other cell lines of the same TCGA label), so raw retrieval without tissue annotation is insufficient for some methods.
- PROGENy consensus signatures are generic; tissue-specific signatures may more closely reflect pathway activation in a single-tissue context.

## Evidence

- [intro] the GDSC4,5 and CCLE6 have focussed on preclinical biomarkers of drug sensitivity in cancer cell lines: "the GDSC4,5 and CCLE6 have focussed on preclinical biomarkers of drug sensitivity in cancer cell lines"
- [readme] We took the GDSC data from cancerrxgene.org/gdsc1000, computing pathway scores for each cell line: "We took the GDSC data from
[cancerrxgene.org/gdsc1000](http://www.cancerrxgene.org/gdsc1000)
([article](http://www.sciencedirect.com/science/article/pii/S0092867416307462)),
computing pathway scores"
- [readme] Scripts to download and transform gene expression data, and to generate z-scores for the perturbation experiments are in the data directory: "Scripts to download and transform gene expression data, and to generate
z-scores for the perturbation experiments are in the [data](data) directory."
- [readme] We normalized and QC'd each series as whole (normalize_data.r), then assembled the relevant expression data (expr.r): "We normalized and QC'd each series as whole (`normalize_data.r`), then
assembled the relevant expression data (`expr.r`)"
