---
name: back-to-basics
description: Philosophy for designing agent systems and harnesses, quality-first profile. Use when architecting, building, or reviewing an agent system, harness, or any long-running automated pipeline and correctness matters more than minimal footprint.
license: MIT
---

# Entropy (quality-first profile)

Agent system design is the art of reducing system entropy.

Add a part only when its absence provably blocks the task; usefulness is not necessity.

Every dependency imports its own disorder; the standard library is the lowest-entropy dependency.

Expect any reply from an external system to arrive as a string, a list of parts, or empty; one small normalization where it enters is cheaper than defenses everywhere.
