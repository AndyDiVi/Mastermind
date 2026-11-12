# Laboratory 2 – Solving Mastermind using a Genetic Algorithm
<img width="250" height="287" alt="Mastermind" src="https://github.com/user-attachments/assets/725f09a9-48db-4bfd-b4ac-3738b0202ca6" />
<img width="347" height="258" alt="mastermind" src="https://github.com/user-attachments/assets/1b33365a-a729-4eff-bbee-cdc54422b0e1" />

Authors: Graziani, Di Via & Hiller-Szala

## Overview

This laboratory implements a genetic algorithm in MATLAB to discover a target combination in Mastermind.

Each candidate solution is represented as a sequence of symbols. A population of candidates evolves through fitness evaluation, tournament selection, crossover, and mutation until the target is found or a maximum number of generations is reached.

The laboratory includes:

- Random generation of the target and initial population.
- Fitness evaluation with support for repeated symbols.
- Tournament-based parent selection.
- Three crossover operators.
- Random single-gene mutation.
- Selection of the best offspring for reproduction.
- Visualization of average and best fitness across generations.

The implementation simulates the game by evaluating candidates against a target stored in the program.

## Problem Definition

The default configuration uses sequences of six symbols drawn from an alphabet of six possible values:

```math
\mathcal{A} = \{1,2,3,4,5,6\}
```

```math
N_b = 6
```

The symbols represent six colors: Green, Red, Yellow, White, Black, and Pink.

Repetitions are allowed in both the target and the candidate sequences.

The objective is to find a candidate sequence that matches the target in every position:

```math
x = t
```

Here, $x$ denotes a candidate sequence and $t$ denotes the target.

## Solution Representation

Each individual is a row vector containing $N_b$ symbols.

The initial population contains $P=10$ individuals, represented by a matrix with dimensions:

```math
X^{(0)} \in \mathcal{A}^{P\times N_b}
```

The default initialization is:

```matlab
Nb = 6;
alphabet = 1:6;
P = 10;

target = randi(, 1, Nb);[1][6]
test = randi(, P, Nb);[1][6]
```

In the code, `test` is the population of candidate solutions, not a machine learning test dataset.

No external dataset is required.

## Fitness Function

The fitness function rewards two types of matches:

- Correct symbols in the correct positions.
- Correct symbols in the wrong positions.

The fitness of a candidate is:

```math
f(x,t) = w_1N_{\mathrm{exact}}(x,t) + w_2N_{\mathrm{misplaced}}(x,t)
```

The default weights are:

```math
w_1 = 2
```

```math
w_2 = 1
```

Therefore:

```math
f(x,t) = 2N_{\mathrm{exact}}(x,t) + N_{\mathrm{misplaced}}(x,t)
```

### Exact matches

Exact matches are counted by comparing corresponding positions:

```math
N_{\mathrm{exact}}(x,t) = \sum_{k=1}^{N_b}\mathbf{1}_{\{x_k=t_k\}}
```

### Misplaced matches

To handle repeated symbols without double counting, the implementation first counts the occurrences of each symbol in the candidate and the target.

Let $c_x(a)$ and $c_t(a)$ denote the number of occurrences of symbol $a$ in the candidate and target, respectively.

The number of matching symbols, regardless of position, is:

```math
N_{\mathrm{matching}}(x,t) = \sum_{a\in\mathcal{A}}\min\left(c_x(a),c_t(a)\right)
```

Exact matches are then excluded to obtain misplaced matches:

```math
N_{\mathrm{misplaced}}(x,t) = N_{\mathrm{matching}}(x,t) - N_{\mathrm{exact}}(x,t)
```

### Maximum fitness

With the default weights, the maximum fitness is achieved only by matching every position:

```math
f_{\max} = w_1N_b = 12
```

The MATLAB fitness function is:

```matlab
w1 = 2;
w2 = 1;

fitness = @(Ncgpg, Ncgps) w1 * Ncgpg + w2 * Ncgps;
```

The variable names used in the implementation are:

| Variable | Meaning |
|---|---|
| `Ncgpg` | Number of correct symbols in the correct positions |
| `Ncgps` | Number of correct symbols in the wrong positions |
| `fitness_individuals` | Fitness score for each candidate |

## Methodology

### 1. Population selection

At the beginning of each generation, the algorithm selects the best $P=10$ candidates from the current population.

After the first reproduction cycle, the current population contains 20 offspring, so this step reduces the reproductive pool from 20 candidates to 10.

```matlab
[~, idx] = maxk(fitness_individuals, P);

test = test(idx, :);
fitness_individuals = fitness_individuals(idx);
```

Parents from the previous generation are not explicitly retained alongside their offspring.

### 2. Tournament parent selection

