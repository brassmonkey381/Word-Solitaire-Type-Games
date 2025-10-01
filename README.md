# Wordscapes Solitaire – Research Notebooks

This repo contains two experimental Jupyter notebooks exploring **auto-difficulty control** and **board mechanics** for a word-puzzle game inspired by *Wordscapes* and *Solitaire*.

---

## Notebook 1: Epsilon as a Sine-Wave Difficulty Driver

This notebook demonstrates a **theoretical model** where difficulty is driven by a smooth, periodic function of the number of **tile reveals**.

- **Core idea:** difficulty is a sinusoidal signal over time.
- **Epsilon (ε)** is clamped to `[0, 0.5]` and changes only when **new board tiles are revealed**.
- **Deck flips** consume waste cards but do not advance ε.
- Plots show:
  - ε over time (with win/loss markers).
  - Δ (net options added) over time, smoothed with rolling averages.

### Equations

Target delta for a reveal is:

$$
\text{target} = (1-\varepsilon)\cdot \text{max}_{\text{delta}} + \varepsilon \cdot \text{min}_{\text{delta}}
$$

Epsilon evolves as a clamped sine wave (period = 30 reveals):

$$
\varepsilon(t) = \text{clamp}\!\left(
0.5 \sin\!\left(\tfrac{2\pi t}{30}\right)\,0\,0.5
\right)
$$

### Visuals

👉 *Drag and drop screenshots of the epsilon chart and delta trends here.*
<img width="1036" height="927" alt="epsilon_bot_example" src="https://github.com/user-attachments/assets/069ca88b-7a7a-464e-95b3-54a7fa122da1" />

---

## Notebook 2: Depth Board + Deck + Adaptive Epsilon

This notebook implements a **playable game prototype** with:

- A **multi-layer depth board** (tiles block lower tiles until cleared).
- A **deck/waste system** for drawing extra letters.
- **Epsilon (ε)** as an **adaptive difficulty signal** in `[0,1]`.
- Automatic switching between **Easy / Normal / Hard modes** based on ε with **hysteresis**.

### Difficulty Modes

- **Easy:** reveal/waste pickers try to **maximize new words** and favor longer completions.
- **Normal:** letters drawn **randomly** (weighted).
- **Hard:** mostly random, but occasionally **adversarial**:
  - Picks letters that **minimize options** or reduce longest possible completions.
  - ε is “sticky” in Hard (floored at 0.70, slower to decrease).

### How ε Updates

**After each word:**

$$
\text{target} = \text{clamp01}\!\left(\frac{\text{avg}_{\text{recent-len}} - 3}{4}\right)
$$

Exponential moving average update:

$$
\varepsilon_{\text{new}} = (1-\alpha)\,\varepsilon_{\text{old}} + \alpha \cdot \text{proposed}
$$

**After each game:**
- Targets based on **game score** and **win/loss streaks**.
- Hard resets:
  - 4+ win streak → ε = 1.0 (force Hard).
  - 4+ loss streak → ε = 0.0 (force Easy).

### Visuals

👉 *Drag and drop screenshots of the depth board, waste system, and difficulty chart here.*
<img width="965" height="833" alt="epsilon_easy_medium_hard_bot_example" src="https://github.com/user-attachments/assets/6b5c0f88-bc0b-4a8e-ac70-937097d9389a" />

---

## Demo Videos

👉 *Drag and drop gameplay clips or walkthrough videos here.*


https://github.com/user-attachments/assets/17f6c862-d18f-4142-8fb9-66441569a8db


---

## Summary

- **Notebook 1** shows a **controlled sine-wave model** for difficulty flow.  
- **Notebook 2** builds a **full game prototype** with **adaptive, performance-driven ε** that maps directly to Easy/Normal/Hard modes.  
