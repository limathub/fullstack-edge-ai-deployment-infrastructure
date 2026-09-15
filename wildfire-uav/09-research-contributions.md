[← Back to overview](../README.md) · **Language:** English | [한국어](./ko/09-research-contributions.md)

# 09 — Research Contributions

> Five findings that outgrew the engineering context they came from — and the specific
> experiments that would turn them into a paper.

---

## Framing

Most of this repository documents engineering decisions. This chapter is different: these are
**mechanism-level findings about how open-vocabulary detectors fail on amorphous classes**
(smoke, fog, plumes — things without stable boundaries or canonical shape), which generalize
past the wildfire application they were discovered in.

They were produced as a byproduct of building a product, not as a research project. That
origin is their strength — every one is grounded in a deployed system and a locked benchmark —
and it also sets the agenda for what comes next, which the closing section lays out.

---

## 1. Multi-synonym text queries fragment detections

Adding synonyms to the text query of a **late-fusion** open-vocabulary detector fragmented its
detections and dropped precision sharply, with a corresponding F1 collapse. The same change
applied to a **cross-attention** detector produced the opposite effect — that architecture
required three or more concepts to perform well.

The analysis traced the behavior through a per-patch causal chain, which is what elevates it
from a prompt-tuning observation to a **diagnostic for an architectural distinction**: late
fusion scores each patch against each query independently and then competes them, while
cross-attention fuses the concepts before scoring.

The practical consequence is that "add synonyms to improve recall" — standard advice — is
architecture-dependent and can be actively harmful. The deployed configuration was changed as a
result.

---

## 2. Sub-patch invisibility is a formula, not a tuning problem

Below a certain target size, a patch-based detector cannot see the target at all — and that
threshold is derivable from the model's **patch stride divided by the input-to-frame scaling
ratio**, not discovered by experiment.

This matters because it reframes a whole class of complaint. "The model misses small fires" is
an accuracy problem to be trained away; "targets below N pixels are structurally invisible at
this resolution" is an **operating constraint** that determines altitude, zoom limits, and
tiling strategy. The first invites months of futile fine-tuning; the second is engineering
input.

The rule was measured empirically before it was expressed as a formula, then used to set real
operating limits (see [04](./04-realtime-control.md)).

---

## 3. A precision-collapse signature shared across paradigms

Four architecturally unrelated models — a semantic segmentation network, a tiled supervised
detector, a small zero-shot detector, and an anchor-based detector — landed in the **same
narrow F1 band**, each emitting roughly five spurious boxes per frame, with **zero frames
clean of false positives**.

Four different inductive biases, four different training regimes, one identical failure mode.
That is not four coincidences; it suggests a property of the task (amorphous, low-contrast,
scale-ambiguous targets) rather than of any architecture. Reported as an empirical regularity,
it is more informative than any individual model's score.

---

## 4. Image-guided querying collapses, and the mechanism is identifiable

Image-guided detection — supplying an example image instead of a text prompt — failed
catastrophically rather than merely underperforming. The mechanism: **image queries live in the
same patch space as the features they are matched against**, so the similarity saturates near
its ceiling across effectively the entire frame (over a thousand patches), leaving nothing to
discriminate.

This explanation does not appear in the literature covering these models, and it converts an
apparent implementation failure into a predictable consequence of where the query embedding
lives.

---

## 5. Recall-only grading lets published results overstate

A published wildfire segmentation method was re-implemented and re-evaluated under strict
grading on domain data. Its best reported variant scored **far below its published figures**,
with no frame free of false positives.

The gap traces to the original evaluation's **recall-only IoU grading** — precisely what the
strict frame-level metric in [01](./01-model-selection.md) exists to rule out. Finding that
error in a peer-reviewed venue is the strongest available argument that evaluation design
deserves the scrutiny usually reserved for model design.

*The specific paper is deliberately not named here. The finding is about the grading
methodology, which is a systemic issue, not about a particular research group.*

---

## What it would take to publish this

These findings were assessed for publishability by three independent analyses. They disagreed
on which was strongest, and agreed on what the work needs next — which makes the path concrete
rather than speculative.

**The gap is coverage, not validity.** Every finding is grounded in a deployed system and
reproducible against a locked benchmark. What they lack is breadth: most rest on one or two
detectors, all of them on a single amorphous class, and the architectural claim is
well-evidenced but not yet formalized as an argument.

**Scoped at roughly one month of work:**

| Step | What it closes |
| :--- | :--- |
| Replicate fragmentation across 4+ open-vocabulary detectors | Moves a two-model observation to a mechanism claim |
| Extend to 3+ amorphous classes — fog, oil spill, cloud | Makes "amorphous class" a category rather than a synonym for smoke |
| Formalize the invisibility rule with multi-detector validation | Turns a working formula into a stated result |
| Express late-fusion vs cross-attention as a formal argument | Supplies the theory the empirical work already implies |

The reviewer objections are equally identified in advance — *"this is just prompt engineering"*
(answered by the per-patch causal analysis, if presented as mechanism rather than tuning) and
*"why fire and smoke"* (answered only by the multi-domain extension above). Target venue for
the extended version: an applied or workshop track at a major vision conference.

Stating the gap this precisely is the point. Findings that came out of shipping a product
rather than running a study start with an advantage — they are grounded in a system that had to
work — and a specific deficit, which is breadth. Knowing exactly which experiments convert one
into the other is what makes this a plan rather than a hope.

---

[← Back to overview](../README.md)
