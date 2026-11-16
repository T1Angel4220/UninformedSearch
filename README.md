# Uninformed Search Agent — 10×20 Grid

> An interactive Python implementation of four classic uninformed search strategies over a discrete 10×20 state space, with a full Tkinter GUI dashboard.

[![Python](https://img.shields.io/badge/Python-3.8%2B-blue?logo=python)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![GUI](https://img.shields.io/badge/GUI-Tkinter-informational)](https://docs.python.org/3/library/tkinter.html)
[![No External Libs](https://img.shields.io/badge/Dependencies-stdlib%20only-green)](https://docs.python.org/3/library/)

---

## Overview

This project implements an **uninformed search agent** that navigates a discrete Cartesian grid representing a 10×20 classroom floor plan. The agent has no prior knowledge of the environment and no heuristic guidance — it perceives only its current state $s = (x, y)$ and applies a formal **Successor Function** respecting the grid boundaries.

All frontier data structures (**Queue** FIFO and **Stack** LIFO) are implemented **from scratch** without any external libraries or Python's `collections.deque`.

---

## Problem Formulation

| Component | Definition |
|-----------|------------|
| **State Space $S$** | $s = (x, y),\quad x \in [1,10],\quad y \in [1,20]$ — total $\|S\| = 200$ states |
| **Initial State $S_0$** | Random or user-selected $(x_0, y_0) \in S$ |
| **Goal State $S_g$** | Fixed at $(1, 10)$ (classroom exit) |
| **Goal Test** | `goal_test(s) → True` iff $s = (1, 10)$ |
| **Action Set $A$** | $\{\text{North},\ \text{South},\ \text{East},\ \text{West}\}$ |
| **Path Cost $g(n)$** | Unit cost: $c(s, a, s') = 1$ per move |

### Successor Function $\text{Succ}(s, a)$

The function applies only boundary physics — no shortcut conditionals:

$$
\text{Succ}((x,y), a) = \begin{cases}
(x,\ y+1) & \text{if } a = \text{North},\ y < 20 \\
(x,\ y-1) & \text{if } a = \text{South},\ y > 1 \\
(x+1,\ y) & \text{if } a = \text{East},\ x < 10 \\
(x-1,\ y) & \text{if } a = \text{West},\ x > 1
\end{cases}
$$

---

## Implemented Algorithms

### 1. Breadth-First Search (BFS)
- **Frontier:** custom `Queue` (FIFO)
- **Strategy:** level-by-level expansion; guarantees the shallowest solution
- **Complexity:** Time $O(b^{d+1})$, Space $O(b^{d+1})$
- **Nodes generated:** $b + b^2 + \cdots + b^d + (b^{d+1} - b)$

### 2. Depth-First Search (DFS)
- **Frontier:** custom `Stack` (LIFO)
- **Strategy:** deepest branch first, with visited-state tracking to ensure completeness
- **Complexity:** Time $O(b^m)$, Space $O(b \times m)$
- **Nodes generated (worst case):** $b^m$

### 3. Iterative Deepening DFS (IDDFS)
- **Frontier:** custom `Stack` (LIFO) with depth limit $L$ increasing from $0, 1, 2, \ldots$
- **Strategy:** combines DFS memory efficiency with BFS optimality
- **Complexity:** Time $O(b^d)$, Space $O(b \times d)$
- **Nodes generated:** $\displaystyle\sum_{i=1}^{d} (d - i + 1)\, b^i$

### 4. Bidirectional BFS
- **Frontiers:** two custom `Queue`s (FIFO) expanding simultaneously from $S_0$ and $S_g$
- **Strategy:** expands both frontiers until they intersect at a meeting point
- **Complexity:** Time $O(b^{d/2})$, Space $O(b^{d/2})$
- **Nodes generated:** $2 \times b^{d/2}$

---

## Complexity Comparison

| Strategy | Exact Formula | Time | Space | Complete | Optimal |
|:---------|:-------------|:----:|:-----:|:--------:|:-------:|
| **BFS** | $b + b^2 + \cdots + b^d + (b^{d+1} - b)$ | $O(b^{d+1})$ | $O(b^{d+1})$ | ✅ | ✅ |
| **DFS** | Depends on branch (max $b^m$) | $O(b^m)$ | $O(b \times m)$ | ✅ (w/ visited) | ❌ |
| **IDDFS** | $(d)b + (d-1)b^2 + \cdots + (1)b^d$ | $O(b^d)$ | $O(b \times d)$ | ✅ | ✅ |
| **Bidir. BFS** | $2 \times b^{d/2}$ | $O(b^{d/2})$ | $O(b^{d/2})$ | ✅ | ✅ |

> Where $b$ = branching factor, $d$ = solution depth, $m$ = maximum depth of state space.

---

## Project Architecture

```
UninformedSearch/
├── nucleo/
│   ├── estructuras.py          # Custom Cola (FIFO) and Pila (LIFO) — zero external deps
│   ├── nodo.py                 # Node class: parent pointer, g(n) cost, depth d, ID
│   ├── ambiente.py             # AmbienteCuadricula: 10×20 grid + Successor Function
│   └── metricas.py             # Theoretical & empirical complexity calculator
├── algoritmos/
│   ├── anchura.py              # BFS  — uses custom Queue
│   ├── profundidad.py          # DFS  — uses custom Stack
│   ├── profundidad_iterativa.py# IDDFS — uses custom Stack with depth limit
│   └── bidireccional.py        # Bidirectional BFS — uses two custom Queues
├── visualizador/
│   └── aplicacion.py           # Tkinter GUI dashboard (4 tabs)
├── pruebas/
│   └── probar_busquedas.py     # Automated test suite
├── main.py                     # Entry point
├── .gitignore
├── LICENSE
└── README.md
```

---

## GUI Features

The Tkinter dashboard (`visualizador/aplicacion.py`) provides four interactive tabs:

| Tab | Description |
|-----|-------------|
| **🗺️ Grid View** | Animated 10×20 classroom canvas. Click any cell to set $S_0$; the agent explores and traces the optimal path in gold. |
| **🌲 Search Tree** | Full graphical search tree with high-contrast neon coloring. Zoom in/out (10%–300%) via buttons or `Ctrl+Scroll`. Bidirectional BFS renders two trees facing each other with the meeting point highlighted. |
| **📋 Node Hierarchy** | Sortable table of every generated `Nodo` instance: ID, state $(x,y)$, parent ID, action, $g(n)$, depth $d$. |
| **📊 Metrics Analysis** | Side-by-side comparison of theoretical formulas vs. empirical results (nodes expanded, nodes generated, max frontier, optimal path length, execution time in ms). |

**Controls:**
- 🎲 **Random $S_0$** — generates a new random starting state
- ▶️ **Solve** — runs the selected algorithm and animates the search
- 🚀 **Compare All 4** — executes all strategies from the same $S_0$ and fills the metrics table
- **Speed slider** (5–300 ms per animation frame)

---

## Installation & Usage

**Prerequisites:** Python 3.8+ (Tkinter is included in the standard library — no pip installs needed)

```bash
# 1. Clone the repository
git clone https://github.com/T1Angel4220/UninformedSearch.git
cd UninformedSearch

# 2. Run the GUI application
python main.py

# 3. (Optional) Run the automated test suite
python pruebas/probar_busquedas.py
```

> No virtual environment or external packages required — this project uses only the Python standard library.

---

## License

MIT License — Copyright (c) 2024-2026 Angel Ayuquina. See [LICENSE](LICENSE) for details.
