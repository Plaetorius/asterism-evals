## Accuracy (strict consensus of both judges), every arm on every set

| Set | n | A Model alone | B + corpus | C + web search | D + corpus + web |
|---|---|---|---|---|---|
| P | 60 | 16 (27 %, 17–39) | 28 (47 %, 35–59) | 31 (52 %, 39–64) | 37 (62 %, 49–73) |
| P-uncontested | 50 | 14 (28 %, 17–42) | 24 (48 %, 35–61) | 26 (52 %, 39–65) | 35 (70 %, 56–81) |
| S1-A | 19 | 15 (79 %, 57–91) | 15 (79 %, 57–91) | 16 (84 %, 62–94) | 19 (100 %, 83–100) |
| S1-B | 25 | 3 (12 %, 4–30) | 23 (92 %, 75–98) | 15 (60 %, 41–77) | 23 (92 %, 75–98) |
| S1-OOC | 6 | 4 (67 %, 30–90) | 4 (67 %, 30–90) | 4 (67 %, 30–90) | 4 (67 %, 30–90) |
| ALL-S1 | 50 | 22 (44 %, 31–58) | 42 (84 %, 71–92) | 35 (70 %, 56–81) | 46 (92 %, 81–97) |
| S2 | 30 | 0 (0 %, 0–11) | 30 (100 %, 89–100) | 7 (23 %, 12–41) | 30 (100 %, 89–100) |

## Sensitivity and descriptive counts (set P)

| Arm | lenient (partial = correct) | Gemini judge alone | Claude judge alone | abstained (both) | contradicts key (any judge) | fabricated-citation answers | truncated | no answer | leak flags |
|---|---|---|---|---|---|---|---|---|---|
| A | 43 | 19 | 20 | 4 | 12 | 4 | 3 | 0 | 0 |
| B | 47 | 28 | 34 | 4 | 12 | 1 | 15 | 0 | 3 |
| C | 51 | 34 | 35 | 1 | 14 | 0 | 15 | 0 | 3 |
| D | 58 | 39 | 42 | 2 | 5 | 0 | 18 | 0 | 1 |

## Paired tests (exact McNemar; bootstrap 95 % CI of the difference in points)

