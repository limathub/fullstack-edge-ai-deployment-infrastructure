[← Back to overview](../README.md) · **Language:** English | [한국어](./ko/05-geometry-sensors.md)

# 05 — Sensors, Signal Processing & Time Sync

---

## Aligning video with flight logs

One reason the automatic label projection in [02](./02-data-pipeline.md) failed is that video
timestamps and attitude telemetry did not line up. A dedicated open-source tool for this was
evaluated, but it returned inconsistent results on the same input, so it was not used.

Built instead.

1. Dense **optical flow** extracts per-frame rotation from the video.
2. The yaw-rate time series is extracted from the flight log.
3. The two are Z-score normalized and **cross-correlated**.

---

## Building a time axis on hardware with too many clocks

One drone system carries several clocks: GPS, the flight controller, the edge device (a
monotonic clock, plus an absolute one when it has a network connection), and the camera where it
has its own chip. These time domains run at once, so depending on which clock is taken as the
reference, data from the same instant would frequently disagree. The time axis was built as
follows.

- The base is `(boot_id, monotonic_time)` — the only pair on the device that is continuous and
  unambiguous.
- The edge device only tags its records. Alignment, absolutization and drift fitting are done
  later, offline, from the logs.
- For GPS, the flight controller, the camera and network time, every point where a value changes
  is logged together with the edge device's wall clock at that moment.

---

## Coverage path planning

Generates offline, over real terrain elevation data, the shortest path that **covers a thin
altitude band around a mountain completely with the camera**.

Read the literature on the Traveling Salesman Problem, implemented it directly, and tried
several approaches. Most produced inefficiently long paths; introducing cell decomposition and
then applying TSP gave the shortest path.

To judge whether the band is actually covered end to end, and how efficient a path is, a program
that takes a terrain map and a path and measures performance was written in-house and used
alongside.

---

**Next:** [06 — Hardware & Interface Integration](./06-hardware-integration.md)
