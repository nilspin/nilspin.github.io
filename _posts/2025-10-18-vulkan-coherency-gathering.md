---
layout: post
title: Vulkan subgroup operations for coherency gathering
---

We can use Vulkan subgroups for coherency gathering by having threads within a subgroup communicate to discover, vote on, and collectively process similar work. This strategy turns divergent workloads into a series of more coherent, serialized steps within the subgroup, significantly improving GPU performance by reducing branch divergence and optimizing cache usage.

This works because all invocations (threads) in a subgroup execute in lockstep on the same hardware unit (a SIMD unit). Subgroup operations provide a highly efficient, hardware-accelerated way for these invocations to share data without resorting to slower shared or global memory.

# High-Level Strategies

The general pattern : run a loop that repeats until all work within the subgroup is complete.

## 1. Discover & Communicate: 
Each invocation determines its "task type." This could be a material ID, a pointer to a BVH node, a texture handle, or any other data that defines the work to be done. The invocations then share this information with each other using subgroup operations.

> **subgroupBallot(condition)** : Creates a bitmask where the Nth bit is 1 if the condition is true for the Nth invocation. We can answer questions like, "Who else is processing material ID 3?"

> **subgroupShuffle(value, id)** : Directly reads a value from a specific invocation id within the subgroup. This is perfect for sharing arbitrary data like node pointers.

## 2. Vote & Elect a Task: 
The subgroup analyzes the shared information to pick one common task to execute next. A common strategy is to elect the task that is shared by the most invocations.
- The invocation with `gl_SubgroupInvocationID == 0` often acts as the "leader" to analyze the data.
- It can build a histogram of task types by shuffling data from every other invocation.
- Alternatively, it can iterate through potential tasks, using subgroupBallot to count how many invocations would participate in each.

## 3. Regroup & Process: 
All invocations that have the "elected" task execute it. The other, non-participating invocations are effectively masked off for this iteration.
- This creates execution coherence, as all active invocations take the same code path.
- This also creates memory coherence. For example, if all participating invocations are traversing the same BVH node, one can fetch the node data into subgroup-accessible memory (e.g., a variable in the shader), and all others can read it from there, resulting in a single, coalesced memory read instead of many random ones.

## 4. Iterate: 
A `subgroupBarrier()` ensures all invocations finish the current task before proceeding. The subgroup then marks the completed tasks and repeats the process, electing the next most common task among the remaining work. This continues until all tasks are done.
