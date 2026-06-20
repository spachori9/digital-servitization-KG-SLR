# A Knowledge-Graph-Based Systematic Literature Review of Digital Servitization

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.XXXXXXX.svg)](https://doi.org/10.5281/zenodo.XXXXXXX)
![Neo4j](https://img.shields.io/badge/Neo4j-2026.04.0-blue)
![GDS](https://img.shields.io/badge/GDS-2026.04.0-green)
![Status](https://img.shields.io/badge/Status-Active-brightgreen)

## Overview

This repository contains all Cypher extraction scripts, graph schema documentation, query outputs, and data exports supporting the paper:

> **Pachori, S.** (2025). *A Knowledge-Graph-Based Systematic Literature Review of Digital Servitization.* Executive Doctoral Programme in Management, Indian Institute of Management Indore.

The study constructs a large-scale Neo4j knowledge graph from a corpus of **1,349 academic papers** on servitization and digital servitization, sourced via a systematic Scopus search (February 2026, 2,652 initial records). Graph-theoretic and network analysis methods — including PageRank, Betweenness Centrality, and Louvain Community Detection — are applied to surface structural patterns, research clusters, and citation influence across the literature.

A key finding of this study is that **fewer than 0.43% of papers explicitly link capability development to barrier identification**, revealing that these two streams have developed in parallel but remain structurally disconnected in the literature.

---

## Repository Structure

```
digital-servitization-KG-SLR/
│
├── cypher-queries/
│   ├── extraction/          ← Per-paper Cypher scripts (Papers 1–303)
│   ├── analysis/            ← Analytical queries (PageRank, Louvain, etc.)
│   └── maintenance/         ← Ghost node fixes, null handling, cleanup
│
├── graph-exports/
│   ├── pagerank_results.csv
│   ├── betweenness_results.csv
│   ├── louvain_communities.csv
│   └── Most_influential_papers_by_PageRank_updated.csv
│
├── data/
│   ├── methods_used.csv
│   ├── corpus_metadata.csv
│   └── prisma_flow.csv
│
├── docs/
│   ├── schema.md            ← Node types, relationship types, properties
│   └── loading_pipeline.md  ← How to reproduce the graph from scratch
│
└── README.md
```

---

## Knowledge Graph Statistics

| Metric | Value |
|--------|-------|
| Total Paper nodes | 2,162 |
| Total nodes | 13,751 |
| Node types | 28 |
| Relationship types | 154 |
| Total relationships | 24,368 |
| Corpus (final) | 1,349 studies |
| Extraction period | October 2025 – February 2026 |

---

## Node Types

The graph contains 28 node types including:

`Paper` · `Author` · `Institution` · `Concept` · `Theory` · `Method` · `Technique` · `Construct` · `Hypothesis` · `Finding` · `ResearchGap` · `Limitation` · `FutureWork` · `Barrier` · `Capability` · `Industry` · `Sector` · `DataSource` · `Journal` · `Country` · `Framework` · `Model` · `Variable` · `Outcome` · `Driver` · `Enabler` · `DSBarrier` · `TechnicalChallenge`

---

## Technical Environment

| Component | Version |
|-----------|---------|
| Neo4j | 2026.04.0 |
| Neo4j Desktop | 2.1.3 |
| Neo4j Graph Data Science (GDS) | 2026.04.0 |
| Query Language | Cypher |
| Loading Method | `cypher-shell` (chunked pipeline) |

---

## How to Reproduce the Graph

### Prerequisites
- Neo4j Desktop 2.1.3 or later
- Neo4j GDS plugin 2026.04.0
- Python 3.9+ (for loading pipeline)

### Steps

**1. Clone this repository**
```bash
git clone https://github.com/shivangipachori/digital-servitization-KG-SLR.git
cd digital-servitization-KG-SLR
```

**2. Start Neo4j and create a new database**

Open Neo4j Desktop → New Project → Add Database → Start

**3. Load the graph**

All extraction scripts are in `/cypher-queries/extraction/`. Run them sequentially using `cypher-shell`:

```bash
cat cypher-queries/extraction/all_papers.cypher | cypher-shell -u neo4j -p <password>
```

Or load individual paper scripts:
```bash
cypher-shell -u neo4j -p <password> -f cypher-queries/extraction/paper_001.cypher
```

**4. Run GDS projections**

```cypher
CALL gds.graph.project(
  'citationGraph',
  'Paper',
  {CITES: {orientation: 'NATURAL'}}
);
```

**5. Run analytical queries**

All analytical queries are in `/cypher-queries/analysis/`. See `docs/loading_pipeline.md` for full details.

---

## Key Analytical Queries

### Most Influential Papers (PageRank)
```cypher
CALL gds.pageRank.stream('citationGraph')
YIELD nodeId, score
RETURN gds.util.asNode(nodeId).title AS title,
       gds.util.asNode(nodeId).year AS year,
       gds.util.asNode(nodeId).doi AS doi,
       score
ORDER BY score DESC
LIMIT 25;
```

### Research Clusters (Louvain)
```cypher
CALL gds.louvain.stream('citationGraph')
YIELD nodeId, communityId
RETURN communityId,
       collect(gds.util.asNode(nodeId).title)[0..5] AS sample_papers,
       count(*) AS size
ORDER BY size DESC;
```

### Capability–Barrier Gap (Key Finding)
```cypher
MATCH (cap:Capability)<-[:PROPOSES_CAPABILITY]-(p:Paper)
      -[:IDENTIFIES_BARRIER|IDENTIFIES_CHALLENGE]->(bar)
WHERE bar:Barrier OR bar:DSBarrier OR bar:TechnicalChallenge
RETURN bar.name AS barrier,
       cap.name AS capability,
       count(DISTINCT p) AS papers
ORDER BY papers DESC;
```

---

## Corpus Construction (PRISMA)

| Stage | Count |
|-------|-------|
| Scopus search results (Feb 2026) | 2,652 |
| Mapped to ABDC journal list | 1,749 |
| After sector exclusions | 1,360 |
| After removing unretrieved papers | 1,349 |
| **Final corpus** | **1,349** |

**Scopus Boolean Search String:**
```
TITLE-ABS-KEY ("digital servitization" OR "servitization" OR "service infusion" 
OR "product-service system" OR "PSS") AND TITLE-ABS-KEY ("digital" OR "IoT" 
OR "Industry 4.0" OR "smart" OR "platform") AND DOCTYPE(ar) AND LANGUAGE(English)
```

---

## Citation

If you use this repository or the knowledge graph in your research, please cite:

```bibtex
@phdthesis{pachori2025digitalservitization,
  author    = {Pachori, Shivangi},
  title     = {A Knowledge-Graph-Based Systematic Literature Review of Digital Servitization},
  school    = {Indian Institute of Management Indore},
  year      = {2025},
  type      = {Executive Doctoral Programme in Management},
  url       = {https://github.com/shivangipachori/digital-servitization-KG-SLR}
}
```

---

## Author

**Shivangi Pachori**
Executive Doctoral Programme in Management (EDPM)
Indian Institute of Management Indore
ID: 2025EDPM16

---

## License

This repository is licensed under the [MIT License](LICENSE). You are free to use, adapt, and build upon this work with attribution.

---

*Last updated: June 2026*
