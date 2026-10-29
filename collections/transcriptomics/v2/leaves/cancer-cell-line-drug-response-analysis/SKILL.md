---
name: cancer-cell-line-drug-response-analysis
description: Use when you have matched gene expression profiles for cancer cell lines
  plus drug sensitivity (IC50) data (e.g., the GDSC panel) and want to identify which
  signaling pathway activities explain drug response.
when_to_use_negative:
- You need pathway activity estimates for pathways outside the 11 PROGENy footprints
  (EGFR, Hypoxia, JAK-STAT, MAPK, NFkB, p53/DDR, PI3K, TGFb, Trail, VEGF/PDGF) — no
  perturbation-based signature exists for them
- Your drug-sensitivity values are raw dose-response curves rather than processed
  IC50 values requiring no further curve fitting
- You require tissue- or context-specific pathway signatures; the article notes consensus
  PROGENy signatures may be less reflective of activation than tissue-specific ones
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: progeny
  role: Scores the pathway-responsive gene signatures in each cell line expression
    profile to produce the pathway activity scores used in the ANOVA
  repo: https://github.com/saezlab/progeny
- name: decoupler
  role: Recommended companion for inferring pathway activities from the PROGENy signatures
    (R or python)
  repo: https://github.com/saezlab/decoupler-py
- name: footprints
  role: Analysis code for the paper, including the drug-sensitivity association analysis
    (analyses/drug_assocs) for Fig. 4
  repo: https://github.com/saezlab/footprints
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-017-02391-6
    title: Perturbation-response genes reveal signaling footprints in cancer gene
      expression
  source_content_hash: sha256:2fbba56f1d6e3a6730e8f2e512c6601ff33e3d3e7b864a0efe9fc10360da40a1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saezlab/progeny
  tools:
  - decoupler
  - footprints
  - progeny
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# cancer-cell-line-drug-response-analysis

## Summary

Reproduces drug-sensitivity associations by testing whether PROGENy-inferred pathway activity scores explain IC50 values across a cancer cell line panel (GDSC: 265 drugs, 805 cell lines), yielding a drug-pathway association table filtered at 10% FDR. It demonstrates that perturbation-derived pathway footprints provide markers for drug indications.

## When to use

Use when you have matched gene expression profiles for cancer cell lines plus drug sensitivity (IC50) data (e.g., the GDSC panel) and want to identify which signaling pathway activities explain drug response.

## When NOT to use

- You need pathway activity estimates for pathways outside the 11 PROGENy footprints (EGFR, Hypoxia, JAK-STAT, MAPK, NFkB, p53/DDR, PI3K, TGFb, Trail, VEGF/PDGF) — no perturbation-based signature exists for them
- Your drug-sensitivity values are raw dose-response curves rather than processed IC50 values requiring no further curve fitting
- You require tissue- or context-specific pathway signatures; the article notes consensus PROGENy signatures may be less reflective of activation than tissue-specific ones

## Inputs

- GDSC v17a microarray gene expression matrix for cell lines (ArrayExpress E-MTAB-3610)
- GDSC drug-sensitivity table of IC50 values for 265 drugs across 805 cell lines
- Cell line annotations: MSI status and tissue of origin (TCGA label)
- PROGENy pathway-responsive gene signatures (R/Bioconductor progeny package)

## Outputs

- drug_pathway_associations.csv: per drug-pathway association table with effect sizes and FDR values (178 significant rows at 10% FDR)
- Per-cell-line PROGENy pathway activity score matrix (11 pathway scores per cell line)

## How to apply

Download the GDSC v17a microarray expression dataset (ArrayExpress accession E-MTAB-3610, also available via cancerrxgene.org/gdsc1000) together with the GDSC IC50 table covering 265 drugs in 805 cell lines. Score the 11 PROGENy pathway-responsive gene signatures (EGFR, Hypoxia, JAK-STAT, MAPK, NFkB, p53/DDR, PI3K, TGFb, Trail, VEGF/PDGF), which use the 100 most significant responsive genes per pathway as model coefficients, in each cell line's expression profile with the progeny package. For each drug-pathway pair, fit an ANOVA relating IC50 to the pathway score while conditioning on MSI status and tissue of origin, then adjust p-values for multiple testing and retain associations significant at 10% FDR. Write the resulting table with effect sizes and FDR values to drug_pathway_associations.csv; the reference result contains 178 significant associations, with MAPK/EGFR activity dominating MAPK-targeting drugs and Nutlin-3a against p53 as the strongest hit.

## Related tools

- **progeny** (Scores the pathway-responsive gene signatures in each cell line expression profile to produce the pathway activity scores used in the ANOVA) — https://github.com/saezlab/progeny
- **decoupler** (Recommended companion for inferring pathway activities from the PROGENy signatures (R or python)) — https://github.com/saezlab/decoupler-py
- **footprints** (Analysis code for the paper, including the drug-sensitivity association analysis (analyses/drug_assocs) for Fig. 4) — https://github.com/saezlab/footprints

## Examples

```
library(progeny); scores <- progeny(expr_matrix, scale = TRUE, top = 100)
```

## Evaluation signals

- The output drug_pathway_associations.csv contains exactly 178 significant association rows
- All reported FDR values in the output table fall within [0, 0.1]
- The table contains 'Nutlin-3a' as the strongest drug-pathway hit (Nutlin-3a vs p53-responsive genes)
- MAPK/EGFR pathway scores appear among the top associations with MAPK-targeting drugs
- Each ANOVA model conditions on both MSI status and tissue of origin (no marginal-only models)

## Limitations

- Pathway activity can only be inferred for the 11 pathways with perturbation-derived consensus signatures; expression level of a pathway gene set should not be interpreted as its activity
- While PROGENy estimates pathway activity well in large heterogeneous datasets, tissue-specific signatures may more closely reflect activation status in a given context than the consensus signatures
- More studies are required to fully elucidate the differences between the common core response and additional transcriptional modules that may not always be activated

## Evidence

- [readme] Explaining drug sensitivity (Fig. 4) analysis location: "#### Explaining drug sensitivity (Fig. 4)

Analyses available in [analyses/drug_assocs](analyses/drug_assocs)."
- [readme] GDSC data source and per-cell-line pathway scoring: "We took the GDSC data from
[cancerrxgene.org/gdsc1000](http://www.cancerrxgene.org/gdsc1000),
computing pathway scores for each cell line"
- [readme] Top-100 responsive gene coefficient selection: [evidence span withheld — anchor status: external_doc]
- [abstract] PROGENy provides markers for drug indications: "(ii) provide or improve strong markers for drug indications"
- [discussion] Tissue-specific signatures caveat: "signatures derived from, for instance, a specific tissue may still more closely reflect activation"
