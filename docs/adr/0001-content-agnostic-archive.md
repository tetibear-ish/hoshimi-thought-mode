# The archive is content-agnostic

A thought-capture system whose canonical example is "I want to die (10)" contains zero
content detection, zero flags, and zero crisis resources. This is deliberate, and it will
look like an oversight to anyone who finds it later.

Three mechanisms were considered and each was rejected:

- **A local classifier** that recognizes distressing content and adapts the interface.
  Rejected because it hands a model the authority to label your sentences as crisis
  material and act on that label — the precise failure the project's epistemic-provenance
  commitment exists to prevent, applied to the most sensitive content in the archive. A
  system that reads what you write in order to decide how to treat you is a system you
  have to perform for.
- **An explicit user-set flag** on a Thought. Considered at length. Its only coherent
  behaviour would have been "never surface this unprompted," and the owner chose not to
  have it rather than carry a mechanism whose effect was that narrow.
- **A single static resource line** in the contemplative queue, triggered by nothing.
  Rejected last, on the grounds that the system should make no response to content at all,
  consistently.

## Consequences

Nothing in the system inspects what a thought says in order to treat it differently.
Capture never interrupts, never withholds, never escalates, and never adapts to content.
Every thought is stored, counted, and surfaced by exactly the same rules.

This is the deliberate posture of a private, single-owner archive, chosen by its owner.
Safety is located in the person's own life and relationships, not in software that reads
their sentences. **Do not add content inspection here without reversing this decision
explicitly.**
