# Mixed-Signal Timing Studio

A single-file, offline web tool for drawing timing diagrams that mix **analog waveforms** (supplies, LDO outputs, bias, battery) with the **digital signals they produce** (POR, resets, enables, clocks, mode/state buses).

Open `index.html` in any browser. No install or build step. Type on the left and the diagram on the right updates as you type.

## Why text-driven?
Write the analog behaviour once. Digital signals are *computed* from it, so timing stays consistent when you change something:

```
analog  VDD    = 0:0 0.4:0 2.6:3.3@rc 9.8:0@s ; range=0:3.6 unit=V
level   VDD 2.4 "V_{POR,R}"
digital por_ok = thr(VDD, 2.4, 2.1)                      # comparator with hysteresis
digital por_n  = valid(delay(por_ok, 0.35, 0), VDD > 0.9) # X while logic is unpowered
analog  VCORE  = min(follow(por_n, 0, 1.2, 0.45), VDD)    # LDO ramps when enabled
digital rst_n  = delay(por_n & (VCORE > 1.08), 0.5, 0)
digital clk    = clock(0.08, rst_n)
bus     state  = pick(!por_n, "POR", !rst_n, "RESET", "ACTIVE")
span    VDD.up(2.4) por_n.rise "t_{POR} = {dt}"
```

Moving a VDD point moves every derived edge, span measurement and state with it.

## Features
- Analog: point lists with linear, RC, S-curve or step segments; `follow()` (regulator/bias ramps driven by a digital enable); `min/max/clamp`, arithmetic, `ripple()`, `noise()`.
- Digital: comparators (`VDD > 1.8`, `thr()` with hysteresis), `& | ^ !`, `delay()` (separate rise and fall times), `deglitch()`, `sr()` latch, `clock()` gated by an enable, `valid()` for X-states, plus 0/1/X/Z edge lists.
- Buses and states: lists, `sel()`, `pick()` (priority encoder).
- Annotations: threshold `level` lines, `marker`s, shaded `region`s, `span` dimension arrows with auto-measured `{dt}`, cause-to-effect `arrow`s. Events can be `sig.rise[2]`, `VDD.up(1.8)`, `mode.is(RUN)`, with `+offset`.
- Labels: `V_{DD}` subscript, `~{RST}` overline.
- Hover readout of every signal's value at the cursor.
- Export SVG/PNG, copy SVG, save/open the source text. Paper (light) or scope (dark) diagram colors.

The full syntax is in the **Syntax reference** tab inside the tool.
