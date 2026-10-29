---
name: mitochondrial-gene-filtering
description: Use when ranking per-gene interpretability/attribution scores (e.g.,
  saliency maps) or building a gene signature, where mitochondrial transcripts could
  otherwise dominate or confound the top-ranked feature list.
when_to_use_negative:
- When the analysis goal is itself mitochondrial expression or metabolic-state assessment
  (filtering would remove the signal of interest)
- When mitochondrial genes have already been excluded upstream during normalization/QC
  of the spot expression matrices
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Cancer-Finder
  role: Produces the saliency-map per-gene importance scores that this filter is applied
    to before ranking
  repo: https://github.com/Patchouli-M/SequencingCancerFinder
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-46413-6
    title: Domain generalization enables general cancer cell annotation in single-cell
      and spatial transcriptomics
  source_content_hash: sha256:28cc618c101771b9cb0b84b56d294bff6022ea64b19f66f2a36d009267df208e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Patchouli-M/SequencingCancerFinder
  tools:
  - Cancer-Finder
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# Mitochondrial gene filtering

## Summary

Removes mitochondrial genes from feature-importance (saliency-map) results before ranking genes, as applied in the Cancer-Finder ccRCC 10-gene signature workflow so that mitochondrial transcripts do not contaminate the tumor-signature gene list.

## When to use

Use when ranking per-gene interpretability/attribution scores (e.g., saliency maps) or building a gene signature, where mitochondrial transcripts could otherwise dominate or confound the top-ranked feature list.

## When NOT to use

- When the analysis goal is itself mitochondrial expression or metabolic-state assessment (filtering would remove the signal of interest)
- When mitochondrial genes have already been excluded upstream during normalization/QC of the spot expression matrices

## Inputs

- Per-gene aggregated saliency-map importance scores (e.g., from top10_saliency_genes.csv workflow) across the 5 ccRCC Visium slides
- Gene symbol/identifier list from the spot expression matrices (log-transformed global-scaling normalized, 5000 ST input genes)

## Outputs

- Filtered importance-score table with mitochondrial genes removed
- Ranked gene list (top10_saliency_genes.csv) containing only non-mitochondrial genes

## How to apply

After running the Cancer-Finder modified saliency-map interpretability module (gradient-based feature attribution through the trained feature-extraction and classification layers) over all spots of the 5 ccRCC Visium slides and aggregating per-gene importance scores, filter mitochondrial genes out of the interpretability results BEFORE ranking. Identify mitochondrial genes by their gene-symbol prefix (conventionally MT-/mt- in human Ensemble/Symbol identifiers). Only then rank the remaining genes by aggregated importance; for the ccRCC ST analysis this yields the top-ten signature NDRG1, TAGLN, MALAT1, IGKC, IGHA1, IGHG4, IGLC2, IGHG3, SOD2, KRT19. The rationale is that mitochondrial expression reflects QC/metabolic state rather than malignant-cell identity, so it must not enter the signature ranking.

## Related tools

- **Cancer-Finder** (Produces the saliency-map per-gene importance scores that this filter is applied to before ranking) — https://github.com/Patchouli-M/SequencingCancerFinder

## Examples

```
import pandas as pd; imp = pd.read_csv('saliency_importance.csv', index_col=0); imp_filt = imp[~imp.index.str.upper().str.startswith(('MT-', 'MT-'))].sort_values('importance', ascending=False); imp_filt.head(10).to_csv('top10_saliency_genes.csv')
```

## Evaluation signals

- No gene in the ranked output begins with the mitochondrial prefix (MT-/mt-)
- The filtered top-ten list reproduces the published ccRCC signature exactly: NDRG1, TAGLN, MALAT1, IGKC, IGHA1, IGHG4, IGLC2, IGHG3, SOD2, KRT19
- Filtering occurs before ranking; re-ranking after filtering must not change relative order of non-mitochondrial genes
- Output file top10_saliency_genes.csv contains exactly ten rows

## Limitations

- Filtering is only meaningful in the ranking stage; applying it to the raw expression matrix would not substitute for the article's interpretability-specific filter placement
- Cancer-Finder is not recommended for hematologic tumor data, limiting the scope of signatures this filter supports

## Evidence

- [methods] workflow step 5: [evidence span withheld — anchor status: not_found]
- [abstract] signature definition: "identifies a gene signature consisting of 10 genes that are significantly co-localized and enriched at the tumor-normal interface"
- [readme] interpretability module: "Cancer-Finder integrated an interpretability module, which can utilizes a saliency map to identified important genes related to prognosis"
