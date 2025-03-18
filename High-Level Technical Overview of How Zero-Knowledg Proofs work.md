# High-Level Technical Overview of How Zero-Knowledge Proofs Work

*Special thanks to Gink and Mirror from Zerobase for reviewing and giving feedback. Special thanks to Ying Tong, Flying Nobita and Bing for the workshops, and my gratitudes to PSE for organising zk workshop series.*

In February, I had the privilege of attending an incredible Zero-Knowledge (ZK) workshop series organized by PSE in Singapore. As a ZK enthusiast, it was a fantastic opportunity to deepen my understanding of cryptographic protocols like **Groth16**, **Plonk**, and **STARKs**. These protocols were the main focus of the workshop, and this article reflects what I’ve learned. It’s a high-level technical dive into how zero-knowledge proofs (ZKPs) function and how these specific protocols differ, written in a way that’s hopefully clear and engaging for fellow enthusiasts. Or even with less technical background you should be able to understand

---

## What Are Zero-Knowledge Proofs?

Lets start from what is ZK and what it is supposed to work. Zero-knowledge cryptography is a cryptographic technique heavily discussed in 80s but took of in recent years because it found a perfect use case in blockchain industry and good support for the research and its application.

Zero-knowledge proofs (ZKPs) are cryptographic techniques that let one party (the **prover**, let’s call her Alice) convince another party (the **verifier**, let’s call him Bob) that a statement is true without revealing any details beyond the fact itself. Imagine Alice proving she knows a secret password to Bob without ever telling him the password—ZKPs make this possible.

### Key Properties of ZKPs

ZKPs rely on three essential properties to work their magic:

- **Completeness**: If Alice’s statement is true and she follows the protocol honestly, Bob will always accept her proof as valid.
- **Soundness**: If Alice’s statement is false, she can’t trick Bob into accepting it (except with an extremely tiny chance, like winning the lottery a million times in a row).
- **Zero-Knowledge**: Bob learns nothing about Alice’s secret beyond the fact that her statement is true.

These properties ensure ZKPs are secure, private, and reliable.

---

## General Framework of ZKPs

While there are many ZK protocols, most follow a common structure. I picked this up from Ying Tong, an amazing ZK researcher at the workshop, who explained it as a four-step process:

<aside>
📎

> Problem → Arithmetization → Polynomials → Commitment Schemes
> 
</aside>

Not every protocol fits this perfectly, but it’s a great way to understand how ZKPs turn real-world problems into cryptographic proofs. Let’s break it down step by step.

Lets start with widely used protocol called groth16. Groth16 is a zkSnark protocol, and is very famous and widely used for generating small size proofs and hence fast verification time. There are other widely discussed protocols as well. They are PLONK and STARK. Each of them can be another big countless articles, but we will focusing on understanding how they work in simpler terms.

### Step 1: Problem

Every ZKP starts with a statement or problem to prove. For example:

- "I know the solution to this Sudoku puzzle."
- "I know a secret number s such that $s^2=25.$"
- or it can be “I have more than 1ETH in my wallet, and I am eligible for the loan”

The goal is to prove knowledge of the solution without revealing it.

### Step 2: Arithmetization

Artihmetastion is a process of turning a real life problems intro an arithmetic circuit, or we can say arithmetic equation. We have to create step by step constraints that solves one big problem if all of them are fulfilled. Then we will represent each constraint as polynomial, so that we can do operations on polynomials. Arithmetization can be in different forms based on the protocol, for example groth16 turns into R1CS constraints and PLONK uses AIR.

<aside>
📎

>Here is a note why do we need arithmetization. Zero-knowledge (ZK) cryptography works by using algebraic equations, polynomials, and constraint systems to prove statements without revealing data. To use it in real life, we need to turn the actual problem into a mathematical problem, and that’s where we use a process called arithmetization.
> 
>This means expressing the computation as arithmetic circuits over finite fields, which general programming languages like Python or C++ often struggle with. And we use domain specific languages to write circuits.

</aside>

Different protocols use different flavors of arithmetization:

- **Groth16**: Uses **R1CS (Rank-1 Constraint System)**, breaking the problem into constraints like $A⋅B=C$.
- **Plonk**: Uses a custom system with **selectors and wiring constraints**, offering more flexibility.
- **STARKs**: Uses **AIR (Arithmetic Intermediate Representation)**, modeling the problem as state transitions (like a virtual machine).

