# Genetic Algorithm (GA)

The Genetic Algorithm in EvoSynth is a classic implementation of Darwinian evolution, adapted to operate exclusively on continuous $[0, 1]$ vectors.

## Mechanics

Because the search space is abstracted away by the [Universal Latent Space](../concepts/latent-space.md), the GA does not need specialized operators for integers or categoricals. 

### 1. Selection (Tournament)
The GA selects parents using **Tournament Selection**. It picks $k$ random individuals from the population and chooses the one with the highest absolute fitness. This is highly effective at maintaining selection pressure without prematurely converging on a single super-individual.

### 2. Crossover (Uniform)
EvoSynth uses **Uniform Crossover**. For each dimension $d$ in the latent vector, there is a 50% chance of inheriting the allele from Parent A, and a 50% chance from Parent B.

### 3. Mutation (Gaussian)
With a small probability (e.g., $10\%$), an allele is mutated. EvoSynth applies **Gaussian Mutation** by adding random noise drawn from a normal distribution $\mathcal{N}(0, \sigma)$. If the resulting value falls outside the $[0, 1]$ bounds, it is clipped.

## When GA Excels

The Genetic Algorithm is an exceptional "Explorer". 
Because uniform crossover can create radically new vectors that share traits from disparate parents, GA is highly effective at escaping local minima. In the EvoSynth tier system, GA is often relied upon to discover entirely new basins of attraction.
