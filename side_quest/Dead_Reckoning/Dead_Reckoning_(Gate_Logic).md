# 16-bit Fixed-Point 2D Dead Reckoning in Gate Logic

A small digital-design project that implements 2D dead reckoning directly in logic using [Digital](https://github.com/hneemann/Digital).

No CPU, floating-point math, or software loop — just registers, adders, multipliers, and ROMs.

The circuit takes **speed (`v`)** and **turn rate (`ω`)** as inputs and continuously updates:

* Heading `θ`
* Position `X`
* Position `Z`

---

## How It Works

Each clock tick represents:

<p align="center">
  <b>Δt = 1/256 = 2<sup>−8</sup> s</b>
</p>

At every tick:

<p align="center">
  <b>θ<sub>1</sub> = θ<sub>0</sub> + ω<sub>0</sub>Δt</b>
</p>

<p align="center">
  <b>X<sub>1</sub> = X<sub>0</sub> + v<sub>0</sub>cos(θ<sub>0</sub>)Δt</b>
</p>

<p align="center">
  <b>Z<sub>1</sub> = Z<sub>0</sub> + v<sub>0</sub>sin(θ<sub>0</sub>)Δt</b>
</p>

Because <b>Δt = 2<sup>−8</sup></b>, multiplying by the time step is just a **right shift by 8 bits**. No extra multiplier is required.

---

## Number Formats

| Signal        |  Width | Format                         |
| ------------- | -----: | ------------------------------ |
| `ω`           | 16-bit | Signed Q16                     |
| `θ`           | 24-bit | Binary Angle Measurement (BAM) |
| `v`           | 16-bit | Signed Q8.8 m/s                |
| `sin/cos`     | 16-bit | Signed Q1.14                   |
| `v × sin/cos` | 32-bit | Fixed-point product            |
| `X, Z`        | 40-bit | Position accumulators          |

### Binary Angle

The heading uses a **Binary Angle Measurement (BAM)** instead of radians.

One complete revolution is:

<p align="center">
  <b>1 revolution = 2<sup>24</sup></b>
</p>

This means the heading naturally wraps around when the 24-bit accumulator overflows.

The upper 8 bits are used directly as the address for the sine and cosine lookup tables.

---

## Architecture

The design is split into small blocks that can be tested independently.

```text
OMEGA
  │
  ▼
[ HEADING ] ─────────► THETA
      │
      ▼
  8-bit ROM ADDRESS
      │
      ▼
 [ SIN / COS ROM ]
      │
      │
V ────┴────► [ POSITION ]
                  │
              ┌───┴───┐
              ▼       ▼
              X       Z
```

### Project Files

| File           | Function                                       |
| -------------- | ---------------------------------------------- |
| `accumulator.dig`      | 24-bit accumulator with synchronous reset      |
| `accumulator_40.dig`    | 40-bit position accumulator                    |
| `head.dig`  | Heading calculation and ROM address generation |
| `trig.dig`     | Sine and cosine lookup tables                  |
| `position.dig` | Velocity projection and position updates       |
| `dead_reckoning.dig`      | Top-level circuit connecting everything        |

The ROM contents are stored in:

```text
sin_rom.hex
cos_rom.hex
```

---

## Running the Simulation

1. Install **Digital**.
2. Keep all `.dig` and `.hex` files in the same directory.
3. Open:

```text
dead_reckoning.dig
```

4. Set:

```text
V      = 0x4000
OMEGA  = 0x4000
RESET  = 0
```

5. Start the clock.

The system should produce a roughly circular trajectory.

Changing:

```text
OMEGA = 0xC000
```

reverses the direction of the turn.

The `THETA`, `X`, and `Z` signals can be viewed using Digital's measurement graph.

---

## What to Expect

With constant speed and turn rate, the system behaves approximately like a vehicle moving around a circle.

The ideal turning radius is:

<p align="center">
  <b>R = v/ω</b>
</p>

The simulated path will not be perfectly circular because of the fixed-point representation and lookup-table resolution.

---

## Limitations

This implementation intentionally keeps the hardware simple.

### Angle Quantisation

The sine and cosine ROMs contain only **256 entries**.

Therefore, the heading resolution is approximately:

<p align="center">
  <b>360° / 256 ≈ 1.41°</b>
</p>

This produces some visible faceting in the trajectory.

### Fixed-Point Error

The multiplication results are truncated rather than rounded.

Small errors therefore accumulate during integration.

### Simplified Motion Model

The model assumes:

* Forward motion only
* No lateral velocity
* Constant input speed unless changed externally
* No sensor noise
* No acceleration model

**This is a simple digital dead-reckoning model, not a complete navigation system.**

---

## Possible Next Steps

The next version could add:

* Acceleration → speed integration
* A ROM containing a recorded IMU trace
* A bit-exact Python reference model
* Automated comparison between the Python model and gate-level simulation
* Position-drift measurements over time
* Higher-resolution sine/cosine tables
* Different fixed-point formats

---

## Goal

The main goal is straightforward:

> **Take the dead-reckoning equations and build them as actual digital hardware.**

No CPU. No floating point. Just fixed-point arithmetic, memory, registers, and logic.
