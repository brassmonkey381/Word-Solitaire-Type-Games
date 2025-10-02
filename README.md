# Wordscapes Solitaire – Research Notebooks

This repo contains two experimental Jupyter notebooks exploring **auto-difficulty control** and **board mechanics** for a word-puzzle game inspired by *Wordscapes* and *Solitaire*.

---
All v1,2,3 notebooks implements a **playable game prototype** with:
- A **multi-layer depth board** (tiles block lower tiles until cleared).
- A **deck system** for drawing extra letters
- A bot programmed to always pick the longest word makable using the board + last drawn deck card(s)
- A Player stats tracking panel to display average word length and score, well as how many new words become available as a new card tile is flipped / revealed (we call this metric Δ)
- Some plots of epsilon, the dynamic difficulty tuning parameter, and it's relationship to the metric Δ.

## Notebook v1: Epsilon as a stand alone Difficulty Driver

This notebook demonstrates a **theoretical model** where difficulty is driven by a smoothly oscillating parameter, epsilon, and how it's value affects how many new words become available on card tile flips.

- **Core idea:** difficulty is a sinusoidal signal over time.
- **Epsilon (ε)** is clamped to `[0, 0.5]` and changes only when **new board tiles are revealed**.
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

This leaves room for epsilon to be updated by other in-game events and logic.  
Some rough ideas for events that would trigger an epsilon adjustment.  We can also add decay parameters, and other functions that alter epsilon.
- Player makes an in app purchase, lower their epsilon by 10% for 24 hours (to subtly reward in-app purchase)
- Player uses several power-ups to beat a level, lower their epsilon by 10% for 5 games (to prevent churn from frustration)
- Player performance is very high or low, or player goes on a win streak or loss streak (See implementation of this idea in notebook 2!)
### Visuals

<img width="1036" height="927" alt="epsilon_bot_example" src="https://github.com/user-attachments/assets/069ca88b-7a7a-464e-95b3-54a7fa122da1" />

---

## Notebook v2: Depth Board + Deck + Adaptive Epsilon

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

<img width="965" height="833" alt="epsilon_easy_medium_hard_bot_example" src="https://github.com/user-attachments/assets/6b5c0f88-bc0b-4a8e-ac70-937097d9389a" />

---

## Demo Videos

https://github.com/user-attachments/assets/17f6c862-d18f-4142-8fb9-66441569a8db

## Notebook 3
### Logic for puzzle creation:
Each tile has a connectivity to other tiles on the board.  For example, those in the first row are 100% connected, as they are guaranteed to be revealed at the same time and can be played together.  Tiles in the second row are not guaranteed to all be revealed at the same time, but are still more likely to be available together compared to tiles in the 4th row for instance.  We create a connectivity coefficient (pointwise mutual information (PMI)) for each tile and the other tiles on the board.  Then we randomly select tiles, row by row, and populate a letter that generates the desired possibility of valid words being created.  When difficulty is high, we want to populate less word possibilities between highly connected letters, and the inverse when difficulty is low.  This process should lead to puzzles on the difficulty range (0,1).

## Letter Reveal Delta Demo
### Logic for simulation and stats
**Candidate** letter is an extra DECK tile (not forced into prefix).
**Score** = Δ_used, where Δ_used = (AFTER bank-used) - (BEFORE bank-used).
**AFTER bank-used** does NOT include the deck letter usage.
**Preview** shows BEFORE (green bank) and AFTER (green bank, red deck if actually needed).
**Difficulty** is used to select which letter in the sorted list to serve as the next leter reveal.  0 would select the first, best letter, .5 would select the 13th, etc.


### Visuals
<img width="1296" height="583" alt="next_letter_selection_demo" src="https://github.com/user-attachments/assets/02f9b96e-1112-499e-9460-8e3714e859ff" />

## Demo Videos
https://github.com/user-attachments/assets/5f353d04-3d43-4b78-b648-51e0ef23fd51


---

## Summary

- **Notebook 1** shows a **controlled sine-wave model** for difficulty flow.  
- **Notebook 2** shows **adaptive, performance-driven ε** that maps directly to Easy/Normal/Hard modes.
- **Notebook 3** show how we can premake puzzles on a difficulty scale of 0->1, where easier puzzles favor more frequent bigrams in our valid word set within adjacent layers, to allow easier valid word formation.
- **Next Letter Selection Demo** shows how we can quickly exhaustively simulate how many new board tiles we could select to form a longer word, if we add one any of the 26 possible letters to our options, by drawing it from the deck.
