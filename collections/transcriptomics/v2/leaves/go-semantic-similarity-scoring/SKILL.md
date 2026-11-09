---
name: go-semantic-similarity-scoring
description: Use when scoring candidate genes for functional relatedness to a specific
  GO term (e.g., spliceosome-mediated circRNA biogenesis) so the semantic similarity
  becomes a per-gene feature for downstream clustering or ranking.
when_to_use_negative:
- The candidate genes' biological role is unrelated to splicing — the single reference
  term GO:0000398 would give uninformative Zg values
- A feature matrix already containing per-gene functional annotation scores is available
  — recompute only if the gene set or reference GO term changed
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_3512
tools:
- name: GoSemSim
  role: R package computing the GO semantic similarity scores between each gene's
    annotations and the selected GO term, then Z-scored into Zg
  repo: https://github.com/BioinformaticsSTU/CRIT
- name: CRIT
  role: Pipeline whose features-collecting stage consumes the Zg feature; provides
    GET_Gosim_zscore.R implementing this step
  repo: https://github.com/BioinformaticsSTU/CRIT
- name: biomaRt
  role: Gene identifier retrieval/mapping needed to attach GO annotations to the candidate
    RBP list
  repo: https://github.com/BioinformaticsSTU/CRIT
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.omtn.2022.10.015
    title: 'CRIT: Identifying RNA-binding protein regulator in circRNA life cycle
      via non-negative matrix factorization'
  source_content_hash: sha256:1a836cb4a4532cc7c7cf3dacfa48bbc905975ea654ea98be6147f53ed80f717e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/BioinformaticsSTU/CRIT
  repo_url: https://github.com/BioinformaticsSTU/CRIT
  tools:
  - CRIT
  - GoSemSim
  - R
  - biomaRt
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# go-semantic-similarity-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes a Zg feature for each candidate gene: the Z-score of GO semantic similarity between the gene's annotations and the spliceosome term GO:0000398 (mRNA splicing, via spliceosome), using the R package GoSemSim, as one of the five features CRIT feeds into its NMF clustering of RBP regulators.

## When to use

Use when scoring candidate genes for functional relatedness to a specific GO term (e.g., spliceosome-mediated circRNA biogenesis) so the semantic similarity becomes a per-gene feature for downstream clustering or ranking.

## When NOT to use

- The candidate genes' biological role is unrelated to splicing — the single reference term GO:0000398 would give uninformative Zg values
- A feature matrix already containing per-gene functional annotation scores is available — recompute only if the gene set or reference GO term changed

## Inputs

- List of candidate genes (RBP_list.RData with 1,344 RBPs annotated by Ensembl gene ID, HGNC symbol, Entrez ID, UniProt SwissProt accession)
- GO term GO:0000398 (mRNA splicing, via spliceosome) as the reference term
- Human GO annotation database (org.Hs.eg.db) for semantic similarity computation

## Outputs

- GO_sim_Z_Score.txt: per-gene Zg feature (Z-score of GO semantic similarity to GO:0000398)
- Zg column merged into MATRIX.txt feature matrix alongside rho_m, Zp, Zs, and C value

## How to apply

For each candidate gene, retrieve its GO annotations (via org.Hs.eg.db / biomaRt gene-ID mapping across Ensembl, HGNC, Entrez, and UniProt identifiers) and compute the semantic similarity score between those annotations and the single selected term GO:0000398 'mRNA splicing, via spliceosome' using the R package GoSemSim. Then convert the raw similarity score of each gene into a Z-score across all candidate genes, yielding the Zg feature that reflects how closely a regulator's biological role relates to spliceosome function. Merge the resulting Zg column with the other CRIT features (rho_m, Zp, Zs, C value) into the feature matrix with GET_feauters_MATRIX.R prior to NMF clustering.

## Related tools

- **GoSemSim** (R package computing the GO semantic similarity scores between each gene's annotations and the selected GO term, then Z-scored into Zg) — https://github.com/BioinformaticsSTU/CRIT
- **CRIT** (Pipeline whose features-collecting stage consumes the Zg feature; provides GET_Gosim_zscore.R implementing this step) — https://github.com/BioinformaticsSTU/CRIT
- **biomaRt** (Gene identifier retrieval/mapping needed to attach GO annotations to the candidate RBP list) — https://github.com/BioinformaticsSTU/CRIT

## Examples

```
Rscript GET_Gosim_zscore.R  # computes GO semantic similarity of each candidate gene to GO:0000398 and Z-scores it into GO_sim_Z_Score.txt
```

## Evaluation signals

- GO_sim_Z_Score.txt exists and contains a Zg value for every one of the 1,344 candidate RBPs
- Zg values are Z-scored (approximately mean 0, unit variance) across the candidate set
- Genes annotated directly to GO:0000398 or core spliceosome components show top-ranked Zg values
- Merged MATRIX.txt contains a populated Zg column with no missing entries for the candidate RBPs

## Limitations

- Zg is defined against a single functional term (GO:0000398), so it captures only spliceosome-related functional proximity, not other circRNA-relevant roles
- Genes lacking GO annotations to splicing-related terms will score at the low end of the similarity distribution regardless of actual regulator potential
- Generalization of the CRIT feature strategy (including GO-term choice) to other diseases beyond cancer is untested per the article's discussion

## Evidence
