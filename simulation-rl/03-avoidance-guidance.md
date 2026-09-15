[← Back to overview](../README.md) · **Language:** English | [한국어](./ko/03-avoidance-guidance.md)

# 03 — Avoidance, Guidance and Mission Tooling

> The non-learned half of the same year: a distance transform that turns an obstacle map into
> a steering decision, guidance logic driven by a detector, and the ground-station tooling
> around both.

---

## Obstacle avoidance by distance transform

The avoidance path is four steps: **depth imagery → binary obstacle map → Euclidean distance
transform → directional safety scoring**, selecting the heading with maximum clearance.

The distance transform is the interesting choice. A binary obstacle map answers only "is this
cell blocked", which means evaluating a candidate heading requires walking along it and testing
cells — a search, repeated per heading, per frame. The Euclidean distance transform converts
that same map into a **continuous clearance field in a single pass**, so "how far is the nearest
obstacle from this point" becomes a lookup. Directional safety then reduces to sampling the
field along each candidate heading, and picking a heading becomes an argmax over a small set of
scores instead of a search.

That is a general pattern worth naming: when a per-query search is too slow, the fix is often
to precompute the field the query is really asking about, once.

A related depth-based avoidance implementation ran against a photorealistic simulator (AirSim),
alongside a reinforcement-learning gateway — which is where this line of work met the learned
approach in [01](./01-multi-agent-rl.md). Having both on the same problem is what makes the
comparison honest: the classical method is the baseline the learned one has to beat.

---

## Detection-triggered intercept guidance

Roughly **1,800 lines of guidance logic triggered by object detection** — the detector decides
*that* there is a target and where it is in frame, and the guidance law decides what the
aircraft does about it.

This is the early version of a pattern that reappears later in a different program: a detector
is not an output, it is an input to control, and once a detection can command an actuator the
engineering problem changes shape. Detector jitter becomes actuator motion. Detector latency
becomes control lag. A confidence threshold stops being a metric-tuning knob and becomes a
safety parameter, because the cost of a false positive is no longer a wrong number in a report
— it is the aircraft doing something.

The same pattern, worked out properly with entry gates, deadzones and watchdogs, is the
real-time control chapter of the wildfire program
([04 — Real-Time Perception & Control](../wildfire-uav/04-realtime-control.md)). This
was where it started.

---

## Mission tooling

Two smaller tools, both aimed at the gap between what an operator wants and what a mission file
contains:

- **Waypoint spline smoothing** — ground-station mission waypoint files converted into smooth
  three-dimensional splines for swarm missions. A waypoint list is a sequence of corners; an
  aircraft flying corners wastes energy and time decelerating into each one, and a swarm flying
  corners does it in formation, which amplifies the cost.
- **Speech-driven mission generation** — a ground station that turns Korean speech recognition
  into mission assignment and automatic path planning, so an operator states an intent rather
  than authoring a waypoint file.

Neither is deep. Both are honest examples of the same judgment that recurs throughout: the
friction worth removing is usually the friction between a person and the system, not the
friction inside the algorithm.

---

## The stack this ran on

Across these projects: physics simulation for realistic-background scenarios including
depth-based obstacle avoidance (Gazebo), a game engine for algorithm deployment and
multi-agent RL with custom CNN policies (Unreal Engine), a lightweight physics engine for MARL
(PyBullet), working familiarity with two further robotics simulators (MuJoCo, Isaac), a viewer
for 3D Gaussian Splatting results (SIBR), and ROS 2 throughout.

Also from this period: SLAM, depth algorithms, teleoperation systems, and drone logistics
deployment experience — the last of which, like the wildfire program that followed, involved
customer sites far from the office, outdoor rather than warehouse conditions, and the full
distance between a demo and a system someone depends on.

---

## What this would need to be a stronger claim

| Missing | Why it matters |
| :--- | :--- |
| Avoidance performance — success rate, clearance margins, failure cases | The method is described; its behavior is not |
| What the guidance logic was evaluated against | 1,800 lines is a size, not a result |
| Whether the classical and learned avoidance were ever compared directly | If they were, that comparison is the most valuable thing in this chapter |

---

[← Back to overview](../README.md)