For each reproduction event:

1. Randomly select four distinct individuals from the reproductive pool.
2. Compare their fitness scores.
3. Choose the two highest-scoring individuals as parents.

The tournament size is:

```math
T = 4
```

The function is called using:

```matlab
[parent1, parent2] = tournament_selection( ...
    test, fitness_individuals, 4);
```

The reproductive pool must contain at least four individuals for this configuration.

### 3. Crossover

The selected parents generate two children.

Crossover is performed with probability:

```math
P_c = 0.9
```

If crossover is not performed, the children remain clones of their respective parents before mutation.

The implementation supports three crossover operators.

#### One-point crossover

A random cut position is selected between 1 and $N_b-1$.

The children exchange the portions of the parent sequences after the cut.

```matlab
[child1, child2] = crossover( ...
    parent1, parent2, alphabet, Pc, 1);
```

This is the default operator used in the documented execution.

#### Two-point crossover

Two cut positions are drawn and sorted.

The segment between the cuts is exchanged between the parents.

```matlab
[child1, child2] = crossover( ...
    parent1, parent2, alphabet, Pc, 2);
```

In the supplied implementation, the two cuts may coincide. In that case, no segment is exchanged.

#### Uniform crossover

A random mask determines which positions are exchanged.

Each position has a 50% probability of being selected for exchange, and the same mask is used for both children.

```matlab
[child1, child2] = crossover( ...
    parent1, parent2, alphabet, Pc, 3);
```

### 4. Mutation

After crossover, each child is independently passed to the mutation function.

The default mutation probability is:

```math
P_m = \frac{1}{N_b} = \frac{1}{6}
```

For each child, the implemented mutation procedure:

1. Draws one random value to determine whether mutation occurs.
2. Selects one random gene position if mutation is triggered.
3. Replaces that gene with a randomly selected alphabet symbol different from its current value.

Therefore, a child undergoes either no mutation or exactly one gene replacement per call.

```matlab
child1 = mutation(child1, alphabet, Pm);
child2 = mutation(child2, alphabet, Pm);
```

The alphabet must contain at least two distinct symbols for this replacement procedure.

### 5. Offspring generation and replacement

Each generation performs 10 reproduction events, producing two children per event:

```math
P_{\mathrm{offspring}} = 2P = 20
```

The offspring replace the current population.

Their fitness values are evaluated, and the algorithm records:

- The highest offspring fitness.
- The average offspring fitness.

If no stopping condition is met, the best 10 offspring form the reproductive pool for the next generation.

### 6. Stopping criteria

The search stops when either:

- A candidate reaches the maximum fitness of 12.
- The number of generations reaches 500.

The default loop condition is:

```matlab
while max(fitness_individuals) < 12 && count < max_count
```

If the target is already present in the initial population, the algorithm stops without generating offspring.

## Algorithm Parameters

| Parameter | Default value |
|---|---:|
| Sequence length | 6 |
| Alphabet | `1:6` |
| Initial population | 10 |
| Selected reproductive pool | 10 |
| Offspring per generation | 20 |
| Tournament size | 4 |
| Crossover probability | 0.9 |
| Mutation probability per child | 1/6 |
| Exact-match weight | 2 |
| Misplaced-match weight | 1 |
| Maximum fitness | 12 |
| Maximum generations | 500 |
| Default crossover | One-point |

## Results

The execution documented in the report uses the target:

```text
4 6 6 1 3 3
```

The algorithm finds the exact combination after 41 generations.

Reported output:

```text
Convergence reached in 41 iterations.
The target combination is: 4 6 6 1 3 3
```

| Result | Value |
|---|---|
| Target combination | `4 6 6 1 3 3` |
| Recovered combination | `4 6 6 1 3 3` |
| Generations required | 41 |
| Final best fitness | 12 |

This is the result of one documented execution, not an average over repeated experiments.

Since initialization and genetic operators are random, other executions may require a different number of generations or reach the generation limit without finding the target.

## Fitness Visualization

The script plots the average and best fitness of each newly generated offspring population.

For an offspring population of size $2P$, the average fitness is:

```math
\bar{f}^{(g)} = \frac{1}{2P}\sum_{i=1}^{2P}f\left(x_i^{(g)},t\right)
```

The best fitness is:

```math
f_{\mathrm{best}}^{(g)} = \max_{1\leq i\leq 2P}f\left(x_i^{(g)},t\right)
```

The plot is generated using:

```matlab
if count > 0
    figure();

    plot(average_fitness(1:count));
    hold on;
    plot(best_fitness(1:count));

    title('Fitness plot of the population');
    legend('Average fitness', 'Best fitness');
    xlabel('Iteration');
    ylabel('Fitness value');
end
```

