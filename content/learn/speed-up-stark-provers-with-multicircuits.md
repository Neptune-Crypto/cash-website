+++
title = "Speeding up STARK Provers with Multicircuits"
date = "2022-11-25"
weight = 0

[extra]
author = "Thorkil Værge"
+++

A significant challenge when building STARK provers is the computational overhead of generating a proof of correct
execution of a program. How do we get it fast enough?

For a program of $N$ instructions the asymptotic runtime for generating a proof is $O(N\cdot log(N))$ but this hides the fact that the involved constants can be very big. The asymptotic limit comes from executing number-theoretic transform (NTT) over a large domain, where NTT
is the finite-field version of FFT, fast Fourier transform. But this was not where the very big constants came from for the [STARK VM we are building, Triton VM](https://triton-vm.org/). Instead, the big constants came from evaluating multivariate polynomials over a big domain.

## STARKs and Multivariate Polynomials

STARKs achieve their guarantees of computational integrity through constraints on allowed register values in the virtual machine.[^tutorials] These constraints are expressed
as multivariate polynomials over the register values over pairs of rows in the execution trace. The simplest example of such a constraint is the rule
that the clock cycle register must increase by one for each cycle: $clk_{n+1} - clk_n - 1 = 0$. In this example the clock cycle counter is a register in the virtual machine. If the clock cycle register is considered the first register of the virtual machine, this constraint can be expressed in terms of the multivariate polynomial $f(x_{0,n}, x_{1,n}, ..., x_{N-1,n}, x_{0,n+1}, x_{1,n+1}, ..., x_{N-1,n+1}) = x_{0,n+1} - x_{0,n} - 1$ where $N$ is the number of registers in the VM.

More complex constraint polynomials come into play when the constraints of an instruction are defined. The constraint for an add instruction on a [stack machine](https://en.wikipedia.org/wiki/Stack_machine) is
$st0_{n+1} = st0_n - st1_n \Rightarrow st0_{n+1} - st0_n - st1_n = 0$ meaning that the next value of the $st0$ register must be the sum of the previous value of the $st0$ and $st1$ registers. But this constraint can
obviously only be active when the add instruction is executed and this qualification is achieved by deselector polynomials of the form $\prod_{op' \neq add}\left(ci - a_{add}\right)$
where $ci$ is the current register and $a_{add}$ is the [opcode](https://en.wikipedia.org/wiki/Opcode) for the add instruction. The deselector polynomial for add is zero for all other instructions than add. These deselector polynomials are multiplied with the instruction-specific constraints.

The details of the constraint polynomials are not important for this blogpost though. Suffice it to say that the constraint polynomials can reach a high complexity, or degree if you will[^1], that makes computing them slow.

An integral part of the STARK proof is to evaluate the multivariate polynomials over a domain that's significantly bigger than the number of cycles executed in the program.

[^tutorials]: To learn how zk-STARKs work, take a look at our tutorials: [Anatomy of a STARK](/learn/stark-anatomy/) and [BrainSTARK](/learn/brainfuck-tutorial/).

## Naive Evaluation

A multivariate polynomial can be expressed as a hash map with an array of integers as keys and finite field elements as values. Then by looping over all key/value pairs, a
multivariate polynomial can be evaluated. The Rust code for this evaluation could look like this:
```rust
    pub fn evaluate(&self, point: &[FF]) -> FF {
        let mut acc = FF::zero();
        for (k, v) in self.coefficients.iter() {
            let mut prod = FF::one();
            for i in 0..k.len() {
                // If the exponent is zero, multiplying with this factor is the identity operator.
                if k[i] == 0 {
                    continue;
                }
                prod *= point[i].mod_pow_u32(k[i] as u32);
            }
            prod *= *v;
            acc += prod;
        }

        return acc;
    }
```
**Example 1:** *Evaluating a multivariate polynomial with generic code.*

If you're evaluating exactly one multivariate polynomial and you don't know anything about it at compile time, this is probably the best you can do.
But what if
1. The multivariate polynomials are known at compile time?
2. There are many multivariate polynomials that share common factors?

What's the best we can do in that case?

## Known Multivariate Polynomial at Compile-time
If you know the multivariate polynomial at compile time you can simply replace the data (multivariate polynomial) with code.
Evaluating the polynomials $f(x,y,z) = x(yx^2 + 2z), g(x,y,z) = yx^2 + 2z + 12$ could then be written as

```rust
 pub fn evaluate_all_constraints(&self, point: &[FF]) -> Vec<FF> {
   return vec![
     point[0] * ( point[1] * point[0] * point[0] + 2 * point[2]),
     point[1] * point[0] * point[0] + 2 * point[2] + 12,
   ];
 }
```
**Example 2:** *Evaluating a multivariate polynomial with hardcoded coefficients and exponents. Flat structure.*

This speeds up the code in three ways:
- No cycles are spent going over all indices in the key vectors of the multivariate polynomial. So adding an extra register (increasing $N$) will not slow down the evaluation.
- There are no branches, so no time is wasted checking if the outer loop or inner loop from Example 1 is completed or if any of the coefficients are zero.
- No time is spent pulling out coefficient or exponent values from a hash map, as the values live directly in code.

## Reusing Shared Nodes
Consider the constraint associated with the instructions for addition and subtraction. These both involve deselectors of the form
$\prod_{op' \neq op}\left(ci - a_{op}\right)$, so they have
a common factor of $\prod_{op' \not\in \\{add, sub\\} }\left(ci - a_{op}\right)$. To express these shared factors we need a structure
that's similar to a [binary tree](https://en.wikipedia.org/wiki/Binary_tree) but where multiple roots can share a node. This structure is similar to a [circuit](https://en.wikipedia.org/wiki/Circuit_(computer_science)) but it has multiple outputs, or multiple roots if you will. Let's call it a multicircuit.

The reduction in complexity with this technique is up to the number of roots in your multicircuit. Each root represents a constraint and Triton VM's Processor Table has [74 transition constraints](https://github.com/TritonVM/triton-vm/blob/master/triton-vm/src/table/processor_table.rs#L1031), so the speedup potential is significant for some of these calculations.

This code can be auto-generated by counting the number of times that each node in the multicircuit is visited by starting at all roots and then incrementing the counter each time a node of a subtree is reached by visiting children of each encountered node. Once this has been counted, the code can be auto-generated by again visiting all nodes in the tree and adding a new declaration each time the counter for a child node is higher than the parent node.

![0](/learn/multi-circuits/multi-circuit.svg)
**Figure 1:** *If the multicircuit is evaluated bottom-up, any shared nodes will be available before the final constraint evaluation takes place. In this example the "+" in the middle is used by both expressions $f(x,y,z) = x(yx^2 + 2z), g(x,y,z) = yx^2 + 2z + 12$.*

When using a multicircuit, the above calculation of $f(x,y,z) = x(yx^2 + 2z), g(x,y,z) = yx^2 + 2z + 12$ becomes

```rust
 pub fn evaluate_all_constraints(&self, point: &[FF]) -> Vec<FF> {
   let node_0 = point[1] * point[0] * point[0] + 2 * point[2];
   return vec![
     node_0 * point[0],
     node_0 + 12,
   ];
 }
```
**Example 3:** *Evaluating a multivariate polynomial with hardcoded coefficients and exponents. Multicircuit structure.*

## Observed Speedup

### Generic code

```
### prove halt                                                  14.82s
├─
  ...
  ├─LDE 1                                                       8.09ms
  ├─Merkle tree 1                                               56.91ms
  ...
  ├─quotient codewords                                          12.03s    81.17%
    ...
  │ ├─ProcessorTable with lifted matrix with data with data     11.84s    79.92%
    ..
  │ │ ├─transition quotients                                    11.73s    79.15%
```

The constraint evaluation takes 12.03 seconds for a very simple program! A recursive proof will involve thousands of instructions, so clearly a speedup is needed.

### Hardcoding Multivariate Polynomials and Reusing Shared Nodes

```
### prove halt                                                  3.03s
├─
  ...
  ├─LDE 1                                                       7.58ms
  ├─Merkle tree 1                                               60.08ms
  ...
  ├─quotient codewords                                          283.50ms
    ...
  │ ├─ProcessorTable with lifted matrix with data with data     119.28ms
    ..
  │ │ ├─transition quotients                                    6.55ms
```

Only the transition constraints have been rewritten with auto-generated multicircuit-based code for this benchmark. And this shows a 1790 times speedup on a Core i9-11950H, and a 2000 times speedup on a slower Dell XPS with 16GB RAM.

This is just one of the *required* algorithmical tricks that we need to exploit to make STARKs provers feasible. This has eliminated constraint evaluation as a bottleneck for now and revealed the next bottleneck in our prover: [interpolation](https://github.com/Neptune-Crypto/twenty-first/blob/2738baabc69134274eb250c207e0236ce4b0ecc8/twenty-first/src/shared_math/polynomial.rs#L402).

### Source Code
Check out the [data structure that was used to represent the multicircuits](https://github.com/TritonVM/triton-vm/blob/master/triton-vm/src/table/constraint_circuit.rs) and the [compiler that outputs (what I think is the) optimal code for evaluating multiple polynomials](https://github.com/TritonVM/triton-vm/blob/471b56f498d77cf05aaf46e0290b4621cf3535eb/constraint-evaluation-generator/src/main.rs).

### Acknowledgements

Thanks to Ferdinand Sauer and Alan Szepieniec for feedback and comments.

[^1]: The degree of a naive implementation of deselector polynomials grows linearly with the number of instructions. But by adding more registers for decoding the instruction, one can lower
this growth to the logarithm of the number of instructions. The degree of a polynomial is defined as the highest number of times that input variables are multiplied in a term in the multivariate polynomial. So $f(x,y) = xy$ has degree two, and $f(x,y) = x^4y + xy - 32x^3$ has degree five.
