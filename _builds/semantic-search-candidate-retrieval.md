---
title: "Semantic Search for Candidate Retrieval"
eyebrow: "Virtual Internships"
eyebrow_tags: "pgvector &middot; HNSW &middot; Kafka &middot; BigQuery"
tone: 1
show_in_writing: true
date: 2026-07-09
excerpt: >-
  Rebuilding candidate search as a semantic system on pgvector and HNSW, and the event-driven
  pipeline that keeps it consistent with two other systems that never agree on candidate state
  at the same time.
---

Candidate search used to mean keyword filters: exact skill matches, exact title matches, missing every good candidate whose resume used different words for the same thing. I rebuilt it as a semantic search system on pgvector with an HNSW index, so retrieval ranks by meaning instead of exact string matches, and typeahead suggestions stay fast enough to feel instant.

The part that mattered most wasn't the embedding model, it was keeping the index consistent with two other systems that never agreed on candidate state at the same time. Profile edits land in MySQL, personalization signals live in PostgreSQL, and both needed the vector index to reflect the latest state within seconds, not minutes. I designed an event-driven pipeline on Kafka that syncs vector profiles and personalization signals as they change, so the search index and the recommendation engine are reading from the same picture of a candidate instead of two stale ones.

On top of that, the recommendation system itself needed real signal, not just a vector match. I wired engagement, content, and cohort data through a BigQuery and Google Analytics pipeline to drive personalized ranking, which took click-through rate on recommended matches up 2x. The API path for all of this, search plus personalization, holds under 300ms, because none of it is useful if it's not fast enough to be part of the page load.