### Step 3: Polynomials

Once the problem is arithmetised into constraints, the next step is to represent these constraints as polynomials. Using techniques like Lagrange interpolation, we can construct polynomials that “encode” the constraints. These polynomials are advantageous because they can be manipulated algebraically and checked for identities.

To convince the verifier, the prover and verifier use an interactive method:

1. **Prover Commits**: The prover “locks” the polynomial $P(x)$ using a commitment scheme (more on this later) and sends the commitment to the verifier.
2. **Verifier Challenges**: The verifier picks a random point, say $r=7$, and asks, “What’s $P(7)?$”
3. **Prover Responds**: The prover computes $P(7)=7^2−25=49−25=24$ and sends it back.
4. **Verifier Verifies**: The verifier checks if the response fits the expected properties of $P(x)$, using additional proof steps (we’ll see how with commitments).

Many ZKP systems use an IOP framework where the prover provides evaluations of these polynomials in response to random challenges from the verifier. The interaction ensures that if even one constraint is not satisfied, the chance of cheating successfully becomes negligibly small. In simple terms, if we repeat challenging the prover multiple times, we can be sure that he is not lying.

However, In real-world ZKPs (like on blockchains), we don’t want back-and-forth interaction. The **Fiat-Shamir transform** solves this by replacing the verifier’s random challenges with a hash of the prover’s earlier messages. The prover computes everything upfront, creating a single proof the verifier can check.

### Step 4: Commitment Schemes

A commitment scheme is like sealing your secret in an envelope. The prover "commits" to the polynomial (locks it in) without revealing it, then later proves properties about it to the verifier.

**Why it’s needed**: It ensures the prover can’t change their secret mid-proof and lets the verifier check the proof without seeing the secret. We discussed it a bit earlier in the step 3.

**Examples**:

- **KZG (Kate-Zaverucha-Goldberg)**: Used in Groth16 and Plonk, based on elliptic curve pairings.
- **FRI (Fast Reed-Solomon Interactive)**: Used in STARKs, based on error-correcting codes.

---

## Comparison Table

Here’s a refined comparison of the three protocols:

| **Aspect** | **Groth16** | **STARKs** | **Plonk** |
| --- | --- | --- | --- |
| **Arithmetization** | R1CS | AIR (state transitions) | Custom (selectors, wiring) |
| **IOP** | Schwartz-Zippel polynomial checks | Schwartz-Zippel polynomial checks | Schwartz-Zippel + permutation args |
| **Commitment Scheme** | KZG (pairing-based) | FRI (Reed-Solomon codes) | KZG (pairing-based) |
| **Trusted Setup** | Per-circuit | None (transparent) | Universal (reusable) |
| **Proof Size** | Very small (~200 bytes) | Large (~100s of KB) | Small (~300-500 bytes) |
| **Verification Time** | Fast | Slower | Fast |
| **Quantum Resistance** | No | Yes | No |

In a nutshell, this is a bigger picture of how ZKPs work. There might be a lot of other new protocols, but they follow more or less the similar structure. The details might be different in each step, such as choosing what type of commitment schemes are used or how it is arithmetized.  Here are some extra info i think it might be useful. Other articles will cover more on each of the protocols.

<aside>
📎

>**Circuit**
>A circuit in zk is a mathematical representation of a computation using basic arithmetic gates (addition, multiplication, etc.).  In my opinion It's called a circuit because it's conceptually similar to an electronic circuit, where:
> 
> - **Inputs** go into **gates** (like AND, OR in logic circuits).
> - Each **gate** performs an operation.
> - The final **output** depends on the structured flow of operations.
</aside>

<aside>
📎

> **Why Finite Field**
>
> Zero-knowledge cryptography works in **finite fields** for mathematical efficiency and security. You might think **Why Not Regular Numbers?** There are a few reasons:
>
> - Regular numbers grow too large for cryptographic operations.
> - Floating-point numbers cause precision issues.
> - Bitwise operations (used in CPU computation) do not map cleanly to algebraic circuits.

> And here is  **Why Finite Fields Work?**
>
> - **Modulo a prime** keeps numbers within a fixed size.
> - Arithmetic remains well-defined.
> - Operations become efficient when expressed as **modular polynomials**.
</aside>
