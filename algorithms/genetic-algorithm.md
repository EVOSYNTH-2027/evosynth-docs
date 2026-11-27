# Genetic Algorithm (GA)

The Genetic Algorithm in EvoSynth is a classic implementation of Darwinian evolution, adapted to operate exclusively on continuous $[0, 1]$ vectors.

## Mechanics

Because the search space is abstracted away by the [Universal Latent Space](../concepts/latent-space.md), the GA does not need specialized operators for integers or categoricals. 

### 1. Selection (Tournament)
The GA selects parents using **Tournament Selection**. It picks $k = 2$ random individuals from the population and chooses the one with the higher fitness. This is highly effective at maintaining selection pressure without prematurely converging on a single super-individual.

### 2. Crossover (Arithmetic)
EvoSynth uses **Arithmetic Crossover**. For each dimension $d$ in the latent vector, the child's value is a random blend of the parents: $\alpha_d \cdot A_d + (1 - \alpha_d) \cdot B_d$, with a fresh random $\alpha_d \in [0, 1]$ per dimension.

### 3. Mutation (Gaussian)
With a small probability ($10\%$), an allele is mutated. EvoSynth applies **Gaussian Mutation** by adding random noise drawn from a normal distribution $\mathcal{N}(0, 0.1)$. If the resulting value falls outside the $[0, 1]$ bounds, it is clipped.

### 4. Survival (Elitist)
Parents and children are pooled and the fittest individuals survive, so the best configuration found is never lost.

## When GA Excels

The Genetic Algorithm is an exceptional "Explorer". 
Because crossover and mutation can create new vectors that combine traits from disparate parents, GA is highly effective at escaping local minima. In the EvoSynth tier system, GA is often relied upon to discover entirely new basins of attraction.
