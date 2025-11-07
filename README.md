# **Hybrid Evolutionary Algorithm for TSP**

The algorithm is a **Hybrid Evolutionary Algorithm (EA)**. It is not a standard Genetic Algorithm (GA) but a custom solver that integrates three main components:
1.  A **Greedy & Random** strategy for population initialization.
2.  A **Genetic Algorithm** ($\mu+\lambda$ model) using operators for evolution.
3.  A **Probabilistic Hill Climber** for fast local optimization of solutions.

---

## **1. Dynamic Initialization**

A key feature of this algorithm is its dynamic population initialization. Instead of starting with 100 completely random individuals, the population is initialized with a dynamic mix random and greedy solutions to accelerate convergence.

The `POPULATION_SIZE` is fixed at 100 individuals, composed as follows:

* **3 Fixed individual:**
    * **1 Greedy - Middle Start:** A single tour is generated using the Nearest Neighbor heuristic, starting from a "middle-cost" city (the city with the lowest total outgoing edge cost). This is an assumption that serves to simplify the problem by admitting that the return route will most likely be less expensive because it is probably quite close.
    * **1 Ordered Tour:** A simple tour `[0, 1, 2, ...N]`.
    * **1 Reversed Tour:** A tour `[...N, 2, 1, 0]`.

* **Greedy (Dynamic %):**
    * Approximately 50% of the population is built using the **Nearest Neighbor (Greedy)** heuristic.
    * To ensure variety, each of these individuals starts from a different, randomly selected city.
    * This number is dynamic; for a 10-city problem, it will create fewer greedy-randoms than for a 100-city problem.

* **Random (Dynamic %):**
    * The remaining slots in the population (approx. 50%) are filled with completely random permutations to ensure high genetic diversity and allow for broad exploration.

---

## **2. Evolutionary Model & Selection**

The algorithm uses a **$(\mu+\lambda)$ steady-state model**.
* $\mu = 100$ (Population Size)
* $\lambda = 40$ (Offspring Size)

In each generation, 40 new children are created (offspring). These are added to the main population, creating a combined pool of 140 individuals. **Elitist Truncation** is then applied: the entire pool is sorted by fitness, and only the **top 100** individuals survive to the next generation. This guarantees that the best-found solution is never lost.

### **Parent Selection: Tournament**

Parents are chosen using **Tournament Selection**.
* A "tournament" of $\tau=10$ individuals is randomly sampled from the population.
* The single fittest individual from that sample is selected as a parent.
* This method applies high selective pressure, favoring high-fitness individuals while still giving others a chance to be selected.

---

## **3. Genetic Operators**

Two specialized, permutation-safe operators are used to create offspring.

### **Crossover: Order Crossover (OX)**
* **Probability:** 90%.
* **Why this choice?** OX is a classic and robust operator for permutation problems like TSP. It works by copying a random, contiguous segment from the first parent into the child. The remaining slots are then filled with cities from the second parent, in the order they appear. This effectively preserves the relative order of cities from one parent and the absolute positions of a segment from the other.

### **Mutation: Inversion Mutation**
* **Probability:** 10%.
* **Why this choice?** This is a powerful, TSP-specific mutation. It selects two random points in the tour and reverses the entire segment of cities between them. 

---

## **4. The Hill Climber**
The Hill Climber used is a custom strategy. 
1.  It loops through each city (`i`) in the tour.
2.  For that city, it scans all other possible insertion points (`j`) to find the best new position for that city.
3.  If a move is found that improves the total cost (`best_delta < 0`), the move is executed, and the HC breaks to restart its `while` loop, searching for the next improvement on the newly modified tour.

**A simple example:**
* Imagine the current tour is `[A, B, C, D]` with a total cost of 50.
* The algorithm picks **City B** to check.
* It tries moving `B` to all other spots:
    * Try `[A, C, B, D]` (Cost: 45)
    * Try `[A, C, D, B]` (Cost: 52)
* It finds that `[A, C, B, D]` is the best move for `B` and its cost (45) is better than the original (50).
* The algorithm immediately executes this move. It *breaks* its search and restarts the entire `while` loop with the new, better tour `[A, C, B, D]`.

This approach is faster than a "true" Hill Climber, which would need to scan *all $N \times N$ possible moves* before executing only the single best one.

### **The Solution: Dynamic Application Probability**
To balance performance and quality, the fast HC is applied probabilistically. The probability of an offspring being optimized is dynamic and based on the problem size:

`hc_probability = min(1.0, 100 / N)`

* **Why this formula?** It's a critical trade-off.
* **For Small Problems ($N \le 100$):** The probability is `1.0` (100%). The HC is fast enough, so we run it on *every child* to ensure the highest solution quality (exploitation).
* **For Large Problems ($N=1000$):** The probability drops to `0.1` (10%). The HC is very slow, so we run it rarely. This saves immense computation time and allows the GA's Crossover/Mutation operators to handle the "exploration" part of the search.

## Final Results

| Problem Type | Cities | Best Cost |
| :--- | ---:| :--- |
| G | 10 | 1497.66 |
| G | 20 | 1755.51 |
| G | 50 | 2629.99 |
| G | 100 | 4105.70 |
| G | 200 | 5674.72 |
| G | 500 | 8995.91 |
| G | 1000 | 12983.08 |
| R1 | 10 | 184.27 |
| R1 | 20 | 337.29 |
| R1 | 50 | 544.59 |
| R1 | 100 | 708.92 |
| R1 | 200 | 1041.95 |
| R1 | 500 | 1648.04 |
| R1 | 1000 | 2442.38 |
| R2 | 10 | -411.70 |
| R2 | 20 | -861.67 |
| R2 | 50 | -2276.90 |
| R2 | 100 | -4722.13 |
| R2 | 200 | -9641.09 |
| R2 | 500 | -24584.44 |
| R2 | 1000 | -49471.05 |
| TEST | 20 | 2823.79 |
