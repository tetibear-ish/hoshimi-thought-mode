# Similarity proposes, exact keys decide

Embeddings are available to this system, and capture still files thoughts by exact string
match against a normalized fold. Anyone who notices will assume semantic matching was
simply not implemented yet. It was considered, specified, and rejected.

The goal is real: `I wanna die` should end up filed with `I want to die`. The question was
only what performs that join, and when.

**Matching semantically at capture time was rejected** for three reasons:

1. **Negation.** Embedding models are weak on it. `I want to die` and `I don't want to die`
   sit very close together in vector space, because negation barely moves the vector while
   completely inverting the meaning. Any threshold loose enough to catch `I wanna die` also
   catches `I don't want to die`. This is a property of the representation, not a cutoff
   that can be tuned.
2. **It puts a model in the capture path**, breaking the guarantee that capture works
   instantly and offline. A spontaneous thought must never wait on inference.
3. **It lets a model decide two of your thoughts are one** — silently redirecting new
   wording into an existing Thought.

## What replaces it

Similarity runs after the fact and only on request, proposing aliases and merges. On
acceptance, the proposed fold is written as a **Match Key** on that Thought. Every
subsequent capture of that wording then matches exactly, deterministically, and offline,
with no model involved.

## Consequences

The model teaches the deterministic index once and then leaves. Matching gets more accurate
the longer the archive is used, while the capture path stays model-free permanently. The
cost is that a near-duplicate is recorded as its own Thought until you review proposals —
accepted, because the alternative puts an unreviewable judgement at the one moment you are
trying to get a thought out of your head.
