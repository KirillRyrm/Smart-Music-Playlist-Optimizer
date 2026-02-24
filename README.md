# 🎵 Smart Music Playlist Optimizer

> Graph-based playlist ordering that minimizes perceptual discontinuity between songs — modeled as a Traveling Salesman Problem variant.

**Course:** Algorithmics (MTAT.03.238) — University of Tartu, Institute of Computer Science  
**Authors:** Kirill Ryrmak, Heigo Tornik, Erik Presnov

---

## Overview

Random shuffle on music streaming platforms frequently creates jarring transitions — a 140 BPM rock track followed by a 60 BPM ballad is a common example of what this project aims to eliminate.

We model a playlist as a **weighted complete graph**, where each song is a node and edge weights represent the perceptual cost of transitioning between two songs. The optimization goal is finding an ordering that **minimizes total transition cost** while preserving musical diversity. Three algorithms are implemented and benchmarked: **Nearest Neighbor Algorithm (NNA)**, **2-opt Local Search**, and **Simulated Annealing (SA)**.

The target was a 50–60% improvement over random baseline. The combined NNA + 2-opt approach achieved **56% reduction** in transition cost.

---

## Problem Formulation

### Audio Features (via Spotify API)
| Feature | Weight | Description |
|---------|--------|-------------|
| Tempo (BPM) | 0.4 | Most perceptually impactful |
| Energy | 0.3 | Intensity and activity level |
| Valence | 0.2 | Musical mood (0 = sad, 1 = happy) |
| Key compatibility | 0.1 | Harmonic compatibility penalty |

### Transition Cost Function
```
cost(song_i, song_j) = 0.4 × |tempo_i - tempo_j|_norm
                     + 0.3 × |energy_i - energy_j|
                     + 0.2 × |valence_i - valence_j|
                     + 0.1 × key_penalty(i, j)
```

All features are normalized to [0, 1] before combining.

---

## Dataset

- Source: 114,000-song Spotify dataset filtered by popularity (score > 83)
- Working dataset: **30 diverse tracks**
- BPM range: 67.5 – 173.9
- Energy range: 0.23 – 0.96
- Valence range: 0.12 – 0.96
- Total pairwise transitions: **870** (stored in a precomputed 30×30 cost matrix for O(1) lookup)

Benchmarks also run on synthetically generated playlists of sizes: **5, 10, 20, 50, 100, 200, 500, 1000**.

---

## Algorithms

### 1. Nearest Neighbor Algorithm (NNA)
A greedy constructive heuristic. Starting from a random song, it iteratively picks the unvisited song with the lowest transition cost.

- Time complexity: **O(n²)**
- Average cost on 30-song dataset: **450**
- Improvement over random: **52%**
- Limitation: susceptible to local optima

### 2. 2-opt Local Search
An iterative improvement algorithm that repeatedly tests whether reversing a segment of the route reduces total cost, accepting the swap if it does.

- Time complexity: **O(n² × k)**, where k = iterations until convergence (typically 5–15)
- Standalone average cost: **443** → **53% improvement**
- **NNA + 2-opt** average cost: **420** → **56% improvement** in ~1.5 ms

### 3. Simulated Annealing (SA)
A probabilistic global optimization method. Accepts worse solutions with probability `exp(-Δcost / T)`, with cooling schedule `T × 0.9999` per iteration.

Neighbor generation strategy:
- 60% — 2-opt moves
- 25% — random swaps
- 15% — double-bridge perturbations (for escaping deep local optima)

- 200,000 iterations, 20 independent runs
- Average cost: **427** → **55% improvement**
- Runtime: ~**60 seconds** (impractical for real-time use)

---

## Results Summary

| Algorithm | Avg. Cost | Improvement | Runtime |
|-----------|-----------|-------------|---------|
| Random Shuffle | ~951 | — | — |
| Random + 2-opt | 443 | 53.4% | fast |
| NNA | 450 | 52.0% | fast |
| **NNA + 2-opt** | **420** | **55.8%** | **~1.5 ms** |
| Simulated Annealing | 427 | 55.1% | ~60 s |

### Key Findings

- **NNA + 2-opt** is the best practical choice: strong cost reduction with near-instant runtime.
- **SA** matches NNA + 2-opt in quality but is ~40,000× slower — unsuitable for production.
- **NNA alone** plateaus on large playlists; 2-opt refinement is essential for scalability.
- For **small playlists (< 20 songs)**, random orderings occasionally approach the optimum, so improvement margins are smaller.
- **Tempo** (highest weight) is optimized most visibly; **valence** and **key** (lower weights) remain somewhat noisy.

---

## Project Structure

```
├── dataset.csv                  # Full 114k-song Spotify dataset
├── selected_songs_30.json       # Filtered 30-song working set
├── selected_songs_30.csv
├── cost_matrix.csv              # Precomputed 30×30 transition cost matrix
├── cost_matrix_detailed.csv
├── Smart_music_optimizer.ipynb  # Main notebook (preprocessing, algorithms, benchmarks, visualizations)
└── README.md
```

### Notebook Structure

| Section | Description |
|---------|-------------|
| `# Preprocessing` | Dataset loading, filtering, statistics, cost matrix construction |
| `# Testcases / random shuffling` | Baseline random shuffle generation |
| `# Algorithms` | NNA, 2-opt, NNA+2-opt, Simulated Annealing implementations |
| `# Benchmarks` | Performance testing across playlist sizes 5–1000 |
| `# Visualizations` | Cost matrix heatmap, before/after flow, algorithm comparison charts |

---

## Getting Started

### Requirements

```bash
pip install numpy pandas matplotlib seaborn
```

The notebook was developed in **Google Colab**. Dataset files should be placed in the working directory or mounted via Google Drive (see the `PATH` variable at the top of each section).

### Running

1. Open `Smart_music_optimizer.ipynb` in Colab or Jupyter.
2. Run the **Preprocessing** section to generate `selected_songs_30.json` and `cost_matrix.csv`.
3. Run the **Algorithms** section to execute and evaluate each optimizer.
4. Run **Benchmarks** to reproduce the playlist-size scaling results.
5. Run **Visualizations** to generate all plots.

---

## Visualizations

The notebook produces the following figures:

- **Cost matrix heatmap** — 30×30 pairwise transition costs
- **Algorithm comparison bar chart** — average costs per algorithm with improvement percentages
- **Before/After playlist flow** — tempo, energy, valence profiles for random vs. optimized ordering
- **Improvement vs. playlist size** — scaling behavior for each algorithm
- **Efficiency plot** — cost reduction vs. execution time trade-off
- **Feature-by-feature comparison** — how well each audio feature gets optimized

---

## License

Academic project — University of Tartu, 2024/2025.