The best fitness is always at least as large as the average fitness within the same population.

However, because the implementation does not explicitly preserve the best parent across generations, the best fitness is not guaranteed to increase monotonically.

## Implementation

The laboratory is implemented in MATLAB using four custom functions.

### Fitness evaluation

```matlab
[Ncgpg, Ncgps, fitness_individuals] = evaluate_fitness( ...
    test, target, alphabet, fitness);
```

Computes exact matches, misplaced matches, and fitness scores for all candidates.

The three outputs are column vectors with one entry per candidate.

### Tournament selection

```matlab
[parent1, parent2] = tournament_selection( ...
    test, fitness_individuals, T);
```

Selects the two best individuals from a randomly sampled tournament.

### Crossover

```matlab
[child1, child2] = crossover( ...
    parent1, parent2, alphabet, Pc, crossover_type);
```

Generates two children using the selected crossover operator.

| `crossover_type` | Operator |
|---|---|
| `1` | One-point crossover |
| `2` | Two-point crossover |
| `3` | Uniform crossover |

### Mutation

```matlab
individual = mutation(individual, alphabet, Pm);
```

Optionally changes one randomly selected gene to a different valid symbol.

The functions include input checks for sequence dimensions, alphabet membership, fitness inputs, and probability ranges.

## Customization

### Reproducible execution

The report includes an optional seed statement.

Uncomment it before generating the target and initial population to make repeated executions reproducible:

```matlab
rng(42);
```

Setting this seed does not imply that the documented 41-generation result will be reproduced, since that result is shown with the seed statement commented out.

### Alternative alphabets

The functions are designed to work with numeric or character alphabets.

Examples include:

```matlab
alphabet =;[1][3][12][30]
```

```matlab
alphabet = ['G', 'R', 'Y', 'W', 'B', 'P'];
```

When changing the alphabet, replace the default `randi([1, 6], ...)` initialization with alphabet-based sampling:

```matlab
random_indices = randi(numel(alphabet), 1, Nb);
target = alphabet(random_indices);

random_indices = randi(numel(alphabet), P, Nb);
test = alphabet(random_indices);
```

Use a homogeneous alphabet with distinct symbols. Mixed numeric and character arrays are not treated as a separate heterogeneous symbol type.

### Sequence length and fitness weights

If the sequence length or fitness weights are changed, update the success threshold consistently.

For the default scoring scheme:

```matlab
max_fitness = w1 * Nb;
```

Replace the hard-coded value `12` in the loop condition, success check, and solution lookup.

The fitness weights should continue to make an exact solution uniquely optimal.

### Crossover selection

Change the final argument of the `crossover` call to choose a different operator:

```matlab
crossover_type = 1; % 1: one-point, 2: two-point, 3: uniform

[child1, child2] = crossover( ...
    parent1, parent2, alphabet, Pc, crossover_type);
```

## Requirements

- MATLAB.
- The main script.
- The four custom functions, either included in the script or available on the MATLAB path:
  - `evaluate_fitness`
  - `tournament_selection`
  - `crossover`
  - `mutation`

No external data files are required.

The active mutation implementation uses `randi`, rather than the commented-out `randsample` alternative.

## How to run

1. Clone or download this repository.
2. Open the main MATLAB script.
3. Ensure that all four custom functions are available.
4. Optionally configure the alphabet, sequence length, population size, and genetic parameters.
5. Optionally enable the random seed.
6. Execute the script.

The script generates the target and initial population, runs the genetic algorithm, prints the outcome, and displays the fitness plot if at least one generation was produced.

## Limitations

- The algorithm evaluates candidates directly against the stored target; it is a simulated optimization experiment rather than a complete solver that reasons only from the history of player guesses and feedback.
- The search is stochastic and is not guaranteed to find the target within 500 generations.
- The documented result represents a single execution.
- The replacement strategy does not explicitly preserve the best parent.
- The default success threshold is hard-coded for six symbols and an exact-match weight of two.
- A generation evaluates multiple candidate sequences, so generation count should not be interpreted as the number of guesses in a standard Mastermind game.

## Conclusions

This laboratory demonstrates a genetic algorithm for discovering a Mastermind target sequence.

Tournament selection favors high-fitness candidates, crossover combines portions of parent sequences, and mutation introduces alternative symbols.

The fitness evaluation accounts for repeated symbols by separating exact matches from misplaced matches without double counting.

In the documented execution, the algorithm recovers the target `4 6 6 1 3 3` after 41 generations, reaching the maximum fitness of 12.

The modular implementation supports experimentation with crossover operators, alphabets, and evolutionary parameters.

---

Laboratory 2 – Graziani, Di Via & Hiller-Szala
