# Particle Swarm Optimization (PSO)

Particle Swarm Optimization is a mathematical optimization algorithm inspired by the flocking behavior of birds. In EvoSynth, it is highly effective at exploiting known local optima.

## Mechanics

Each Individual in PSO is a "particle." Unlike GA, particles have a concept of **Velocity**.

A particle remembers:

1. **Personal Best ($P_{best}$):** The best position it has ever visited.
2. **Global Best ($G_{best}$):** The best position any particle in the swarm has ever visited.

### The Velocity Update

At each iteration, a particle updates its velocity vector using three components:

1. **Inertia:** Its current velocity.
2. **Cognitive Force:** Attraction towards its $P_{best}$.
3. **Social Force:** Attraction towards the $G_{best}$.

$$ V*{t+1} = w V_t + c_1 r_1 (P*{best} - X*t) + c_2 r_2 (G*{best} - X_t) $$

The particle then updates its position by adding the velocity to its current vector.

### Handling Bounds

Because the latent space strictly requires $[0, 1]$ floats, if a velocity update pushes a particle out of bounds, its position is clipped to the boundary. EvoSynth uses an inertia weight $w = 0.5$ and $c_1 = c_2 = 1.5$.

### Starting from a Seeded Population

When PSO receives an already-evaluated island, those configurations become the particles' starting positions and personal bests, and the very first step moves them, so no evaluation is wasted re-scoring a known configuration. Particles are moved in turn, so every particle keeps moving even when the Meta-Engine asks for fewer candidates than the swarm size.

## Elite Injection Synergy

PSO benefits massively from EvoSynth's Cooperative Co-evolution. When PSO's Tier 2 island receives elites from its partner island, its worst particles are reset to those positions, and if an elite beats the swarm's $G_{best}$ it becomes the new $G_{best}$. Instantly, the entire swarm's "Social Force" vector shifts, dragging all particles toward the newly discovered elite region.
