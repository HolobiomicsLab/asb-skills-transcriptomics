---
name: isoform-switch-interaction-loss-assessment
description: Use when a dMDT or isoform switching event (e.g. transcript ENST00000380405)
  is identified and you need to determine which high-confidence STRING interactions
  are lost because the switched isoform lacks domains mediating them.
when_to_use_negative:
- The switching event's interactions are not supported by any known Pfam domain-domain
  pairs in 3did — physical-interaction loss cannot be established by this domain-comparison
  logic.
- The analysis goal is per-sample isoform expression quantification rather than network
  disruption assessment (CanIsoNet separates these explorations).
- STRING edges below the combined score > 900 confidence threshold are the target,
  since low-confidence edges are excluded from this high-quality interaction set.
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_0621
tools:
- name: CanIsoNet
  role: Web resource queried for the disease-specific dMDT entry and its isoform switching
    event, providing the STRING network disruption view and PFAM domain IDs of disrupted
    interactions
  repo: https://github.com/kahramanlab/CanIsoNet_Web
- name: STRING
  role: Source of protein interaction partners and combined scores used to filter
    high-confidence (> 900) edges whose loss is assessed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbad050
    title: 'CanIsoNet: a database to study the functional impact of isoform switching
      events in diseases'
  source_content_hash: sha256:2faa08beed8e47b3342645101a1c1ea8a4d5afdc99fa9e84454573221800f7d6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kahramanlab/CanIsoNet_Web
  tools:
  - CanIsoNet
  - STRING
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# isoform-switch-interaction-loss-assessment

## Summary

Assesses which protein interactions are functionally disrupted by a disease-specific isoform switching event by comparing STRING interaction networks filtered to high-confidence edges (combined score > 900) against Pfam domain-domain interactions (3did) for the dominant versus switched isoform. Grounded in CanIsoNet (caniso.net), which merges isoform switching events with STRING network disruptions.

## When to use

Use when a dMDT or isoform switching event (e.g. transcript ENST00000380405) is identified and you need to determine which high-confidence STRING interactions are lost because the switched isoform lacks domains mediating them.

## When NOT to use

- The switching event's interactions are not supported by any known Pfam domain-domain pairs in 3did — physical-interaction loss cannot be established by this domain-comparison logic.
- The analysis goal is per-sample isoform expression quantification rather than network disruption assessment (CanIsoNet separates these explorations).
- STRING edges below the combined score > 900 confidence threshold are the target, since low-confidence edges are excluded from this high-quality interaction set.

## Inputs

- dMDT identifier / transcript ID of an isoform switching event (e.g. HSPH1-002, ENST00000380405) from CanIsoNet
- STRING interaction network edges for the transcript's protein with combined scores
- Pfam domain composition (domain IDs) of the dominant and switched isoforms
- 3did catalog of known Pfam domain-domain interactions

## Outputs

- Set of high-quality STRING interactions passing combined score > 900 (e.g. 44 edges)
- Table of lost domain-mediated physical interactions (e.g. 12 edges, including DNAJB2 and DNAJB6)
- Flagged subset of edges supported by known 3did Pfam domain-domain interactions

## How to apply

Query CanIsoNet (caniso.net) for the dMDT entry of interest (e.g. the Alzheimer's disease transcript HSPH1-002 / ENST00000380405) and retrieve its STRING interaction partners. Filter the interaction list to high-quality edges using the STRING combined score threshold > 900. Cross-reference the filtered edges against the 3did catalog of Pfam domain-domain interactions to flag edges mediated by known domain-domain pairs. Compare Pfam domain composition of the dominant versus switched isoform to determine which domain-mediated physical interactions the switch abolishes, then compile the lost interactions into a table. Validate by reproducing reported counts (e.g. 44 high-quality interactions and 12 lost physical interactions including DNAJB2 and DNAJB6 for HSPH1-002).

## Related tools

- **CanIsoNet** (Web resource queried for the disease-specific dMDT entry and its isoform switching event, providing the STRING network disruption view and PFAM domain IDs of disrupted interactions) — https://github.com/kahramanlab/CanIsoNet_Web
- **STRING** (Source of protein interaction partners and combined scores used to filter high-confidence (> 900) edges whose loss is assessed)

## Examples

```
Browse caniso.net disease=Alzheimer's Disease → dMDT HSPH1-002 (ENST00000380405) → network disruption view; filter STRING edges combined_score > 900; flag edges with 3did Pfam domain-domain support; tabulate lost physical interactions (expect 12 incl. DNAJB2, DNAJB6).
```

## Evaluation signals

- High-quality interaction count after combined score > 900 filtering matches the reported value (44 for HSPH1-002).
- Lost physical interaction table contains exactly 12 edges for HSPH1-002.
- Lost interaction table includes edges to DNAJB2 and DNAJB6.
- Every flagged lost interaction is mediated by a Pfam domain pair present in 3did and present in the dominant isoform but absent in the switched isoform.
- Transcript identifiers (e.g. ENST00000380405) match the CanIsoNet dMDT entry for the disease type (Alzheimer's disease).

## Limitations

- Assessment depends on the coverage of the 3did catalog; domain-mediated losses are only detectable for Pfam domain-domain pairs already known.
- High-quality filtering at combined score > 900 discards lower-confidence STRING edges, so losses among weaker interactions are not captured.
- The two CanIsoNet GitHub mirrors report slightly different isoform coverage counts (11,528 vs 11,811 isoforms annotated), so database version may affect retrieved interactions.

## Evidence

- [abstract] CanIsoNet is the first webserver that incorporates isoform expression data with STRING interaction networks and ClinVar annotations to predict the pathogenic impact of isoform switching events: "CanIsoNet is the first webserver that incorporates isoform expression data with STRING interaction networks and ClinVar annotations to predict the pathogenic impact of isoform switching events"
- [abstract] PFAM domain IDs of disrupted interactions: "PFAM domain IDs of disrupted interactions"
- [readme] CanIsoNet provides followings to the users: 1) Browsing various statistics on disease-specific isoform switching events in each diseases: "2) Exploring and visualizing network disruptions in the STRING interaction network."
- [intro] first webserver that merges isoform switching events ... showing them on STRING interaction network: "CanIsoNet is the first webserver that merges isoform switching events in different disease types with their functional impact level and showing them on STRING interaction network"
- [abstract] CanIsoNet is freely available at https://www.caniso.net: "CanIsoNet is freely available at https://www.caniso.net."
