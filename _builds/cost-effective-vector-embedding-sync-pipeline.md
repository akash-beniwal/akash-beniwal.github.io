---
title: "Building a Cost-Effective Vector Embedding Sync Pipeline for Semantic Search"
eyebrow_tags: "pgvector &middot; OpenAI Embeddings &middot; Subscribers &middot; BullMQ"
tone: 1
show_in_writing: true
date: 2026-07-11
excerpt: >-
  Keeping a candidate's vector fresh sounds like it should mean constant re-embedding.
  It gets cheap once you stop guessing what changed and start tracking it.
---

<svg width="0" height="0" style="position: absolute">
  <filter id="sketchy" x="-20%" y="-20%" width="140%" height="140%">
    <feTurbulence type="fractalNoise" baseFrequency="0.012 0.028" numOctaves="2" seed="7" result="noise" />
    <feDisplacementMap in="SourceGraphic" in2="noise" scale="3.5" xChannelSelector="R" yChannelSelector="G" />
  </filter>
</svg>
<style>
  /* Scoped to this article: the queuing-architecture sketch diagram. */
  .sketch-diagram { display: flex; flex-direction: column; align-items: center; margin: 1.4rem 0 1.6rem; }
  .sketch-card {
    position: relative; background: var(--bg-elev); border-radius: 9px; padding: 0.95rem 1.3rem;
    text-align: center; max-width: 380px;
  }
  .sketch-card::before {
    content: ''; position: absolute; inset: -3px; border-radius: 9px; border: 2px solid var(--text);
    filter: url(#sketchy); pointer-events: none;
  }
  .sketch-card:nth-child(4n+1) { transform: rotate(-0.5deg); }
  .sketch-card:nth-child(4n+3) { transform: rotate(0.4deg); }
  .sketch-title { display: block; font-family: var(--mono); font-size: 0.85rem; font-weight: 700; color: var(--text); }
  .sketch-detail { display: block; font-size: 0.78rem; color: var(--text-muted); margin-top: 0.25rem; line-height: 1.5; }
  .sketch-detail code { font-family: var(--mono); font-size: 0.9em; background: none; padding: 0; color: inherit; }
  .sketch-arrow { position: relative; width: 2px; height: 2rem; background: var(--text-faint); filter: url(#sketchy); margin: 0.1rem 0; }
  .sketch-arrow::after {
    content: ''; position: absolute; bottom: -1px; left: 50%; transform: translateX(-50%);
    border-left: 5px solid transparent; border-right: 5px solid transparent; border-top: 7px solid var(--text-faint);
  }
  .sketch-label {
    font-family: var(--mono); font-size: 0.72rem; color: var(--text); text-transform: uppercase;
    letter-spacing: 0.03em; margin: 0.55rem 0 0.2rem;
  }
  .sketch-substeps { display: flex; flex-wrap: wrap; gap: 0.3rem; margin-top: 0.6rem; justify-content: center; }
  .sketch-substeps span {
    font-family: var(--mono); font-size: 0.68rem; color: var(--text-muted); border: 1.2px dashed var(--border-strong);
    border-radius: 8px; padding: 0.15rem 0.5rem;
  }
</style>

A candidate's profile changes constantly: a new skill, a finished certification, an edited work description. Every one of those edits means the vector sitting in the database no longer matches who they actually are.

The real question isn't whether to keep vectors fresh. It's how you know which profiles changed, without checking all of them.

## Knowing what changed

<p class="flow-line">entity changes → decorator checks watched columns → sync-state queue marks candidate dirty</p>

A decorator sits on every entity that makes up a profile, skills, education, work experience, certifications, and tells a single subscriber which columns actually matter. A skill's proficiency level is watched. An unrelated internal field isn't. Touch a column nobody's watching, and nothing happens at all, no dirty mark, no queue, no call.

When a watched column does change, the subscriber doesn't touch OpenAI or pgvector itself. It sends one message to a sync-state queue, whose only job is working out which candidate this actually affects and marking them dirty.

That queue earns its keep on the cases that aren't obvious. Most changes are direct, a candidate edits their own work history, and the affected ID is right there. But one change to a batch can affect hundreds of candidates at once. Figuring out which hundred means a database query, and that query can't safely run inside the subscriber's own transaction. So the sync-state queue resolves that fan-out asynchronously, outside the transaction that triggered it, then writes the result to a small tracking table.

| Column | Purpose |
|---|---|
| `candidate_id` | Which candidate this row tracks |
| `last_updated_on` | Set whenever a watched column changes |
| `last_synced_on` | Set after a successful embed, null if never synced |

A candidate is dirty whenever `last_updated_on` is newer than `last_synced_on`. That single comparison is the entire signal the rest of the pipeline runs on.

## Draining the dirty set

<p class="flow-line">sync-state queue marks dirty → cron drains every 10 min → embedding queue rebuilds, embeds, upserts</p>

A cron checks that tracking table every 10 minutes. Whatever's dirty gets pushed onto a second queue, the one that actually does the expensive part: rebuild the candidate's text, embed it, upsert the vector into pgvector and mark it synced.

The trade-off is a 10-minute window where a candidate's vector can be stale. In exchange, someone who can't decide between "Software Engineer" and "Senior Software Engineer" and edits their title four times in five minutes still costs one embedding call, not four, because all four edits are still sitting in the same dirty row when the cron finally runs.

Worth calling out on its own: the embedding call itself goes out in sub-batches of 25 candidates per request, not one request per candidate. Fewer, larger requests keep the pipeline further from OpenAI's per-minute rate limits, not just its per-token price, which matters most exactly when a busy sync window has the most candidates queued up at once.

That is the whole pipeline, end to end:

<div class="sketch-diagram">
  <div class="sketch-card">
    <span class="sketch-title">entity change</span>
    <span class="sketch-detail">direct edit, or a batch fan-out</span>
  </div>
  <div class="sketch-arrow"></div>
  <div class="sketch-card">
    <span class="sketch-title">sync-state queue</span>
    <span class="sketch-detail">resolves + marks dirty</span>
  </div>
  <div class="sketch-arrow"></div>
  <div class="sketch-card">
    <span class="sketch-title">tracking table</span>
    <span class="sketch-detail"><code>candidate_id</code>, <code>last_updated_on</code>, <code>last_synced_on</code></span>
  </div>
  <div class="sketch-label">cron, every 10 min</div>
  <div class="sketch-arrow"></div>
  <div class="sketch-card">
    <span class="sketch-title">embedding queue</span>
    <div class="sketch-substeps">
      <span>rebuild text</span><span>cache check</span><span>embed</span><span>upsert</span>
    </div>
  </div>
</div>

## Making it cheap

<p class="flow-line">relevant-only sync + coalescing + caching (a cheap model too) → small bill, most of it invisible</p>

Three levers do most of the work, and none of them is the embedding model.

- **Watch only what matters.** The subscriber fires only on columns that actually feed the embedding text or the filter metadata. Touch a column nobody's watching and nothing happens at all, no dirty mark, no queue, no call.
- **Coalesce instead of reacting.** Dirty-tracking plus the 10-minute drain means only candidates who actually changed get re-embedded, and several edits in one window collapse into a single call.
- **Cache identical text.** Every embedding call checks Redis first, keyed on a hash of the exact text, 1-day TTL, so identical text never reaches OpenAI twice. FNV-1a is the deliberate choice here: a non-cryptographic 64-bit hash is fast and stable, and cryptographic strength buys nothing for "same input, same key."

Two things underneath help without being levers you tune: the vectors live on the same Postgres instance as everything else, so there's no separate vector-database bill, and the model itself is cheap, `text-embedding-3-small` at 1536 dimensions, 6.5x under the large one. Neither is where the cost lived.

The coalescing lever is the one worth doing the math on. Picture the pipeline at a larger, steady-state scale, say 250,000 candidates. Each document is capped at the top 5 entries per section, so it lands around 550 tokens whether a candidate has two years of history or fifteen. A new candidate typically builds their profile in about 8 separate saves: an introduction, an education entry, one or two work entries, skills, a certification, a portfolio piece.

```
naive (embed on every save):
  250,000 × 8 calls × 550 tokens ≈ 1.1B tokens → ~$22.00

coalesced (drained every 10 min, ~2.5 embeds/candidate):
  250,000 × 2.5 × 550 tokens ≈ 343.75M tokens → ~$6.88

≈ 69% fewer embedding operations for the same onboarding burst
```

The dollar figure looks small either way, embedding tokens are just cheap. What actually matters is the call count when it scales: fewer calls means less exposure to rate limits and less queue throughput spent reprocessing text nobody asked to change.

Caching earns its keep in a specific, mechanical case: a sync batch hits a transient failure partway through, a timeout calling OpenAI, a temporary upsert error, and the retry rebuilds the exact same text it already generated an embedding for moments earlier. Without the cache, that retry pays for the same embedding twice. With it, the second attempt is a Redis lookup, not an OpenAI call.

That's a narrower win than it sounds: most retries are rare, and most edits do produce genuinely new text. But it's real money saved on exactly the runs that would otherwise cost double for doing the same work twice.

## Proving it, before it went anywhere

<p class="flow-line">vectors built → validated on an internal endpoint → live product untouched</p>

Before any of this touched the real candidate-facing product, the pipeline shipped its own small test surface: an internal-only similarity search endpoint, metadata filters and all. Its one job was to prove the vectors were good, that ranking by meaning beat the SQL formula it was about to replace, before it went anywhere a host company could see.

Nothing user-facing changed yet. The existing search kept serving while the vectors were built, backfilled, and checked in place. Wiring these vectors into the browse page, and the ranking that rides on top of them, is a story of its own.

That caution carried into the infrastructure too. The HNSW index is created once, before the first row is backfilled, and never needs the periodic rebuild IVFFlat would, so the initial backfill runs the same incremental upsert path as every future sync.

## The one thing to take from this

The expensive version of this pipeline was never the embedding model. It was not knowing what actually changed, and re-processing everything just to be safe.

Watch only what matters, coalesce it, cache it: the cost problem disappears on its own, not because saving money was ever the goal, but because keeping every candidate's vector correct and current was. Cheap is just what that looks like when you're not guessing.
