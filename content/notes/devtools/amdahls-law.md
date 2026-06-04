---
title: Amdahl’s Law
description: Explains Amdahl’s law as an upper bound on speedup from optimizing or parallelizing part of a system.
tags:
  - performance
  - parallelism
  - scalability
date: "2026-06-03T08:44:31Z"
lastmod: "2026-06-03T08:45:11Z"
draft: false
---

Amdahl’s law describes the maximum speedup you can get by improving or parallelizing only part of a system.

It is commonly used in performance engineering to answer questions like:

- “If I make this function 10× faster, how much faster will the whole program be?”
- “If I parallelize this workload across 8 CPU cores, what is the best possible speedup?”
- “Why did adding more machines stop helping?”

The key idea is that **the parts you do not improve become the limiting factor**.

## Basic idea

Suppose a program takes 100 seconds to run.

- 80 seconds are spent in a part that can be parallelized or optimized.
- 20 seconds are spent in a part that cannot be improved.

Even if the 80-second part became infinitely fast, the program would still take 20 seconds.

So the maximum possible speedup is:

```text
original time / best possible new time
= 100 / 20
= 5×
```

That means no optimization to the 80% portion can make the whole program more than 5× faster, because the remaining 20% still has to run.

## Formula

Amdahl’s law is often written as:

```text
Speedup = 1 / ((1 - p) + (p / s))
```

Where:

- `p` is the fraction of the original runtime that benefits from the improvement.
  - Example: if 80% of runtime can be parallelized, `p = 0.8`.
- `s` is the speedup applied to that fraction.
  - Example: if that portion becomes 4× faster, `s = 4`.
- `1 - p` is the fraction that does **not** benefit.

The result is the speedup of the **entire system**, not just the optimized part.

## Example: optimizing part of a program

Imagine a program takes 100 seconds:

```text
60 seconds: database query
40 seconds: application logic
```

You make the database query 3× faster.

Here:

```text
p = 0.6
s = 3
```

Using Amdahl’s law:

```text
Speedup = 1 / ((1 - 0.6) + (0.6 / 3))
        = 1 / (0.4 + 0.2)
        = 1 / 0.6
        = 1.67×
```

So even though the database query is 3× faster, the whole program is only about **1.67× faster**.

The new runtime is:

```text
100 seconds / 1.67 ≈ 60 seconds
```

You can also see it directly:

```text
new database time = 60 / 3 = 20 seconds
application logic = 40 seconds
new total time    = 60 seconds
```

## Example: parallel execution

Amdahl’s law is frequently used for parallel computing.

Suppose 90% of a program can run in parallel, but 10% must remain serial. A **serial** portion is work that must happen one step at a time and cannot be split across multiple workers.

If you run the parallel portion on 4 cores, then:

```text
p = 0.9
s = 4
```

```text
Speedup = 1 / ((1 - 0.9) + (0.9 / 4))
        = 1 / (0.1 + 0.225)
        = 1 / 0.325
        ≈ 3.08×
```

So 4 cores do **not** give a 4× speedup. The serial 10% limits the result.

If you increase to 16 cores:

```text
Speedup = 1 / (0.1 + (0.9 / 16))
        = 1 / (0.1 + 0.05625)
        ≈ 6.4×
```

Even with 16 cores, the speedup is only about 6.4×.

If you had infinitely many cores, the parallel part would take effectively zero time, but the serial 10% would remain:

```text
Maximum speedup = 1 / (1 - p)
                = 1 / 0.1
                = 10×
```

So if 10% of the work is inherently serial, the absolute maximum speedup is 10×, no matter how much hardware you add.

## Intuition: bottlenecks dominate as you improve everything else

A **bottleneck** is the part of a system that limits overall performance.

Amdahl’s law says that when you improve one part of a system, the unimproved parts become a larger share of the remaining runtime.

For example, start with this runtime breakdown:

```text
90% slow computation
10% file I/O
```

If you make the computation 9× faster, the new breakdown becomes:

```text
10% computation
10% file I/O
20% total relative work
```

The file I/O used to be only 10% of runtime. After the optimization, it is now 50% of the new runtime:

```text
file I/O share = 10 / 20 = 50%
```

The bottleneck moved.

This is why performance optimization is often iterative: after one bottleneck is fixed, another appears.

## Important implication

Amdahl’s law teaches that you should optimize the parts of a system that take a large fraction of total runtime.

A small improvement to a huge part of the workload may matter more than a large improvement to a tiny part.

For example:

- Making 90% of a program 2× faster:

```text
Speedup = 1 / (0.1 + 0.9 / 2)
        = 1 / 0.55
        ≈ 1.82×
```

- Making 10% of a program 100× faster:

```text
Speedup = 1 / (0.9 + 0.1 / 100)
        = 1 / 0.901
        ≈ 1.11×
```

The first optimization is much more valuable overall, even though it is less dramatic locally.

## Common misconception

A common mistake is to think:

> “If I make this component 10× faster, the whole system will be 10× faster.”

That is only true if the component accounts for essentially all of the runtime.

If the component is only 20% of runtime, even making it infinitely fast gives at most:

```text
Maximum speedup = 1 / (1 - 0.2)
                = 1 / 0.8
                = 1.25×
```

So a 10× local improvement may produce only a modest global improvement.

## Assumptions and limitations

Amdahl’s law is a simplified model. It assumes:

- The total problem size stays fixed.
- The fraction `p` is measured from the original runtime.
- The improved and unimproved portions are cleanly separable.
- Parallelization has no extra overhead.

Real systems often have overhead such as:

- coordination between threads or machines
- communication costs
- locking and contention
- cache effects
- startup and shutdown costs
- load imbalance, where some workers finish earlier than others

Because of this, real speedups are often lower than Amdahl’s law predicts.

However, the law remains useful because it gives a clear upper bound and a strong intuition: **the non-improved portion limits the total improvement**.

## Relationship to scaling

Amdahl’s law is most relevant when the workload size is fixed and you ask how much faster it can run with more resources.

For example:

> “How much faster can I process this same dataset if I use more CPU cores?”

There is a related idea called **Gustafson’s law**, which looks at a different situation: instead of keeping the problem size fixed, you increase the problem size as more resources become available.

For example:

> “If I have more CPU cores, how much larger a dataset can I process in the same amount of time?”

Amdahl’s law focuses on fixed-size speedup. Gustafson’s law focuses on scaled workloads.

## Summary

Amdahl’s law says:

```text
Overall speedup = 1 / ((unimproved fraction) + (improved fraction / improvement factor))
```

Its main lesson is:

> The maximum benefit of an optimization is limited by the fraction of total work that the optimization affects.

In practical terms: measure where time is actually spent, optimize the dominant bottleneck, and remember that every part you do not improve becomes more important after the optimization.