# Hoshimi Thought Mode — Specification

**Version:** 0.2 — design settled, implementation not started.
**Date:** 2026-11-11
**Supersedes:** v0.1, which was distilled from a [design conversation](https://chatgpt.com/share/6af2913c-be70-83e9-a28c-74d822901d98) and has since been corrected on several substantial points.
**Companions:** [`GLOSSARY.md`](./GLOSSARY.md) for vocabulary, [`docs/adr/`](./docs/adr/) for decisions whose rationale would otherwise be lost.
**ESL specification:** v1.1, vendored at [`docs/esl.md`](./docs/esl.md) — a snapshot taken 2026-11-11 of `~/Training/see/docs/esl.md`, which remains upstream.

## 1. What this is

An Emacs system for capturing recurring thoughts the moment they arrive, tracking how often they recur, relating them to one another, and — optionally and without pressure — examining them through Pali contemplative lenses.

It is not a journal and not a note database. It is an epistemic memory system: one that keeps separate what you said, what a model inferred about it, and how much warrant anyone has for either.

### Commitments

These constrain every feature below. Where a later section appears to violate one, the section is wrong.

- **What you typed is never destroyed.** Normalization, transpilation, splitting, merging, and interpretation all add; none overwrite.
- **Model output is a proposal.** It carries its origin and awaits your acceptance. It never becomes a fact about you by default.
- **Capture cannot be blocked by a model.** In v1 this is true by construction: the capture path contains no model call of any kind.
- **Similarity is not identity.** No model ever decides that two of your thoughts are one. See [ADR-0002](./docs/adr/0002-similarity-proposes-exact-keys-decide.md).
- **Frequency is not endorsement.** A thought recurring often is not one you agree with, want, or are failing to resolve.
- **The archive is content-agnostic.** Nothing inspects what a thought says in order to treat it differently. See [ADR-0001](./docs/adr/0001-content-agnostic-archive.md).
- **Your vocabulary stays yours.** Functions, Categories, and ESL roots are authored by you. Models may propose entries; they may never mint them.

## 2. Scope

**v1 ships capture and nothing else.** No models, no SQLite, no Atlas, no contemplative queue, no transpilation. It is complete and useful on its own, and it is the floor everything else is built on.

| Layer | Requires | Status |
|---|---|---|
| Capture, recurrence counting, history | Emacs 31 only | **v1** |
| ESL transpilation, `.epist`, Categories | A local generative model | Later |
| Similarity, alias proposals, Merge review | A local embedding model + SQLite | Later |
| Thought Atlas | Embeddings + a projection | Later |
| Contemplative Digestion Queue | Nothing technically; deliberately deferred | Later |

Nothing after v1 may alter the capture path's guarantee of working offline and instantly.

## 3. Domain model

Vocabulary is defined in [`GLOSSARY.md`](./GLOSSARY.md). Structure:

- A **Thought** is unique and persists across its arisings. It has three independent parts:
  - a **ULID**, its identity, which never changes;
  - a **Label**, its display wording, freely editable and never matched against;
  - a set of **Match Keys**, the normalized strings that file captures into it, unique across the whole archive.
- An **Occurrence** is one arising: when it happened and the words it arrived in, as a single record. It belongs to **exactly one** Thought, always.
- A **Function** is what a Thought does for you. A Thought *serves* zero or more; a Function is served by many.
- A **Category** is a class of ESL compilation failure exhibited by a Thought. Zero or more per Thought.

Labels being editable while Match Keys decide filing is what makes the archive restructurable without rewriting history. Correcting a typo in a Label is a display change; joining two Thoughts that should have been one is a **Merge**.

Occurrences having exactly one owner is deliberate: it makes every count a count of real events, and removes any need to reconcile unique events against shared assignments.

## 4. Storage

`~/hoshimi/` is a git repository with **no remote configured**. Nothing in the path or in any filename describes what the archive contains.

```
~/hoshimi/
├── thoughts/     01JC7K8FQ3M2V9T4XZB6NRWPAS.org
├── functions/    01JC7KA2…org
├── categories/   01JC7KB9…org
└── lexicon/      roots.org
```

ULIDs serve as both identity and filename. Human text lives only inside files, never in paths — which keeps it out of `ls`, out of git logs, out of your buffer list and recent-file history. Browsing happens through the mode's commands.

Org files are **authoritative**. The SQLite index and every model-generated artifact are **regenerable** and may be deleted at any time without loss. The one exception: an ESL form you corrected by hand is authored, not generated, and is never regenerated over. `.epist` records carry provenance per record so the two are distinguishable.

### 4.1 Thought file

```org
:PROPERTIES:
:ID:       01JC7K8FQ3M2V9T4XZB6NRWPAS
:CREATED:  [2026-11-11 Wed 22:24]
:END:
#+TITLE: I want to die.

* Match Keys
- i want to die

* Occurrences
** [2026-11-11 Wed 22:24]
:PROPERTIES:
:SOURCE: voluntary-capture
:END:
I want to die.

* Functions
- [[id:01JC7KA2…][Express pain]]

* Categories
- [[id:01JC7KB9…][Mind-reading: -më applied to an external state]]
```

Notes on the format:

- The property drawer precedes `#+TITLE:`, following the org-roam convention already installed on this machine.
- The Occurrence body holds the display-normalized text. A `:RAW:` property is added **only when** what you typed differs from it, so the common case stays clean and the uncommon case stays lossless.
- Match Keys is a section rather than a property because keys are multi-word strings and Org's property accumulation mangles those. The session index scan reads only each file's head.
- Function and Category links are written into the Thought file on assignment. SQLite holds a derived copy for querying. Losing SQLite loses nothing you authored.
- `SOURCE` is a small closed vocabulary. v1 emits only `voluntary-capture`; `contemplation` is reserved for occurrences recorded during an examination session.
- **No ambient context is ever recorded** — not the buffer, file, project, or location you were in. The archive records thoughts, not surveillance of your day.

## 5. Capture

`SPC t t` (alias) or `SPC n t t` opens a minibuffer prompt. Vertico renders it.

1. You type. **Nothing is offered, ranked, or suggested.** There is no completion UI.
2. `RET` commits. `C-g` or `C-c C-k` cancels.
3. The text is display-normalized, then match-folded.
4. The fold is looked up against all Match Keys. Exact hit → a new Occurrence is appended to that Thought. No hit → a new Thought is created, with the fold as its first Match Key and the display form as its Label.
5. The minibuffer states which happened: `recorded · 11th occurrence` or `new thought recorded`.

A Thought is a **single line**. Longer reflection belongs in the Thought's Org body or a contemplation note, never in the string being matched on.

The design conversation originally proposed completions ranked by vector similarity. That is deliberately gone: it required a model in the capture path, and it asked you to adjudicate whether two thoughts were the same at the exact moment you were trying to get one out of your head. Similarity now does its work afterwards, on request — see §7.

### 5.1 Normalization

Two distinct transformations, because the form you read and the form you match on want different aggressiveness.

**Display form** — what is stored and shown:
- capitalize the sentence-initial letter;
- capitalize standalone `i`;
- collapse internal whitespace runs, trim the ends;
- append `.` only when no terminal `.`, `?`, or `!` is present.

**Match fold** — derived from the display form, stored as a Match Key:
- downcase;
- strip all punctuation;
- collapse whitespace.

So `I want to die!!!` and `i want to die.` fold together, correctly. `I wanna die` does not — different words, and joining them is Merge's job, not a fold's.

A consequence left deliberately unhandled: Labels are free text, so you can edit a Label to a string that a later capture reproduces exactly, and get a second Thought — because Labels are not keys. Capture stays silent and deterministic; Merge resolves it whenever you notice.

## 6. Merge and Split

Both are refinements of interpretation. Neither rewrites history, and both are undoable.

**Split** divides a Thought into children because it carried several distinct meanings. The parent is retained; each Occurrence is assigned to exactly one child or left on the parent; Match Keys are distributed among the children.

**Merge** combines Thoughts that should have been one. All Occurrences move to the survivor, each keeping its own verbatim text; the absorbed Thought's Match Keys move to the survivor; both originals remain on disk as redirects rather than being deleted.

Occurrence-level embeddings make a further capability possible later: a Thought recorded many times may carry different contextual meaning each time, and clustering its Occurrence vectors can reveal two latent groups inside what looked like one Thought. The system then *proposes* a Split. It never performs one.

## 7. Similarity

Similarity proposes; exact keys decide. Rationale and the rejected alternative are recorded in [ADR-0002](./docs/adr/0002-similarity-proposes-exact-keys-decide.md).

`SPC n t m` opens the merge-proposal review, computes similarity at that moment, and shows nothing unless you went looking. No timers, no startup scan, no background model calls, no notifications.

A proposal reads: *"`i wanna die` may belong with `I want to die.`"* On acceptance, that fold is written as a **Match Key** on that Thought. Every future capture of `I wanna die` then matches **exactly, deterministically, and offline** — no model involved. The model teaches the index once and leaves.

The embedding model is deliberately unchosen. See §12.

## 8. ESL and the `.epist` record

ESL v1.1 — **Epistemic Synthetic Language** — is specified in [`docs/esl.md`](./docs/esl.md) and is a dependency of this project, not a part of it. The vendored copy is a snapshot; `~/Training/see/docs/esl.md` is where it is authored, and the two can drift. In summary: a synthetic, obligatory morphology over a fixed six-slot template, `ROOT-ACCESS-EVIDENCE-ASPECT-FRAME-ENDORSEMENT`, built so that omitting provenance is a structural malformation rather than a stylistic choice.

Transpilation is a **compilation**: it either produces a well-formed ESL unit or emits diagnostics. The `.epist` record stores both.

```json
{
  "format": "hoshimi.epist",
  "version": "0.2",
  "thought_id": "01JC7K8FQ3M2V9T4XZB6NRWPAS",
  "source": { "language": "en", "text": "I am angry." },
  "esl": {
    "version": "1.1",
    "operator": "ka",
    "surface": "kodha-më-prat-jā-fi-che",
    "slots": {
      "root": "kodha", "root_language": "pali",
      "access": "më", "evidence": "prat",
      "aspect": "jā", "frame": "fi", "endorsement": "che"
    },
    "provenance": "model"
  },
  "diagnostics": [],
  "interpretations": []
}
```

- `operator` sits outside `slots` because `vo` (Boundary) **truncates** the template to ROOT + ACTOR, bypassing evidence and endorsement entirely — a boundary is a performative act, not a descriptive report. The operator therefore determines the form's shape and cannot be one slot among six.
- `provenance` is `model` or `authored`. Model-generated records are regenerable; authored ones are never regenerated over.
- `root_language` is `pali` or `english`. **English roots are permitted**, so transpilation is never blocked waiting on lexicon work. Pali is preferred where a root exists, and upgrading later is free: author the Pali root, re-transpile, and any model-generated record picks it up.

### 8.1 Lexicon

`lexicon/roots.org` maps concepts to roots. Each entry carries **root, IPA, and English gloss** — which makes the lexicon double as the tooltip source described in §11.

You author roots. A model may *propose* a Pali root from Pali sources, recorded as a proposal awaiting acceptance, never auto-adopted. This is the vocabulary you would least want drifting into a model's choices.

## 9. Categories as compiler diagnostics

A Category is a class of ESL compilation failure — the characteristic way a statement's form outruns the warrant available for it.

The taxonomy is **seeded from ESL v1.1's own documented failure cases**, which are your language's diagnostics rather than an imported framework:

| Failure | Diagnostic |
|---|---|
| `prat` applied to an unobserved cause | Direct-encounter claim for something only inferred |
| A past event reported with `prat` | Memory reported as present encounter; should be `me` |
| Report data given `prat` | Testimony reported as direct encounter; should be `te` |
| A direct sensation labelled `sa` | Sensation misfiled as inference merely for being subjective |
| `nu` where the source is known | Provenance confused with confidence |
| `-më` applied to another's state | Mind-reading fallacy; manifestational `-në` required |
| Non-`-un` endorsement inside a `uf` frame | Epistemically Illegal under ESL's compiler constraint |

Categories beyond this set emerge empirically: the transpiler records each failure with its concrete reason, and recurring failure shapes get promoted to named Categories **by you**, never automatically.

A Category describes a statement's transpilation, **never the thinker**. Concretely: no per-person error score, no archive-wide totals, no streaks, no trends over time. Per-category occurrence counts are visible when you go looking; nothing aggregates into a number meaning "how wrong I am."

This also gives §10's decentering real machinery. *"I'm an inadequate person"* fails compilation in a specific, nameable way — a global trait claim asserted from introspective access alone. *"A thought about my inadequacy has arisen"* compiles cleanly. The distortion is the diagnostic, named formally rather than clinically.

## 10. Thought Atlas

A map of how Thoughts relate, with two views you toggle between:

| View | Basis |
|---|---|
| **Similarity Map** | Embedding geometry — e.g. a UMAP projection to 2D, with edges from actual cosine similarity above a threshold |
| **Knowledge Graph** | Explicitly declared relations: `derived-from`, `contradicts`, `refines`, `same-theme-as` |

2D positions are an approximation for browsing, never a measure of psychological distance. The underlying similarities stay inspectable rather than hidden behind the projection.

Each node additionally shows two independent dimensions: **position** for semantic proximity, and **outline** for contemplative readiness. This is what lets you open the map, notice a cluster that has never been examined, and gently choose one.

Selecting a node shows its Occurrence history, Functions, Categories, related Thoughts, and Label. Built standalone — not on org-roam, whose model is documents-and-links and whose schema would be fought at every core operation.

## 11. Contemplative Digestion Queue

For the gap between capturing a thought and being ready to contemplate it.

- **Readiness is not importance.** A thought can matter enormously and be unapproachable today.
- **Readiness is chosen by you**, never assigned by a model.
- Thoughts are **never ranked by how spiritually advanced or deficient** they make you look. That would turn contemplation into another self-evaluation system.
- **"Digested" is not a terminal state.** Understanding is not always a discrete event, and a thought can become relevant again in a new context.
- Unexamined thoughts are **contemplative potential**, not debt. ("Contemplative debt" was considered and rejected as framing.)

### 11.1 States

```
Captured → Acknowledged → Approachable → In contemplation
   ↑___________________________________________|
            may return to rest at any time
```

| State | Meaning |
|---|---|
| Captured | The thought has been preserved |
| Acknowledged | I recognize this thought exists |
| Approachable | I can tolerate gently examining it |
| In contemplation | I am exploring its conditions and meaning |

Readiness is kept **separate from ESL's FRAME and ENDORSEMENT** slots, with no derivation in either direction. The resemblance is real — a `uf` frame with mandatory `-un` endorsement is a claim recorded without assertion — but endorsement is a property of a claim's warrant while readiness is a property of your present capacity. A thought can be `-che` and unbearable, or `-un` and perfectly comfortable to look at.

### 11.2 Lenses

Each lens offers a question, never a verdict. These are interpretive prompts, not explanations of why a thought occurred.

| Pāli | Gloss | What it opens |
|---|---|---|
| **Dukkha** | Unsatisfactoriness | What is painful, burdensome, or hard to accept? |
| **Vedanā** | Feeling tone | Is this pleasant, unpleasant, or neutral? |
| **Taṇhā** | Craving | What am I longing for, or wishing gone? |
| **Upādāna** | Clinging | What am I holding as essential to my well-being? |
| **Anicca** | Impermanence | How does this arise, change, and subside? |
| **Anattā** | Not-self | Am I treating an experience as though it defines me? |
| **Karuṇā** | Compassion | Can I acknowledge suffering without adding condemnation? |
| **Sati** | Mindfulness | (general observational anchor) |

A sufficient note can be as short as *"An unpleasant feeling arises when I imagine opening my bank account."* No requirement to understand why. No requirement to resolve anything.

### 11.3 Contemplation history

Accumulates in the Thought's own Org file, alongside its Occurrences, without replacing anything:

```org
* Contemplation
:PROPERTIES:
:READINESS: APPROACHING
:END:

** [2026-11-11 Wed 22:30]
Lens: Vedanā
Observation: Thinking about this feels unpleasant.
Depth: Recognition only.

** Future inquiry
- What part of the task is unpleasant?
- What do I fear discovering?
```

### 11.4 Sessions

`SPC n t e` begins examining a Thought. One question at a time, from one lens, offering exactly three things: record an observation, defer, or end. **It never insists you continue.**

### 11.5 Observation versus rumination

**Decentering** — recognizing a thought as a mental event rather than an unquestionable fact — is the capability worth supporting. *"A thought about my inadequacy has arisen"* leaves space that *"I'm an inadequate person"* does not.

Prolonged analysis can decay into **rumination**: examining distress repeatedly without understanding or relief. The system prompts reflection on whether examination is still helping — *"Is examining this helping you understand it, or are you becoming more entangled in it?"* — rather than deciding unilaterally what you need.

## 12. Pali vocabulary and tooltips

Pali terms are the primary vocabulary throughout the interface, never anglicized labels. Every tooltip carries three things in order:

1. the **Pali term**, diacritics preserved — `ā`, `ṇ`, `ṭ`, `ṃ` are never silently simplified;
2. an **IPA transcription**;
3. a **concise English gloss**.

```
Dukkha
/ˈduk.kʰɐ/
Suffering, stress, or unsatisfactoriness.
```

This applies to ESL's grammatical morphemes too — `më`, `prat`, `jā`, `fi`, `che` and the rest all need the same treatment, independent of whether a given ROOT is Pali or English.

Implementation: `help-echo` text properties for mouse hover; Eldoc for keyboard-only access to the same three-part content. One consistent pronunciation convention throughout, including vowel length and aspiration.

**Principle:** Pali first, pronunciation and meaning one gesture away. The interface is a place of gradual language acquisition as well as contemplative practice.

## 13. Emacs integration

- Emacs 31.1 — built-in `sqlite` and `json-parse` mean no external dependencies for the later layers either.
- Symbols are prefixed `hoshimi-thought-`: `hoshimi-thought-capture`, `hoshimi-thought-mode`, `hoshimi-thought-directory`.
- The package lives in its own git repository with no remote, loaded by Doom via `package!` with a `:local-repo` recipe.
- Doom glue (the `packages.el` entry and the keymap) belongs to `~/.config/doom/`, which is already a symlink into the private dotfiles repository and already tracked in its `run.sh`. No new `run.sh` entry is needed.
- **No SQLite in v1.** The only lookup v1 needs is match-fold → Thought, built by scanning Org file heads once per session and held in memory, updated on write. SQLite arrives with embeddings, which are the first thing that genuinely cannot live in a header scan.

### 13.1 Commands

v1 ships four. The rest are bound when the features behind them exist, not reserved in advance.

| Binding | Action | Ships |
|---|---|---|
| `SPC t t` / `SPC n t t` | Capture a thought | **v1** |
| `SPC n t l` | List Thoughts by recurrence | **v1** |
| `SPC n t h` | Occurrence history for a Thought | **v1** |
| `SPC n t o` | Open a Thought's Org file | **v1** |
| `SPC n t m` | Review merge and alias proposals | Later |
| `SPC n t a` | Open the Thought Atlas | Later |
| `SPC n t s` | Split a Thought | Later |
| `SPC n t d` | Open the Contemplative Digestion Queue | Later |
| `SPC n t e` | Begin examining a Thought | Later |

`SPC t` is Doom's reserved toggle prefix, so the command set lives under `SPC n t` beside your existing seeds at `SPC n s`. `SPC t t` is kept as a direct capture alias.

### 13.2 Tests

ERT coverage for three things, and nothing else in v1:

1. display normalization, including standalone versus mid-word `i`, existing terminal punctuation, repeated punctuation, unicode, and empty input;
2. match folding;
3. the capture filing decision — exact hit appends an Occurrence, miss creates a Thought.

These are pure functions with real edge cases, and they are where incorrectness is both likely and invisible: a bug silently fragments or wrongly joins the archive, and you would notice months later.

## 14. Deliberate non-features

Recorded so that an absence is not mistaken for an oversight.

- **No content inspection of any kind.** No classifier, no keyword matching, no flag, no resource line, no special-casing of distressing material. Full rationale in [ADR-0001](./docs/adr/0001-content-agnostic-archive.md).
- **No semantic matching at capture.** Rationale in [ADR-0002](./docs/adr/0002-similarity-proposes-exact-keys-decide.md).
- **No completion UI.** Removed with the capture-time similarity it depended on.
- **No ambient context capture.** Nothing about what you were doing when a thought arrived.
- **No shared Occurrence ownership.** Every count is a count of real events.
- **No remote on the archive repository.** Added only as an explicit decision, never inherited by default.
- **No model-authored vocabulary.** Functions, Categories, and roots are yours.

## 15. Open questions

Genuinely undecided, and deliberately so.

- **Embedding model.** Ollama is not installed. The choice gets made against a real archive rather than an imagined one.
- **The queue's surfacing heuristic.** Recurrence, unresolved questions, connectivity, and time-unexamined are the inputs; their weighting is unknown.
- **Atlas projection parameters** and redraw cadence.
- **Split and Merge undo mechanics** beyond retaining parents and leaving redirects — the archive's git history may be sufficient.
- **ESL ROOT lexicon coverage.** English roots mean this never blocks, but the Pali lexicon's growth path is unplanned.
