---
title: "Puzzle: Maximising Expected Value When Rolling a Dice"
description: "A probability puzzle about maximising expected gain from rolling a six-sided die."
date: 2025-04-20T23:24:00+10:00
lastmod: 2025-04-20T23:31:00+10:00
categories:
  - Learning
tags:
  - Puzzle
  - Probability
math: true
---

## Puzzle

You may roll a standard six-sided die multiple times. Each roll adds its face value to your total score. However, if you roll a number that you've already rolled before, your total score becomes zero and the game ends immediately. You may choose to stop at any time. What is the optimal number of rolls to maximise the expected value?

## Approach

Clearly, you can roll at most 6 times.

First, let’s compute the probability of rolling distinct numbers — this is straightforward:

| Rolls $n$ | Probability of all distinct outcomes $P(n)$                                                             |
|----------|----------------------------------------------------------------------------------------------------------|
| 1        | 1                                                                                                        |
| 2        | $\frac{5}{6}$                                                                                            |
| 3        | $\frac{5}{6} \cdot \frac{4}{6} = \frac{20}{36}$                                                           |
| 4        | $\frac{5}{6} \cdot \frac{4}{6} \cdot \frac{3}{6} = \frac{60}{216}$                                       |
| 5        | $\frac{5}{6} \cdot \frac{4}{6} \cdot \frac{3}{6} \cdot \frac{2}{6} = \frac{120}{1296}$                   |
| 6        | $\frac{5}{6} \cdot \frac{4}{6} \cdot \frac{3}{6} \cdot \frac{2}{6} \cdot \frac{1}{6} = \frac{120}{7776}$ |
| 7        | 0                                                                                                        |

Now let’s calculate the expected value (EV) of stopping at each $n$.

For $n = 1$, the expected value is simply $3.5$ (average of 1 to 6).

For $n = 2$, we need to compute the distribution of the sum of two distinct dice values. The sum table (excluding duplicates) is:

| 🎲\🎲 | 1 | 2 | 3 | 4 | 5 | 6 |
|:----:|--:|--:|--:|--:|--:|--:|
| **1** | 2 | 3 | 4 | 5 | 6 | 7 |
| **2** | 3 | 4 | 5 | 6 | 7 | 8 |
| **3** | 4 | 5 | 6 | 7 | 8 | 9 |
| **4** | 5 | 6 | 7 | 8 | 9 |10 |
| **5** | 6 | 7 | 8 | 9 |10 |11 |
| **6** | 7 | 8 | 9 |10 |11 |12 |

The main diagonal (where both dice show the same number) must be excluded. That leaves us with $6 \times 6 - 6 = 30$ cases. 

If you observe closely, the table is symmetric around the anti-diagonal, and the average of any symmetric pair is 7 (e.g., 6 and 8, 5 and 9, etc.). Therefore, the expected sum is 7.

So, the expected value for $n = 2$ is:

$$
P(2) \times 7 = \frac{5}{6} \times 7 = \frac{35}{6} \approx 5.83
$$

Here’s a key insight: even with the restriction on duplicates, the expected value of $n$ rolls (conditioned on all being distinct) is simply $3.5 \times n$.

We can confirm this for $n = 6$, where the only possible sequence is $(1+2+3+4+5+6) = 21 = 3.5 \times 6$.

So, the general formula becomes:

$$
E(n) = P(n) \times 3.5 \times n
$$

Calculating the expected value for each possible roll count:

| Rolls $n$ | Expected Value $E(n)$                                                                                     |
|----------|------------------------------------------------------------------------------------------------------------|
| 1        | 3.5                                                                                                        |
| 2        | $\frac{5}{6} \times 3.5 \times 2 = \frac{5}{6} \times 7 \approx 5.83$                                      |
| 3        | $\frac{20}{36} \times 10.5 \approx 5.83$                                                                   |
| 4        | $\frac{60}{216} \times 14 \approx 3.88$                                                                    |
| 5        | $\frac{120}{1296} \times 17.5 \approx 1.62$                                                                |
| 6        | $\frac{120}{7776} \times 21 = 0.32$                                                                        |

Hence, the expected value is maximised when you roll **2 or 3 times**, both yielding approximately **5.83**.
