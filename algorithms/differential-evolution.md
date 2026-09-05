# Differential Evolution (DE)

Differential Evolution is a powerful stochastic algorithm that mutates vectors by adding the scaled difference of two population members to a third. 

## Mechanics

DE does not use an explicit velocity vector like PSO, nor does it use a purely random Gaussian mutation like GA. Instead, DE derives its mutation direction and magnitude directly from the current spatial distribution of the population.

### Mutation (The DE/rand/1 Strategy)

For a given target vector $X_i$, EvoSynth selects three mutually exclusive, random vectors from the population: $X_a, X_b, X_c$.

It generates a mutant vector $V_i$:
$$ V_i = X_a + F \cdot (X_b - X_c) $$
Where $F$ is a scaling factor (typically $0.8$).

### Crossover

The mutant vector $V_i$ is then crossed with the original target $X_i$ using binomial crossover to create a trial vector $U_i$. If the fitness of $U_i$ is better than $X_i$, it replaces $X_i$ in the next generation.

## Self-Adapting Variance

Because the mutation step uses the difference between existing vectors ($X_b - X_c$), the mutation step size automatically scales with the population's variance. 

When the population is spread out, the mutations are large (Exploration). As the population converges on a local optimum, the differences shrink, causing the mutations to become extremely fine-grained (Exploitation). This makes DE an excellent all-around algorithm in EvoSynth.
