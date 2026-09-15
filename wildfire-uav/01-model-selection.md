[← Back to overview](../README.md) · **Language:** English | [한국어](./ko/01-model-selection.md)

# 01 — Model Selection & Evaluation Methodology

---

## Starting conditions

- Started with no labeled data, no benchmark, and no firm agreement on what the task even was.
- The first substantive decision was to **narrow the problem**: not "where exactly is the fire"
  but "is there fire in this frame". Localization precision was explicitly deprioritized, and
  accuracy on faint early smoke was put ahead of speed and model size — if a detector misses the
  first ten minutes of a fire, running fast does not help.

---

## The survey

Over 15 model families were compared on a **single leaderboard** under one grader, rather than
by impression.

| Family | Type | Outcome |
| :--- | :--- | :--- |
| zero-shot open-vocabulary detector | text-prompted detection | **Selected** — led on faint, early-stage smoke |
| fine-tuned supervised detector | supervised | Tied with each other at a dataset-imposed ceiling |
| vision-language QA models (3 architectures) | image→text answering | Useful as references; missed the fine smoke cases |
| wildfire-specific segmentation | segmentation | Rejected — watchtower-trained, read roads as smoke, and too slow for the edge budget |
| pretrained open-vocabulary detector (COCO/LVIS lineage) | zero-shot | Rejected — zero detections, no fire/smoke prior in pretraining |

A small zero-shot detector caught faint, early-stage smoke that larger models missed. It held
the lead with no task-specific training and became the production baseline.

---

## The grading criterion, and the benchmark it grades against

- Scored on conventional P/R/F1 (precision, recall, F1) plus a **strict frame-level metric**:
  any frame containing even one false positive counts as wrong. Treating a frame with one false
  positive in it as partially correct does not match what operations need.
- Public data could not supply the benchmark. Footage was shot in-house and labeled on a
  self-hosted annotation platform ([02](./02-data-pipeline.md)).
- Frames were then **classified by context** — viewpoint, range, stage of the fire — and that
  classification was used to weight the scoring set toward the conditions the system actually
  operates in, so a leaderboard number reflects deployed performance rather than an average
  over public data.
- The final held-out benchmark was built from four sources **no candidate model had trained
  on**, with per-source scoring rules fixed.

Two subtler problems surfaced while building it.

**Annotation style skews IoU.** Depending on the labeler, boxes came out loose with generous
margins, or tight around only the certain part. Rather than assume one IoU threshold held
everywhere, the detector was run over footage to settle an appropriate threshold empirically.

**Excluding ground truth below a size cutoff distorts the metric.** Dropping small objects from
the ground truth creates a blind spot: the model can find a small ember correctly and still be
scored a false positive because the label is not there, pushing F1 down. That distortion was
recorded next to the final score rather than hidden, and small detections were judged by eye.

---

## Data context decided performance

- Scaling the training set up lifted validation mAP, and did not move the held-out hard cases.
  Four fine-tuned generations of the same detector family missed the same cases at the same
  rate. The bottleneck was the data, not the architecture.
- The reason lies in what public data is. Surveying five public wildfire datasets for viewpoint
  fitness, the usable ones are watchtower imagery or incidental footage of fires already large,
  because that is what gets photographed. The regime early detection needs — small, faint, seen
  from above at close range — has no public dataset at scale.
- So the decision was to collect in-house, and to make collecting cheap enough to repeat
  (→ [02](./02-data-pipeline.md)).

---

## Prompt engineering

Prompt tuning — the cheapest optimization available at this stage — was pushed in several
directions.

- **Prompt design is architecture-dependent.** No single prompt design worked well across both
  late-fusion and cross-attention architectures; each model structure needed a different
  approach.
- **More complex prompts are not reliably better.** Across a sweep of more than twenty prompt
  configurations, elaborate constructions (for example, supplying both smoke and cloud so the
  model picks only what is closer to smoke) often lost to simpler ones.

---

**Next:** [02 — Data Pipeline & Annotation](./02-data-pipeline.md)
