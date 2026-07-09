---
title: About
permalink: /about/
description: Applied AI & Software Engineer. Search, AI personalization, and backend systems in production.
---

I'm an **Applied AI & Software Engineer**. I build search and AI personalization systems that hold
up in production: semantic search over pgvector and HNSW, event-driven data pipelines, and LLM
workflows that get cheaper as they scale, not more expensive.

I own these features end-to-end. Index design, embedding pipelines, prompt structure, the rate
limits that keep an LLM workflow affordable, and the event-driven sync that keeps a vector store
and a database from drifting apart. The model call is one line. Building around it so it survives
real users and real data is the actual job.

---

<div class="about-row">
  <span class="label">Title</span>
  <span class="value">Applied AI &amp; Software Engineer</span>
</div>
<div class="about-row">
  <span class="label">Open to</span>
  <span class="value">Senior/Mid AI &amp; SDE roles &middot; Remote or hybrid</span>
</div>
<div class="about-row">
  <span class="label">Education</span>
  <span class="value">B.Tech, IIT Roorkee (2019 &ndash; 2023)</span>
</div>
<div class="about-row">
  <span class="label">AI / LLM</span>
  <span class="value">LangChain &middot; pgvector &middot; HNSW &middot; embeddings &middot; prompt design</span>
</div>
<div class="about-row">
  <span class="label">Backend</span>
  <span class="value">Java &middot; TypeScript &middot; Spring Boot &middot; Node.js &middot; Python &middot; Kafka</span>
</div>
<div class="about-row">
  <span class="label">Data</span>
  <span class="value">PostgreSQL &middot; MySQL &middot; Redis &middot; BigQuery &middot; Google Analytics &middot; MongoDB</span>
</div>
<div class="about-row">
  <span class="label">Infra</span>
  <span class="value">AWS &middot; Docker &middot; Git</span>
</div>

---

## How I work

I like the parts of an AI feature that aren't the model call. The prompt structure that keeps a
LangChain pipeline from trusting a resume's text too much. The rate limiter that keeps a pipeline's
cost flat as usage grows. The Kafka consumer that keeps a vector index and a recommendation engine
in sync.

The most underrated skill in shipping AI is **knowing what to constrain the model to.** An LLM is
good at pulling structure out of a messy document. It's bad at deciding what to trust on its own.
Sanitized prompts, rate limits, and a real audit trail instead of blind trust, that's most of the job.

## What I'm into right now

- Embedding pipelines that cost cents instead of dollars
- pgvector at the point where HNSW vs IVFFlat actually matters
- Event-driven architectures that keep a vector store and a relational database from drifting apart
- LLM pipelines that fail safely when the model is unsure, instead of confidently making something up
- Secure prompting for LLMs that process documents from people I don't control

## Get in touch

<ul class="contact-links">
  <li><a href="mailto:{{ site.email }}"><strong>Email</strong> &middot; {{ site.email }}</a></li>
  <li><a href="https://github.com/{{ site.github_username }}" target="_blank" rel="noopener"><strong>GitHub</strong> &middot; {{ site.github_username }}</a></li>
  <li><a href="https://linkedin.com/in/{{ site.linkedin_username }}" target="_blank" rel="noopener"><strong>LinkedIn</strong> &middot; {{ site.linkedin_username }}</a></li>
  <li><a href="{{ site.resume_url }}" target="_blank" rel="noopener"><strong>Resume</strong> &middot; download</a></li>
</ul>
