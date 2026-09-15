[← Back to overview](../README.md) · **Language:** English | [한국어](./ko/03-edge-optimization.md)

# 03 — Edge Optimization

---

## Where the latency actually was

- Before touching any optimization, the inference pipeline was instrumented stage by stage. The
  TensorRT engine segment in particular was re-measured independently in C++ with no Python
  runtime at all, separating the engine's own cost from the overhead the Python path adds
  around it.
- The assumption that the model engine's latency was the dominant cost turned out to be wrong.

### Outside the model engine

The single largest win was **moving preprocessing from CPU to GPU** — with no accuracy cost, as
expected. The same treatment was then applied to postprocessing. That alone took about 20% off
total latency.

### The model engine

Adding FP16 runtime compilation brought end-to-end latency down by **37%** in total, with no
measurable accuracy loss.

---

## The INT8 quantization investigation

INT8 post-training quantization was pushed as far as it would go, since the edge device in use
was reported to be very efficient at quantized inference.

**Symptom.** With naive INT8, accuracy dropped to near zero — not degraded output but no usable
detection at all.

**Root cause.** The *massive activation* phenomenon in vision transformers: per-channel outliers
in the residual stream grow monotonically with depth.

Measuring across the network, the per-tensor quantization scale grew by **about an order of
magnitude** between early and late layers. Per-tensor INT8 cannot represent that distribution:
accommodating the outliers leaves the normal channels with a handful of effective bits.

**Two things were tried:**

1. **Selective FP16 fallback** on the residual add nodes.
2. **SmoothQuant** on the normalization→linear path, shifting the scale of activation outliers
   onto the weights.

**Result.** Together they recovered to *near* the baseline. Even so, the best INT8 configuration
stayed below the FP16 baseline on F1, and what it bought in return was tens of milliseconds of
latency.

The goal was not minimum latency for its own sake, nor a 10–20% reduction. Taking an accuracy
loss to gain under 5% latency was not a trade worth making. As shown above, quantization could
address about 30% of total latency, and this is what it returned within that.

---

## Rejecting zero-copy

Working at the low-level hardware view, the places that create latency were measured directly in
C. That bounded what memory-bandwidth work such as zero-copy could achieve at a few
milliseconds, so it was not adopted.

---

## Cascade inference

A heavy ViT-family detector cannot run on every frame on an edge device. This
detector was chosen for faint-smoke accuracy with speed explicitly deprioritized
([01](./01-model-selection.md)), so holding accuracy meant giving something up elsewhere. The
answer was a **cascade**.

> A **low-frequency detector ("watcher")** paired with a **lightweight CPU tracker** that
> carries detections between invocations.

The vision goal in this project is not to draw an exact box but to show the area where smoke is
coming from, so as long as the watcher is good, a slightly weaker tracker is not a problem.

- Problem found: when several detection boxes land in one frame, CPU tracker latency grows
  roughly in proportion to the box count. **Class-agnostic non-maximum suppression** fixed the
  case where one smoke plume produced several detection boxes.

---

## Prompt-delta

Freeze the vision backbone and train only two text embeddings (512-D each) — 1,024 parameters
total, shipped as a package of a few KB.

- The training data was detection output collected from deployed aircraft and verified in the
  field, false positives included.
- The resulting version showed a lower false-positive rate on the benchmark and was deployed.

---

## SAHI tiling

Splitting a frame into tiles and running the detector on each tile.

- More tiles did not reliably mean more accuracy, so it was not adopted.

---

## Knowledge distillation

- Conventional knowledge distillation uses the full per-class softmax. Here the only thing
  available to teach with was a logit for whether smoke is present — binary logit-KD.
- That means learning a single scalar, which is prone to overfitting, and in practice it
  performed about the same as plain supervised learning on the box labels. Under these
  conditions there was nothing for distillation to add.

---

**Next:** [04 — Real-Time Perception & Control](./04-realtime-control.md)
