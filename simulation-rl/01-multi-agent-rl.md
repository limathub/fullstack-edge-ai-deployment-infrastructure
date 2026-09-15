[← Back to overview](../README.md) · **Language:** English | [한국어](./ko/01-multi-agent-rl.md)

# 01 — Multi-Agent RL for Formation Control

> Training a swarm to hold formation with PPO — and discovering that the algorithm was the
> easy part, the reward coefficients were the hard part, and the infrastructure to run it at
> all was most of the work.

---

## Why reinforcement learning for this

Formation control is not hard to write down as a controller. It becomes hard when three
objectives are live at once: hold the formation shape, do not collide, and still make progress
along the waypoint path. Each pair trades against the other — a tighter formation is a smaller
collision margin; an aggressive path is a looser formation — and the trade is state-dependent,
which is exactly the situation where hand-tuned gains stop generalizing across scenarios.

That is the case for a learned policy: the trade-off is expressed once in a reward function
and searched, rather than re-tuned per scenario. Whether it is the *right* answer for a
deployed swarm is a separate question — this was an investigation, not a product.

Two tasks were trained: **circular formation** and **V-shaped waypoint following**. Both
produced real trained checkpoints; neither was a configuration-only experiment.

---

## Two simulators, two RL stacks

The work ran across a deliberate split, because the two halves want opposite things.

| | Fast iteration | Deployment-grade scenes |
| :--- | :--- | :--- |
| Simulator | Lightweight physics engine (PyBullet) | Game engine (Unreal Engine) |
| Why | Thousands of cheap episodes, no rendering cost | Realistic scenes, custom CNN policies on visual input |
| RL stack | Single-machine baselines library (Stable-Baselines3) | Distributed RL framework (Ray RLlib) |

The lightweight environment is where reward shaping was actually debugged — a reward bug is
cheap to find at a thousand episodes an hour and expensive to find at ten. The game-engine
environment is where policies met visual observations and scene complexity, with custom CNN
policies rather than the default MLP heads.

Keeping both meant the same task definition had to survive two very different observation and
step interfaces, which is its own discipline: anything scenario-specific that leaked into the
policy would show up as a behavior that worked in one simulator and not the other.

---

## Implemented from the paper, not from a checkpoint

The methods came from recent papers, and they were **implemented from scratch, then trained
and evaluated against that implementation** — not by downloading a released checkpoint and
reporting its numbers.

The distinction matters more than it sounds. Running an author's checkpoint tests the author's
engineering; it tells you almost nothing about whether you understand the method. Re-deriving
it is the cheapest available way to find the parts of a paper that are load-bearing and the
parts that are decoration — and the parts that are quietly missing from the write-up, which is
usually where the reproduction stalls.

The same instinct produced the paper reproduction in [02](./02-paper-reproduction.md).

---

## Making it run: headless, parallel, in the cloud

Training a game-engine environment on a laptop is not a strategy. The training moved onto
**cloud instances, rendered headless, running rollouts in parallel** — and getting there was a
substantial fraction of the total effort, none of it reinforcement learning:

- A game engine that expects a display has to be persuaded to run without one, which is where
  most of the early failures lived.
- Rollout workers have to be distributed across GPU instances without the environment becoming
  the bottleneck — a simulator-bound trainer wastes exactly the resource it was scaled onto.
- Runs have to survive being interrupted, because the cheap instances are the interruptible
  ones.

This is where the distributed-GPU and headless-rendering experience came from, as much as from
the RL itself. It is also the part that transfers most directly: the wildfire program's
simulation work ([07 — Simulation & Verification](../wildfire-uav/07-simulation-verification.md))
is the same problem class — run the real software, without the real hardware, unattended.

---

## The hard part was the coefficients

The algorithm was rarely what blocked progress. **Reward coefficient tuning was its own
problem**, and most of the wall-clock went there.

The weights that trade formation tightness against collision penalty against path progress are
not derivable from first principles — they are found by search, and the search is expensive
because each sample is a full training run. Worse, the failure signature is ambiguous: a swarm
that drifts out of formation looks the same whether the formation term is underweighted, the
collision term is overweighted, or the policy simply has not converged yet.

What made it tractable was treating the coefficients as the experiment rather than as setup —
changing one at a time, keeping the runs comparable, and accepting that a large share of
compute buys information about the reward function rather than a better policy. That is an
uncomfortable thing to budget for and it is the honest shape of the problem.

---

## What this would need to be a stronger claim

Stated plainly, because the gap is real:

| Missing | Why it matters |
| :--- | :--- |
| Quantitative results — episode returns, formation error, collision rate | Right now the claim is "it trained and held formation", not "it performed this well" |
| Scale figures — environment steps, wall-clock, instance count and type | The distributed-training claim is qualitative without them |
| A baseline comparison against a classical formation controller | Without it, "RL was the right tool" is asserted rather than shown |
| The reward function as finally settled | The coefficient story is the most interesting part and currently has no artifact |

These are recoverable from the training logs and checkpoints if they are still in hand — which
is the first thing to check before this material goes anywhere public.

---

**Next:** [02 — Reproducing a Paper From Scratch](./02-paper-reproduction.md)