| Test | n | x only | y only | diff (pts) | 95 % CI | p |
|---|---|---|---|---|---|---|
| primary_B_vs_C_on_P | 60 | 9 | 12 | -5.0 | -20.0 to +10.0 | 0.664 |
| D_vs_C_on_P | 60 | 13 | 7 | +10.0 | -5.0 to +25.0 | 0.263 |
| D_vs_C_on_P-uncontested | 50 | 13 | 4 | +18.0 | +2.0 to +34.0 | 0.049 |
| D_vs_C_on_S1-A | 19 | 3 | 0 | +15.8 | +0.0 to +31.6 | 0.25 |
| D_vs_C_on_S1-B | 25 | 8 | 0 | +32.0 | +16.0 to +52.0 | 0.00781 |
| D_vs_C_on_S1-OOC | 6 | 0 | 0 | +0.0 | +0.0 to +0.0 | 1 |
| D_vs_C_on_ALL-S1 | 50 | 11 | 0 | +22.0 | +12.0 to +34.0 | 0.000977 |
| D_vs_C_on_S2 | 30 | 23 | 0 | +76.7 | +60.0 to +90.0 | 2.38e-07 |
| B_vs_A_on_P | 60 | 17 | 5 | +20.0 | +5.0 to +35.0 | 0.0169 |
| B_vs_A_on_P-uncontested | 50 | 15 | 5 | +20.0 | +4.0 to +36.0 | 0.0414 |
| B_vs_A_on_S1-A | 19 | 1 | 1 | +0.0 | -15.8 to +15.8 | 1 |
| B_vs_A_on_S1-B | 25 | 20 | 0 | +80.0 | +64.0 to +96.0 | 1.91e-06 |
| B_vs_A_on_S1-OOC | 6 | 0 | 0 | +0.0 | +0.0 to +0.0 | 1 |
| B_vs_A_on_ALL-S1 | 50 | 21 | 1 | +40.0 | +26.0 to +54.0 | 1.1e-05 |
| B_vs_A_on_S2 | 30 | 30 | 0 | +100.0 | +100.0 to +100.0 | 1.86e-09 |
| C_vs_A_on_P | 60 | 18 | 3 | +25.0 | +11.7 to +38.3 | 0.00149 |
| C_vs_A_on_P-uncontested | 50 | 15 | 3 | +24.0 | +8.0 to +40.0 | 0.00754 |
| C_vs_A_on_S1-A | 19 | 2 | 1 | +5.3 | -10.5 to +21.1 | 1 |
| C_vs_A_on_S1-B | 25 | 12 | 0 | +48.0 | +28.0 to +68.0 | 0.000488 |
| C_vs_A_on_S1-OOC | 6 | 0 | 0 | +0.0 | +0.0 to +0.0 | 1 |
| C_vs_A_on_ALL-S1 | 50 | 14 | 1 | +26.0 | +14.0 to +40.0 | 0.000977 |
| C_vs_A_on_S2 | 30 | 7 | 0 | +23.3 | +10.0 to +40.0 | 0.0156 |
| D_vs_A_on_P | 60 | 26 | 5 | +35.0 | +18.3 to +50.0 | 0.000192 |
| D_vs_A_on_P-uncontested | 50 | 26 | 5 | +42.0 | +24.0 to +60.0 | 0.000192 |
| D_vs_A_on_S1-A | 19 | 4 | 0 | +21.1 | +5.3 to +42.1 | 0.125 |
| D_vs_A_on_S1-B | 25 | 20 | 0 | +80.0 | +64.0 to +96.0 | 1.91e-06 |
| D_vs_A_on_S1-OOC | 6 | 0 | 0 | +0.0 | +0.0 to +0.0 | 1 |
| D_vs_A_on_ALL-S1 | 50 | 24 | 0 | +48.0 | +34.0 to +62.0 | 1.19e-07 |
| D_vs_A_on_S2 | 30 | 30 | 0 | +100.0 | +100.0 to +100.0 | 1.86e-09 |
| D_vs_B_on_P | 60 | 11 | 2 | +15.0 | +3.3 to +26.7 | 0.0225 |
| D_vs_B_on_P-uncontested | 50 | 11 | 0 | +22.0 | +12.0 to +34.0 | 0.000977 |
| D_vs_B_on_S1-A | 19 | 4 | 0 | +21.1 | +5.3 to +42.1 | 0.125 |
| D_vs_B_on_S1-B | 25 | 2 | 2 | +0.0 | -16.0 to +16.0 | 1 |
| D_vs_B_on_S1-OOC | 6 | 0 | 0 | +0.0 | +0.0 to +0.0 | 1 |
| D_vs_B_on_ALL-S1 | 50 | 6 | 2 | +8.0 | -2.0 to +20.0 | 0.289 |
| D_vs_B_on_S2 | 30 | 0 | 0 | +0.0 | +0.0 to +0.0 | 1 |
| B_vs_C_on_P | 60 | 9 | 12 | -5.0 | -20.0 to +10.0 | 0.664 |
| B_vs_C_on_P-uncontested | 50 | 9 | 11 | -4.0 | -22.0 to +14.0 | 0.824 |
| B_vs_C_on_S1-A | 19 | 2 | 3 | -5.3 | -26.3 to +15.8 | 1 |
| B_vs_C_on_S1-B | 25 | 8 | 0 | +32.0 | +16.0 to +52.0 | 0.00781 |
| B_vs_C_on_S1-OOC | 6 | 0 | 0 | +0.0 | +0.0 to +0.0 | 1 |
| B_vs_C_on_ALL-S1 | 50 | 10 | 3 | +14.0 | +0.0 to +28.0 | 0.0923 |
| B_vs_C_on_S2 | 30 | 23 | 0 | +76.7 | +60.0 to +90.0 | 2.38e-07 |

## Citations (all sets)

| Arm | fabricated | miscited | ok | unverifiable_nonscholarly |
|---|---|---|---|---|
| A | 4 | 7 | 195 | 2 |
| B | 1 | 3 | 364 | 0 |
| C | 1 | 7 | 357 | 42 |
| D | 1 | 2 | 368 | 19 |

Judges: Cohen's kappa 0.77 over 560 gradings; 74 disagreements.
Keys: 10 contested in P; ungrounded ['h005']; weak S1-B keys 0.

**Verdict (pre-registered rule):** inconclusive (not evidence of equivalence)
