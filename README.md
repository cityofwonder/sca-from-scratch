# sca-from-scratch

Understanding side-channel analysis by building it from the ground up, then attacking hardware I designed myself.

## Why this project

An attacker only sees the trace—the result with its cause erased. A designer, however, holds the cause. This project uses that advantage: because I build the target, I know the ground truth. I can then attack it to see exactly how that ground truth leaks into a physical signal.

The goal is not to write clean code or design a finished chip. It is to understand the *phenomenon*. I build only the minimum block required for each attack.

## Approach: an abstraction ladder

I move step-by-step from pure math up to real measurements. At every layer, I try to recover the same key using the same CPA (Correlation Power Analysis) engine. This way, if an attack fails, I know the failure comes from whatever was introduced at that specific layer (like noise, glitches, or misalignment).

Two separate tracks share Layer 0, which is the common CPA engine.

### Main track - power / EM (data-value leakage)

| Layer | What | What I learn |
|-------|------|--------------|
| 0 | Pure algorithm + CPA engine (no leakage) | Is my attack code correct? |
| 1 | Behavioral RTL, toggle count as power proxy | Which register leaks? |
| 2 | Gate-level + SDF timing | Where does glitch leakage appear? |
| 4 | FPGA + oscilloscope | Real noise and alignment |

### Parallel track - cache timing (access-address leakage)

| Layer | What | What I learn |
|-------|------|--------------|
| 1 | Python cache model, Prime+Probe in code | Controlled eviction |
| 2 | gem5, real replacement policy | Ground truth from logs |
| 3 | Real PC + perf counters | Noise, black-box reality |
| 4 | Shared LLC between CPU and GPU | Cross-domain leakage |

These two tracks measure different physical quantities (power vs. time) and leak different types of secrets (data values vs. memory addresses). However, they share the same basic structure: hidden state -> physical side effect -> observable signal -> statistical recovery of the state. Because of this shared structure, a single CPA engine can serve both tracks.

## Current status

- [x] Layer 0 - synthetic-trace CPA engine
- [ ] Layer 1 - behavioral RTL (power)
- [ ] Layer 2 - gate-level glitch
- [ ] Parallel Layer 1 - Python cache model
- [ ] Layer 4 - FPGA measurement

## Layer 0: synthetic-trace CPA (done)

Before trusting any real measurements, the attack code must be proven correct using data where I already know the answer.

I target the output of the AES-128 first-round S-box using a Hamming-weight leakage model. I generate the traces myself using a known key, and then attack them as if the key were unknown.

Leakage model: L = HW(Sbox(plaintext XOR key)) + Gaussian noise

The attack makes 256 key guesses and selects the one whose hypothesis shows the highest correlation with the trace.

**What this layer answers**
- Does my CPA implementation recover the correct key?
- How many traces are needed, and does this number match theoretical expectations?
- What happens when the leakage model is incorrect (e.g., using Hamming Distance instead of Hamming Weight, or targeting the S-box input instead of the output)? This is an intentional failure test.

**Results**

<!-- TODO: add after experiments -->
- Correlation vs 256 key guesses - [plot]
- Traces needed vs noise level - [plot]

## Repository layout

<!-- TODO: fill in as it grows -->

security/
  00-synthetic/   Layer 0 CPA engine and notes
  ...

## How to run

<!-- TODO: add make target / environment once code lands -->

## Notes format

Each experiment is documented in five lines:
Question / Hypothesis / Setup / Result / Reading.

## References

- Mangard, Oswald, Popp - *Power Analysis Attacks* (standard textbook)
- Kocher et al. - Differential Power Analysis (1999)
- Brier et al. - Correlation Power Analysis (2004)
- ChipWhisperer tutorials - de-facto reference for trace format and CPA
