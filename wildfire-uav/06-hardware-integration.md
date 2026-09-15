[← Back to overview](../README.md) · **Language:** English | [한국어](./ko/06-hardware-integration.md)

# 06 — Hardware & Interface Integration

---

## Battery monitoring

**Current measurement never worked from the start.** Rather than accept it, the manufacturer's
wiring diagram was re-derived from scratch, which exposed a wiring fault. Current measurement was
restored.

**A ground loop damaged hardware.** A wiring ground loop through the battery monitor's data line
burned out four USB ports on one edge device along with its adapter. The root cause was that the
battery monitor in use was a non-isolated version. It was solved by putting an isolating adapter
in between before rewiring — a matter of understanding electrical isolation when connecting two
power subsystems with different ground references.

---

## Cameras / communication modems

Cameras were bought from several vendors for testing. Manuals were read, what each camera could
and could not do was analyzed, and usable approaches established. Some features the vendor had
documented did not actually work, so the verified subset was written into an internal runbook and
kept current.

---

## Flight controller (ArduPilot) integration

Integrated a flight controller with GPS attached, and designed parameter tuning so it can be done
through the edge device.

---

## Monitoring and logging

A real-time monitoring stack — time-series database plus dashboards — was designed to collect
station status and flight telemetry at **one-second resolution**.

The server was also designed to monitor itself. The result is a platform with dashboards, alerts,
and collected, searchable logs.

---

## Edge device boot time

Work on reducing the time from connecting the battery to the drone being operational. Boot checks
that every connected hardware component is healthy; the path from starting the vision algorithm
to a rendered image reaching the server was profiled, and the reductions found there applied.

---

**Next:** [07 — Simulation & Verification](./07-simulation-verification.md)
