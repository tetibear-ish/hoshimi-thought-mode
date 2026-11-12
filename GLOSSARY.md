# Hoshimi Thought Mode

An Emacs system for capturing recurring thoughts as they occur, tracking their recurrence, relating them to one another semantically, and optionally examining them through Pāli contemplative lenses. It keeps what was said separate from what a model inferred about it.

## Language

### Thought structure

**Thought**:
A recurring proposition or attitude that persists across the separate times it arises. Each one is unique and has a stable identity of its own.
_Avoid_: Note, entry, idea, item

**Occurrence**:
One arising of a Thought: when it happened together with the words it arrived in. The words and the event are a single record, not two.
_Avoid_: Utterance, instance, repetition, entry, hit

**Function**:
What a Thought does for the person who thinks it. A Thought is said to *serve* a Function — "I want to die" may serve "Express pain."
_Avoid_: Purpose, intent, role, speech act, use

**Label**:
The human-readable wording a Thought is displayed under. Freely editable, and never what the system matches on.
_Avoid_: Title, name, text, canonical form

**Match Key**:
A normalized string that files a captured Occurrence into a particular Thought. Keys are unique across the whole archive, which is what makes each Thought unique.
_Avoid_: Hash, slug, index, fingerprint, alias

**Merge**:
Combining Thoughts that should have been one, moving every Occurrence to a survivor while both originals remain on disk. The inverse of Split, and bound by the same rule.
_Avoid_: Dedupe, combine, collapse, link

**Split**:
Dividing one Thought into two or more child Thoughts because it was carrying several distinct meanings. A refinement of interpretation, never a rewrite of what was recorded.
_Avoid_: Fork, break up, refactor

**Interpretation**:
A claim about what a Thought means that originated from a model rather than from the person. Always carries its origin with it and never silently becomes a fact about the person.
_Avoid_: Classification, verdict, diagnosis, analysis

### Representation

**Epistemic Synthetic Language (ESL)**:
A synthetic language whose obligatory six-slot morphology makes the provenance of a claim structurally mandatory, so that omitting it is a malformation rather than a stylistic choice. Specified externally, at v1.1.
_Avoid_: SEL, Synthetic Epistemiologic Language, DSL, IR, the synthetic language

**Operator**:
The speech act a Thought performs — assertion, inquiry, imagination, attention, relinquishment, or boundary. A closed grammatical class, and distinct from the Function a Thought serves.
_Avoid_: Mood, intent, act type, function

**Root**:
The semantic core of an ESL form, drawn from Pali where a root exists for the concept and from English where none does yet.
_Avoid_: Stem, lemma, token, concept

**Lexicon**:
The authored store mapping concepts to Roots, each entry carrying its Root, IPA, and English gloss.
_Avoid_: Dictionary, vocab, wordlist, glossary

**Epist**:
The record pairing a Thought's original English with its ESL form, the Diagnostics that compiling it produced, and any Interpretations of it.
_Avoid_: Translation, encoding, sidecar

**Diagnostic**:
One recorded failure from compiling a Thought into ESL, naming what was wrong with that particular statement.
_Avoid_: Error, warning, violation, issue

**Category**:
A class of Diagnostic — the characteristic way a statement's form outruns the epistemic warrant available for it. A property of statements, never a measure of the thinker.
_Avoid_: Distortion, fallacy, theme, tag, bucket

**Epistemic Provenance**:
The principle that every claim in the system carries where it came from and what warrants believing it.
_Avoid_: Metadata, attribution, audit trail

**Phenomenological Archive**:
The whole durable record of experience as it presented itself, held prior to and independent of any interpretation of it.
_Avoid_: Journal, log, database, corpus

### Relating thoughts

**Thought Atlas**:
The navigable map of how Thoughts relate to one another.
_Avoid_: Graph view, visualization, mind map

**Similarity Map**:
The view of the Atlas whose arrangement derives from embedding geometry, where proximity reflects semantic resemblance rather than psychological distance.
_Avoid_: Cluster view, embedding space, projection

**Knowledge Graph**:
The view of the Atlas built from explicitly declared relations between Thoughts, such as `derived-from`, `contradicts`, `refines`, and `same-theme-as`.
_Avoid_: Links, backlinks, network

### Contemplation

**Contemplative Digestion Queue**:
The place where Thoughts awaiting examination remain unresolved without being neglected.
_Avoid_: Inbox, backlog, to-do, triage queue

**Readiness**:
How approachable a Thought currently is for examination, chosen by the person and never assigned by a model. Independent of how important the Thought is.
_Avoid_: Priority, status, urgency, score

**Lens**:
A Pāli concept offered as a question for examining a Thought, never as an explanation of why it occurred.
_Avoid_: Framework, prompt, template, filter

**Contemplative Potential**:
The latent understanding held in Thoughts not yet examined. Measured as possibility, never as debt or obligation.
_Avoid_: Contemplative debt, backlog size, unprocessed count

**Decentering**:
Recognising a Thought as a mental event that has arisen rather than as an unquestionable fact about oneself.
_Avoid_: Detachment, distancing, reframing

**Rumination**:
Repeatedly examining distress without gaining understanding or relief. The failure mode that contemplation can decay into.
_Avoid_: Overthinking, spiralling, dwelling
