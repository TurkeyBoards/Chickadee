Below is a **single Markdown document** you can drop into your public repo (e.g., `README.md` or `PROJECTS.md`). It’s written so you can keep **all three projects in one place** and link out to per-project folders later.

---

# Chickadee Morse Keys

This repo contains three small, TRS-connected Morse key projects designed for ham radio use:

* **Chickadee Touch** — capacitive-touch iambic paddle (DIT/DAH)
* **Chickadee Tap** — single-button straight key (tap)
* **Chickadee Double Tap** — two-button / tap-driven key variant (double-tap workflow)

All projects target maximum interoperability with common transceivers and keyers using a **3.5mm TRS paddle/key input**.

> **TRS convention (default):**
> **Tip = DIT**, **Ring = DAH**, **Sleeve = GND**

---

## Quick comparison

| Project        | Input style                              | Intended use                      | Power                                            | TRS behavior                   |
| -------------- | ---------------------------------------- | --------------------------------- | ------------------------------------------------ | ------------------------------ |
| **Touch**      | Capacitive touch pads (iambic)           | Paddle keying (DIT/DAH)           | CR2032 + optional radio-harvest (when supported) | Tip=DIT, Ring=DAH              |
| **Tap**        | Single momentary button                  | Straight keying                   | None (passive switch)                            | Tip keyed to GND (Ring unused) |
| **Double Tap** | iambic | Trainer-friendly / compact keying | Depends on implementation                        | Typically Tip/Ring or Tip-only |

---

# Chickadee Touch

## What it is

A **capacitive-touch iambic paddle** implemented with two single-key capacitive touch ICs and two MOSFET outputs that pull the radio key lines to ground (like a conventional paddle).

## Key features

* **Two touch pads:** DIT + DAH
* **3.5mm TRS output:** Tip=DIT, Ring=DAH, Sleeve=GND
* **CR2032 coin cell powered**
* **Optional “radio-powered” behavior** when the connected device provides pull-ups on the key lines (harvest path)
* **ESD protection** on TRS lines and a **VDD clamp** for robustness

## How it works (high level)

* Each pad is sensed by a capacitive touch controller.
* When touched, the controller drives a MOSFET gate.
* The MOSFET pulls **TIP** (DIT) or **RING** (DAH) to **GND**.

## Power modes

* **ON (switch enabled):** coin cell supplies a stable VDD rail.
* **OFF (switch disabled):** battery is disconnected; however, if plugged into a radio/keyer with pull-ups, the circuit can be weakly back-powered through the protection/harvest network unless the design includes full “off isolation.”

### Known v1 behavior (important)

Some radios/keyers provide pull-ups on TIP/RING. With the current Touch architecture, when the unit is **OFF but plugged in**, the key lines can feed a small amount of current into the internal rail via the line-to-VDD paths. That can result in odd keyer behavior on certain rigs (example: alternating dit/dah) until the unit is turned ON or unplugged.

**Mitigations**

* Operational: leave the unit **ON when plugged in**, or unplug when OFF.
* Hardware (v2 idea): add true “off isolation” for TIP/RING (or gate the harvest path) so OFF is electrically dead.

## Recommended parts (summary)

* Capacitive controller: **AT42QT1011** (x2)
* Output switch: **2N7002** (x2) or similar small NMOS
* Pad network: 22k series to SNSK + ~2.2nF between SNS/SNSK (tunable)
* Decoupling: 0.1µF at each IC
* Bulk reservoir: **47–100µF** on VDD (package typically 1206/1210)
* Protection: ESD steering on TIP/RING + VDD clamp to GND

## Layout notes (why it works reliably)

* Keep **SNS/SNSK routing short** and away from TIP/RING and MOSFET drains.
* Avoid **GND plane under the touch pads** (use keepouts).
* Place **ESD + VDD clamp + bulk cap** near the TRS connector entry.
* Remove **paste** from touch pads and battery contact pads (copper + mask opening only).

## Troubleshooting

* **False triggering / too sensitive:** increase the SNS/SNSK capacitor value slightly (example: 3.3nF).
* **Not sensitive enough:** reduce the SNS/SNSK capacitor value (example: 1.0–1.5nF), increase pad area, or expand ground keepout.
* **Weird behavior when OFF but plugged in:** use the mitigations above; consider v2 off-isolation.

---

# Chickadee Tap

## What it is

A **single-button straight key** designed for maximum simplicity and compatibility.

## Recommended wiring to TRS

For broad compatibility with rigs that treat TRS as a paddle input:

* **Sleeve (S) → GND**
* **Tip (T) → Key line**
* **Ring (R) → Not connected** (optional jumper to Tip if you want “try both” behavior)

Button wiring:

* One side of the button → **Tip**
* Other side of the button → **Sleeve (GND)**

### “Most universal” option (recommended)

Add a **3-pad solder jumper** to select whether the button keys **Tip** or **Ring**. Default to **Tip**.

## Power

None required. This is a passive closure to ground.

## Notes

* Use a **ground plane freely** in this design; it improves return path and EMI immunity.
* Keep TRS routing short; optionally add ESD near the connector.

---

# Chickadee Double Tap

## What it is

A “double-tap” variant intended to provide a compact or trainer-friendly workflow. (There are multiple ways to implement this; the repo can support one or more.)

## Common implementation directions

Choose one direction and document it clearly:

### Option A: Two-button straight key (recommended simplest)

* Button A = DIT (Tip)
* Button B = DAH (Ring)
* Passive switch-only design; no battery required.

### Option B: One-button timing logic (advanced)

* Single button; firmware interprets short vs long press (dit vs dah) or double-tap patterns.
* Requires a microcontroller and power.

## Recommendation

If you want “simple and universal” like Chickadee Tap:

* Prefer **Option A** (two mechanical buttons → Tip/Ring to GND)

---

# Repo structure suggestion

```
/Chickadee-Touch/
  README.md
  hardware/
    Chickadee-Touch.kicad_sch
    Chickadee-Touch.kicad_pcb
    gerbers/
  docs/

/Chickadee-Tap/
  README.md
  hardware/
  docs/

/Chickadee-DoubleTap/
  README.md
  hardware/
  docs/
```

---

# Compatibility notes (ham radios & keyers)

Most modern rigs and external keyers treat paddle inputs as:

* internally pulled up,
* and expect a switch closure to ground.

**Always verify the radio/keyer jack type**:

* Some radios have separate jacks for “KEY” vs “PADDLE”
* Some menu settings choose “straight key” vs “paddle”

---

# License

Choose a license appropriate for your goals:

* **CERN-OHL-S** or **CERN-OHL-P** (hardware-friendly)
* **MIT** (if you want very permissive)

---

# Contributing

PRs welcome:

* footprint alternatives for hard-to-source TRS jacks
* v2 improvements for Touch “true OFF isolation”
* enclosure / ergonomic pad shape experiments
* tested radio compatibility notes (rig model + settings)

---

## TODO / Roadmap

* **Touch v2:** true OFF isolation (prevent back-powering from TIP/RING when OFF)
* Standardize TRS jack footprint alternatives (in-stock substitutions)
* Add build photos + recommended enclosure ideas
* Add a compatibility matrix (rigs tested + settings)

---

If you want, tell me what you want **Tap** and **Double Tap** to be *exactly* (passive vs MCU, one button vs two, which TRS mapping), and I’ll tighten those sections so there are no “options” or ambiguity in the public README.
