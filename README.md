# SYDE (Spherical Yield Dynamic Equilibrium)

*A Relational Dynamical System with Inertial Coupling and Bounded Instability*

---

## Abstract

SYDE (Spherical Yield Dynamic Equilibrium) is a computational model of **relationally generated dynamical systems** in which:

- Nodes represent state vectors (entities)
- Edges represent dynamic relational couplings
- System evolution emerges from local alignment, global mean-field attraction, stochastic perturbation, and inertial adaptation

The system exhibits **bounded instability**, where fluctuations persist while global coherence remains structurally constrained.

A coherence order parameter is introduced to quantify emergent structure formation.

---

## Core Idea

SYDE assumes:

> Systems are not composed of static entities, but of continuously generated relations that define those entities.

---

## System Overview

The system is modeled as a dynamic graph:

- **V**: nodes (state vectors in ℝⁿ)
- **E**: edges (relations between nodes)
- **Dynamics**: edge evolution + node coupling dynamics
- **Constraint**: spherical normalization

---

## Key Features

- Relational emergence (no static ontology)
- Local + global coupling dynamics
- Stochastic environmental perturbation
- Inertial adaptation (memory effect)
- Spherical manifold constraint
- Emergent coherence metric

---

## Mathematical Structure

### 1. Node States

Each node is a vector:

\[
s_i \in \mathbb{R}^d
\]

---

### 2. Relational Dynamics

Edges evolve based on distance:

- If distance > `tension_max` → edge removed
- If distance < `tension_min` → probabilistic decay
- New edges may emerge via stochastic generation

---

### 3. Node Evolution Equation

Each node evolves according to:

\[
s_i(t+1) =
s_i(t)
+ \alpha ( \bar{s}_{N(i)} - s_i )
+ \lambda ( \bar{s} - s_i )
+ \xi_i(t)
\]

Where:

- \( \bar{s}_{N(i)} \): local neighborhood mean
- \( \bar{s} \): global mean field
- \( \alpha \): inertial alignment coefficient
- \( \lambda \): global coupling strength
- \( \xi_i(t) \): stochastic noise

---

### 4. Spherical Constraint

System states are projected onto a hypersphere:

\[
\|S\| = \sqrt{N}
\]

This enforces bounded energy and prevents divergence.

---

## Coherence Function

SYDE defines a global order parameter:

\[
C = R \times N \times A
\]

Where:

### R — Relational Density Coherence
Measures optimal edge density structure:

\[
R = \exp\left(-\frac{(d - d_0)^2}{2\sigma^2}\right)
\]

---

### N — Node Alignment Coherence
Measures contraction toward mean field:

\[
N = \exp(-\text{avg distance to centroid})
\]

---

### A — Attractor Stability
Measures variance stability:

\[
A = \exp(-|\mathrm{Var}(S) - V^*|)
\]

---

## Python Implementation

```python
import numpy as np


class SYDESystem:
    def __init__(self, n_nodes=25, dim=3, edge_prob=0.2):

        self.n_nodes = n_nodes
        self.dim = dim

        self.nodes = {
            i: np.random.randn(dim)
            for i in range(n_nodes)
        }

        self.edges = set()
        for i in range(n_nodes):
            for j in range(i + 1, n_nodes):
                if np.random.rand() < edge_prob:
                    self.edges.add((i, j))

        self.tension_max = 3.0
        self.tension_min = 0.2
        self.noise_level = 0.08
        self.target_var = 1.5
        self.lambda_global = 0.1
        self.alpha = 0.15

    def step(self):

        new_edges = set()

        for (i, j) in self.edges:

            dist = np.linalg.norm(self.nodes[i] - self.nodes[j])

            if dist > self.tension_max:
                continue

            if dist < self.tension_min and np.random.rand() > 0.5:
                continue

            new_edges.add((i, j))

            if np.random.rand() < 0.02:
                k = np.random.randint(0, self.n_nodes)
                if k != i and k != j:
                    new_edges.add(tuple(sorted((i, k))))

        self.edges = new_edges

        new_nodes = {}

        states = np.array(list(self.nodes.values()))
        global_mean = np.mean(states, axis=0)

        for i in self.nodes:

            neighbors = [
                self.nodes[b] if a == i else self.nodes[a]
                for (a, b) in self.edges
                if a == i or b == i
            ]

            state_i = self.nodes[i]

            if neighbors:
                neighbors = np.array(neighbors)
                alignment = self.alpha * (
                    np.mean(neighbors, axis=0) - state_i
                )
            else:
                alignment = 0

            global_force = global_mean - state_i
            noise = np.random.randn(self.dim) * self.noise_level

            new_nodes[i] = (
                state_i
                + alignment
                + self.lambda_global * global_force
                + noise
            )

        self.nodes = new_nodes
        self._project_to_sphere()

    def _project_to_sphere(self):

        states = np.array(list(self.nodes.values()))
        norm = np.linalg.norm(states)

        if norm == 0:
            return

        R = np.sqrt(self.n_nodes)

        for i in self.nodes:
            self.nodes[i] = R * self.nodes[i] / norm

    def coherence(self):

        states = np.array(list(self.nodes.values()))

        max_edges = self.n_nodes * (self.n_nodes - 1) / 2
        density = len(self.edges) / max_edges if max_edges > 0 else 0

        R = np.exp(-((density - 0.15) ** 2) / (2 * 0.15 ** 2))

        mean_state = np.mean(states, axis=0)
        avg_dist = np.mean(np.linalg.norm(states - mean_state, axis=1))
        N = np.exp(-avg_dist)

        var = np.var(states)
        A = np.exp(-abs(var - self.target_var))

        return float(R * N * A)


if __name__ == "__main__":

    system = SYDESystem()

    for t in range(150):
        system.step()
        print(f"t={t:03d} | coherence={system.coherence():.5f}")
