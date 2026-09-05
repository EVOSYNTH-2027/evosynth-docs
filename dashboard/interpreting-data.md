# Interpreting the Dashboard

When you open the React dashboard during an active optimization run, you are presented with several distinct visualization panes.

## 1. Global Metrics (The Header)
At the top of the dashboard, you will see the **Global Best Accuracy** and the **Total Evaluations**. This is your immediate indicator of whether the run is finished, and what the absolute best configuration found across all algorithms currently achieves.

## 2. Algorithm Accuracy Tracker (Line Chart)
This Recharts line graph plots the **Best Accuracy** of each algorithm over time (iterations).
- **Steep Curves:** Indicate an algorithm that is rapidly exploiting a gradient.
- **Flat Lines (Stagnation):** Indicate an algorithm that is trapped in a local minimum.
- **Sudden Spikes:** These usually occur immediately after an Elite Migration event. For example, if GA finds a breakthrough, you might see DE and PSO suddenly spike in the next iteration as they are "warm-seeded" with GA's elite.

## 3. The UCB1 Bandit Scores (Bar Chart)
This is the most critical metric for understanding the Meta-Engine. 
The bar chart shows the normalized $[0, 1]$ [Composite Score](../concepts/composite-scoring.md) for each algorithm.
- If CMA-ES has a bar at `0.95` and GA has a bar at `0.20`, CMA-ES is currently dominating the competition.
- Because of the UCB1 exploration parameter, you will occasionally see an underperforming algorithm's score slowly tick upward simply because it hasn't been selected in a while. This forces the engine to eventually give it a small burst of CPU time.

## 4. Tier Status (Badges)
Below the UCB scores, you will see which algorithms are currently assigned to **Tier 1** (Exploration, Low Budget) and **Tier 2** (Exploitation, High Budget). Watch these badges swap in real-time as the UCB scores shift!
