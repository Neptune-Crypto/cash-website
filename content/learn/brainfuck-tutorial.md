+++
title = "Brainfuck STARK Tutorial"
date = "2022-08-03"
weight = 0

[extra]
author = "Thorkil Værge, Ferdinand Sauer, and Alan Szepieniec"
+++

In this tutorial we show how a simple, Turing complete programming language can be arithmetized such that a STARK prover and verifier can be built for it.
With this construction, the correct execution of a program can be verified exponentially faster than it would take to re-run the program.

An exponential speedup of the verification of the correct evaluation of a program opens up a new era of computing, with applications in cloud computing, blockchain, security-critical systems, and much more.

The Turing complete programming language we have chosen for this tutorial is [Brainfuck](https://en.wikipedia.org/wiki/Brainfuck).
It is a simple language, containing only eight instructions.
They are used to manipulate values in an array, and to read to and write from this array.
The language's simplicity makes it a prime candidate for arithmetization.
It also means that programming in Brainfuck is not very ergonomic.
That's part of the reason why we are currently designing and building our own Virtual Machine, [TritonVM](https://github.com/TritonVM/triton-vm).

## Motivation and Prerequisites

For this tutorial, a certain familiarity with polynomial arithmetic and finite field algebra is required to fully grasp the complex machinery of STARKs.
In fact, it is very beneficial to already know what STARKs are and roughly how they work.
If you need or want to refresh your memory on this subject, we recommend the excellent “[Anatomy of a STARK](https://neptune.cash/learn/stark-anatomy/).”
Other good resources also focus on STARKs for more constraint calculations, for example on the [Collatz Sequence](https://medium.com/starkware/arithmetization-i-15c046390862) or the hash function [MIMC](https://vitalik.ca/general/2017/11/09/starks_part_1.html).
That being said, we have tried to make this tutorial as intuitive as possible.

To that end, this tutorial purposefully ignores some details.
For example, for fast calculations to be possible for both prover and verifier, the execution trace (and the associated [tables](#tables)) must be padded to have a length of $2^k$.
In this tutorial, we don't do that, sacrificing prover and verifier speed for conceptual ease.
We also simplify the example calculations by presenting them as if they happen in the base field instead of the extension field.
Using only the base field decreases security, but it is easier to grasp the involved concepts.

## Outline of Arithmetization

Arithmetization of a virtual machine refers to expressing the state transition for the execution of each instruction as a polynomial. This implies that each new register value can be expressed in terms of previous register values using only addition, subtraction, multiplication, and division. 

If we for example have a register that we call `clk` that counts the number of instructions that have been executed so far, then the value in this register can be expressed in terms of its value in the previous cycle as $clk\_{n+1} = clk_n + 1$. From this expression we can derive a constraint that must always hold if a list of instructions have been executed honestly:

$$clk\_{n+1} - clk_n - 1 = 0$$

The left-hand side of this expression can be thought of as a polynomial where the variables are $\bar{r}_n$ and $\bar{r}\_{n+1}$, the register values in the current and the next step of the execution, respectively.

$$P(\bar{r}_n, \bar{r}\_{n+1}) = clk\_{n+1} - clk_n - 1$$

In this expression it's implicitly understood that this polynomial must evaluate to zero in a correct execution.

$$P(\bar{r}_n, \bar{r}\_{n+1}) = 0$$

If we can identify all such **constraint polynomials** for this virtual machine, then we can use these polynomials to prove and verify the correct execution of a Brainfuck program.

One of the main benefits of this technique is that the verification runs faster than it takes to re-run the program. The correct execution of a program can be verified in $O(log^2(N))$ time, where $N$ is the number of cycles that the program ran before termination. This verification time constitutes an asymptotic speedup from the regular way of verifying an execution, which is to re-run the program. Rerunning the program takes $O(N)$ time.

The goal of this tutorial is to show how constraint polynomials can be architected such that they are compatible with a brainfuck virtual machine, not just a simple counter.

## The Brainfuck Programming Language

Brainfuck is a small language with only eight instructions.
Its central piece is an infinitely long array of memory cells, each holding some value.
All cells are initialized to 0.
A single memory pointer references one element in memory.

![](bf.svg)

With this picture about Brainfuck's memory model in mind, it should be pretty straightforward to understand all instructions.

#### Instruction set
| brainfuck | C equivalent       | opcode (ascii) | opcode alias   | description             |
| :-------- | :----------------- | :------------- | :------------- | :---------------------- |
| `<`       | `i--;`             | 60             | $a_<$          | Decrease memory pointer |
| `>`       | `i++;`             | 62             | $a_>$          | Increase memory pointer |
| `+`       | `arr[i]++;`        | 43             | $a_+$          | Increase memory value   |
| `-`       | `arr[i]--;`        | 45             | $a_-$          | Decrease memory value   |
| `[`       | `while(arr[i]) {`  | 91             | $a_\texttt{[}$ | If/while start          |
| `]`       | `}`                | 93             | $a_\texttt{]}$ | If/while end            |
| `,`       | `arr[i] = getc();` | 44             | $a_,$          | Read from input         |
| `.`       | `putc(arr[i]);`    | 46             | $a_.$          | Write to output         |

We set $\text{IS} = \\{\texttt{<}, \texttt{>}, \texttt{-}, \texttt{+}, \texttt{[}, \texttt{]}, \texttt{,}, \texttt{.}\\}$ to be the set of all instructions for more convenient notation below.

#### On Brainfuck Dialects
The STARK framework relies on polynomials that are defined over a finite field.
This means that all calculations that you see here are modulus a specific prime number.
For our implementation we use the prime $2^{64} - 2^{32} + 1$.
So the dialect of Brainfuck in this tutorial uses cells – the elements of the memory array – that have values within the interval $[0, p - 1]$.
Some Brainfuck implementations use `u8` cell values where the values are elements of the interval $[0, 255]$.
If you have a program that relies on overflow/wrap around of these values, the dialect presented here will behave differently than expected.[^wikipediaonbfdialects]

#### Compiling Brainfuck
In our implementation, the `[` (respectively `]`) instruction of a compiled program is directly followed with the location of the end (respectively beginning) of the loop. For example, `+[>+<-]+` compiles to $a_+\ a_\texttt{[}\ 9\ a_>\ a_+\ a_<\ a_-\ a_\texttt{]}\ 3\ a_+$

| 0     | 1              | 2    | 3     | 4     | 5     | 6     | 7              | 8    | 9     | 10   |
| :---- | :------------- | :--- | :---- | :---- | :---- | :---- | :------------- | :--- | :---- | :--- |
| $a_+$ | $a_\texttt{[}$ | 9    | $a_>$ | $a_+$ | $a_<$ | $a_-$ | $a_\texttt{]}$ | 3    | $a_+$ |

Expressed in integers, the fully compiled program is `[43, 91, 9, 62, 43, 60, 45, 93, 3, 43]`.

##### Example Program
Throughout this tutorial, we will illustrate various constructions using the program `++>,<[>+.<-]`. It reads a character from standard in (something that the user inputs with their keyboard) and prints the to following ASCII characters. So if `a` is input, the program will output `bc`.

The example program
```
++>,<[>+.<-]
```
compiles to

$$[a_+\ a_+\ a_> a_,\ a_<\ a_\texttt{[}\ 14\ a_>\ a_+\ a_.\ a_<\ a_-\ a_\texttt{]}\ 7]$$

Where `14` and `7` represent jump targets for `[`, respectively `]`.

And it leaves the memory array as
```
[0, <user input> + 2, 0, 0, 0, ...]
```


#### Registers
The virtual machine that we have built for Brainfuck contains these registers:

| id    | name                 |
| :---- | :------------------- |
| `clk` | cycle                |
| `ip`  | instruction pointer  |
| `ci`  | current instruction  |
| `ni`  | next instruction     |
| `mp`  | memory pointer       |
| `mv`  | memory value         |
| `mvi` | memory value inverse |

The value in the `mvi` register is 0 if and only if `mv` is 0. Otherwise, `mvi` has the inverse value of `mv`.

The registers contain the state before the instruction in `ci` has been executed.

If you've ever written a Brainfuck virtual machine you will probably think that this implementation has too many registers. Don't we just need an instruction pointer, a memory pointer, and the memory array? To execute the program, yes. But to create a proof of correct execution we need a few more registers that will be used to express the constraint polynomials. [^1]

##### Example Program

Running the example program `++>,<[>+.<-]` until end yields the following register values for each cycle. In this example the user inputs `a` which has ASCII value 97. This is the value held in the `mv` register in cycle $4$.

| clk  | ip   | ci             | ni             | mp   | mv   | mvi       |
| :--- | :--- | :------------- | :------------- | :--- | :--- | :-------- |
| $0$  | $0$  | $a_+$          | $a_+$          | $0$  | $0$  | $0$       |
| $1$  | $1$  | $a_+$          | $a_>$          | $0$  | $1$  | $1$       |
| $2$  | $2$  | $a_>$          | $a_,$          | $0$  | $2$  | $2^{-1}$  |
| $3$  | $3$  | $a_,$          | $a_<$          | $1$  | $0$  | $0$       |
| $4$  | $4$  | $a_<$          | $a_\texttt{[}$ | $1$  | $97$ | $97^{-1}$ |
| $5$  | $5$  | $a_\texttt{[}$ | $13$           | $0$  | $2$  | $2^{-1}$  |
| $6$  | $7$  | $a_>$          | $a_+$          | $0$  | $2$  | $2^{-1}$  |
| $7$  | $8$  | $a_+$          | $a_.$          | $1$  | $97$ | $97^{-1}$ |
| $8$  | $9$  | $a_.$          | $a_<$          | $1$  | $98$ | $98^{-1}$ |
| $9$  | $10$ | $a_<$          | $a_-$          | $1$  | $98$ | $98^{-1}$ |
| $10$ | $11$ | $a_-$          | $a_\texttt{]}$ | $0$  | $2$  | $2^{-1}$  |
| $11$ | $12$ | $a_\texttt{]}$ | $7$            | $0$  | $1$  | $1$       |
| $12$ | $7$  | $a_>$          | $a_+$          | $0$  | $1$  | $1$       |
| $13$ | $8$  | $a_+$          | $a_.$          | $1$  | $98$ | $98^{-1}$ |
| $14$ | $9$  | $a_.$          | $a_<$          | $1$  | $99$ | $99^{-1}$ |
| $15$ | $10$ | $a_<$          | $a_-$          | $1$  | $99$ | $99^{-1}$ |
| $16$ | $11$ | $a_-$          | $a_\texttt{]}$ | $0$  | $1$  | $1$       |
| $17$ | $12$ | $a_\texttt{]}$ | $7$            | $0$  | $0$  | $0$       |
| $18$ | $14$ | $0$            | $0$            | $0$  | $0$  | $0$       |

The above table which contains the register values for each clock cycle is commonly referred to as the **execution trace**.

The program terminates when the instruction pointer (`ip`) points beyond the length of the program.

### Tables
The execution trace, presented for the example program above, plays a central role in the construction of the STARK proof.

In the previous tutorial, the honest calculation of a hash digest could be proved using only polynomial arithmetic on the execution trace, like the above table. In STARK engine for Brainfuck, we need more than that.

The execution trace defined above is the 1$^{\text{st}}$ of our so-called tables. In this context, the execution trace is called the **Processor Table**. It contains the values of all the registers and it is sorted by column “cycle.”

With this in mind, we can introduce a **Memory Table** which is similar to the processor table (execution trace) as it is the state of registers but sorted by the memory pointer. So the execution trace and the memory table have the same length, and the rows in each table is a permutation of the other table's rows.

The link between the program's execution and the actual program is the **Instruction Table**.
It is constructed by concatenating the program, i.e., the list of instructions, with the execution trace.
The resulting list is then sorted by instruction pointer first, and cycle second.

Finally, there are the **Input Table** and the **Output Table**. They are formed from the subset of rows in the execution trace which read from input or write to output, respectively.
Concretely, for every executed `,`, there is one row in the Input Table.
Likewise, for every `.`, there is one row in the Output Table.

To sum up, we have five tables: processor table, instruction table, memory table, input table, and output table.

![](aet.svg)

Each table has its own set of transition and boundary constraints that guarantees a specific property of the execution. These constraints are expressed in the form of constraint polynomials, similar to the ones introduced above.

Each table serves the purpose of proving a specific quality of the execution:
1) The processor table proves that each instruction transforms the state as defined in the VM.
2) The memory table proves that memory values are consistent, for example that all memory values are initialized to zero and that when a memory value is accessed again, it has not changed since the last cycle in which this memory value was accessed.
3) The instruction table guarantees that the expected program is read into the instruction registers
4) The input table proves that the correct values are read into memory.
5) The output table proves that the program writes the correct values to output.

### From Tables via Polynomials to Codewords

We briefly recall how in a STARK, an execution trace is turned into a codeword representing a low-degree polynomial.
The same technique is used for all of the five tables defined above, not just the Processor Table, which is the execution trace in our STARK engine.

First, the values in the columns of the tables are interpolated, resulting in a bunch of polynomials.
For interpolation, we use the so-called $\omicron$-domain:
$\\{ \omicron^i \\}\_{i=0}^{N-1}$.
The value of $N$ is the length of the longest table – the Instruction Table.
The value of $\omicron \in \mathbb{F}_p$ depends on $N$;
namely, $\omicron$ must generate a subgroup of $\mathbb{F}_p^\star$ with cardinality at least $N$.
Each interpolation polynomial $ti_c(x)$ represents one column $c$.

Second, the interpolation polynomials $ti_c(x)$ are evaluated on a domain that's larger than $\\{\omicron^i\\}\_{i=0}^{N-1}$.
In our case, we use the so-called $\Omega$-domain:
$\\{\Omega^i\\}\_{i=0}^{4N-1}$.
As before, $\Omega$ depends on $N$, and must generate a subgroup of appropriate cardinality.

The two-step of interpolation over the $\omicron$-domain and evaluation on the $\Omega$-domain is called _low degree extension_.
Its result is one codeword per column from the table we started with.

Third, the AIR polynomials are evaluated on the low-degree-extended codewords.
This gives rise to one transition codeword per AIR polynomial.

Fourth, each transition codeword is divided by the zerofier codeword, resulting in a set of quotient codewords.

Only if the prover is honest does each quotient codeword correspond to a polynomial of low degree.
The low-degreeness of the quotient codewords is then proved using the FRI protocol.

## The Constraints
Now that we have defined Brainfuck, execution traces, tables, and polynomial arithmetic, we are ready to look at which constraints that apply for this virtual machine and its tables. These constraints allow us to calculate both the transition polynomials and transition quotients defined above. If all transition quotients have a sufficiently low degree, then we have proven that all constraints are satisfied and thus that the computation is integral.

The constraints that apply for a STARK engine can be categorized into boundary constraints, consistency constraints, transition constraints, and terminal constraints. These will be covered and defined below.

### Boundary Constraints
A part of the conditions that must be proven in the STARK proof is that the values of the registers are valid at the beginning of program execution. Boundary constraints ensure that the registers are initialized correctly.

| id    | description                                | polynomial                       | note |
| :---- | :----------------------------------------- | :------------------------------- | :--- |
| $B_0$ | cycle is initially 0                       | $clk$                            |
| $B_1$ | instruction pointer is initially 0         | $ip$                             |
| $B_2$ | current instruction is a valid instruction | $\prod\_{op \in IS}(ci - a\_{op})$ | \*   |
| $B_3$ | memory pointer is initially 0              | $mp$                             |
| $B_4$ | memory value is initially 0                | $mv$                             |
| $B_5$ | memory value inverse is initially 0        | $mvi$                            | \*   |

The constraints marked with an asterisk (\*) are unnecessary since they are also ensured by the consistency constraints.

There is no initial constraint on register “next instruction.” This is due to how looping works: if the very first instruction is `[`, then “next instruction” is the location of the corresponding `]`, which is at an arbitrary location (albeit fixed, given a concrete program).


### Consistency Constraints
Consistency constraints are constraint polynomials that must apply to all rows in the tables and that involve only *one* row. In other words, it binds values of one register to values in other registers in each cycle. Formally, a consistency constraint is $P(ti(x))$, where $ti(x)$ is a trace interpolant, the interpolation of all values in a column.

| id    | description                                                                      | polynomial                       | note |
| :---- | :------------------------------------------------------------------------------- | :------------------------------- | :--- |
| $C_0$ | memory_value is 0 or memory_value_inverse is the inverse of memory_value         | $mv\cdot(mv \cdot mvi - 1)$      |
| $C_1$ | memory_value_inverse is 0 or memory_value_inverse is the inverse of memory_value | $mvi\cdot(mv \cdot mvi - 1)$     |
| $C_2$ | current instruction is a valid instruction                                       | $\prod\_{op \in IS}(ci - a\_{op})$ | \*   |

The last consistency constraint, $C_2$ is handled by the transition constrainst $T1-T3$, so we don't need a separate constraint for this.

Using the fact that 0 has no multiplicative inverse, $C_0$ and $C_1$ allow the expression $mv\cdot mvi - 1$ to be interpreted as `is_zero`. Concretely, if both $C_0$ and $C_1$ hold, we have

$$mv\cdot mvi - 1 = \begin{cases} 1, \text{ if }\texttt{memory_value}=0 \\ 0, \text{ else}. \end{cases}$$

In Brainfuck we need a way of checking if $mv = 0$ because this is how we determine if a jump should be taken or not. The expression $(mv\cdot mvi - 1)$ allows for exactly this since this combination is non-zero if and only if $mv = 0$.

### Transition Constraints
A transition constraint is a constraint polynomial that involves two consecutive rows (like two consecutive states of a virtual machine), such that the next state of the virtual machine can be linked to the current state of the virtual machine. Formally a transition constraint has the form $P(ti(x)), ti(\omicron\cdot x))$ where $\omicron \cdot x$ is the $x$ value for the next row in the table; $\omicron$ (omicron) is the value with which $x$ must be multiplied to get the next row in a table.


#### Processor Table
The processor table ensures the consistency for the part of the execution that relates to the registers of the virtual machine. The processor table records all the register values for each cycle that the program ran.

| id    | description                                  | polynomial                                                                                                         |
| :---- | :------------------------------------------- | :----------------------------------------------------------------------------------------------------------------- |
| $P_0$ | cycle increases by one per step              | $clk\_{n+1} - clk_n - 1$                                                                                          |
| $P_1$ | instruction mutates state correctly (part 1) | $\sum\_{op\in IS}\left( instr^{(1)}\_{op}(\vec{r}_n, \vec{r}\_{n+1}) \cdot \prod\_{op' \neq op}(ci - a\_{op'}) \right)$ |
| $P_2$ | instruction mutates state correctly (part 2) | $\sum\_{op\in IS}\left( instr^{(2)}\_{op}(\vec{r}_n, \vec{r}\_{n+1}) \cdot \prod\_{op' \neq op}(ci - a\_{op'}) \right)$ |
| $P_3$ | instruction mutates state correctly (part 3) | $\sum\_{op\in IS}\left( instr^{(3)}\_{op}(\vec{r}_n, \vec{r}\_{n+1}) \cdot \prod\_{op' \neq op}(ci - a\_{op'}) \right)$ |

There are a few things to unpack for $P_1$ through $P_3$. First up: $\vec{r}_n$ is the vector of all registers in cycle $n$, i.e., $\vec{r}_n=(clk_n, ip_n, ci_n, ni_n, mp_n, mv_n, mvi_n)$.

Next, the components of $P_1$ through $P_3$ are as follows.
1. $\prod\_{op' \neq op}(ci - a\_{op'})$ This polynomial, called the _deselector_ for operation $op$, evaluates to zero at any opcode that is not $op$.

2. $instr^{(i)}\_{op}(\vec{r}_n, \vec{r}\_{n+1})$ This polynomial is a part of a constraint modeling the transition for only a single instruction, namely $op$. Brainfuck has 8 instructions, and each instruction has its own $instr^{(i)}\_{op}$ for $ 1 \leq i \leq 3 $. Any partial instruction polynomials will evaluate to 0 if evaluated on $(\vec{r}_n, \vec{r}\_{n+1})$ such that $\vec{r}_n$ corresponds to instruction $op$. Only when seen together do the $instr^{(i)}\_{op}$ describe the “full” instruction polynomial.

In the following, all polynomials $instr^{(i)}\_{op}$ are listed.

Notice that the description of the condition under which the polynomial evaluates to zero is the same as the definition of the associated Brainfuck instruction.

| $op$   | $i$  | description                                                | $instr^{(i)}\_{op}(\vec{r}_n, \vec{r}\_{n+1})$                                                                       |
| :----- | :--- | :--------------------------------------------------------- | :----------------------------------------------------------------------------------------------------------------- |
| `+`    | 1    | instruction pointer increases by 1                         | $ip\_{n+1} - ip_n - 1$                                                                                              |
| &nbsp; | 2    | memory pointer stays the same                              | $mp\_{n+1} - mp_n$                                                                                                  |
| &nbsp; | 3    | memory value increases by 1                                | $mv\_{n+1} - mv_n - 1$                                                                                              |
|        |
| `-`    | 1    | instruction pointer increases by 1                         | $ip\_{n+1} - ip_n - 1$                                                                                              |
| &nbsp; | 2    | memory pointer stays the same                              | $mp\_{n+1} - mp_n$                                                                                                  |
| &nbsp; | 3    | memory value decreases by 1                                | $mv\_{n+1} - mv_n + 1$                                                                                              |
|        |
| `>`    | 1    | instruction pointer increases by 1                         | $ip\_{n+1} - ip_n - 1$                                                                                              |
| &nbsp; | 2    | memory pointer increases by 1                              | $mp\_{n+1} - mp_n - 1$                                                                                              |
| &nbsp; | 3    | —                                                          | $0$                                                                                                                |
|        |
| `<`    | 1    | instruction pointer increases by 1                         | $ip\_{n+1} - ip_n - 1$                                                                                              |
| &nbsp; | 2    | memory pointer decreases by 1                              | $mp\_{n+1} - mp_n + 1$                                                                                              |
| &nbsp; | 3    | —                                                          | $0$                                                                                                                |
|        |
| `[`    | 1    | mv != 0 ⇒ ip increases by 2 <br> mv == 0 ⇒ ip is set to ni | $\begin{align*}&mv_n \cdot(ip\_{n+1} - ip_n - 2) \\ {}+{} &(mv_n\cdot mvi_n - 1)\cdot(ip\_{n+1} - ni_n)\end{align*}$ |
| &nbsp; | 2    | memory pointer stays the same                              | $mp\_{n+1} - mp_n$                                                                                                  |
| &nbsp; | 3    | memory value stays the same                                | $mv\_{n+1} - mv_n$                                                                                                  |
|        |
| `]`    | 1    | mv == 0 ⇒ ip increases by 2 <br> mv != 0 ⇒ ip is set to ni | $\begin{align*}&(mv_n\cdot mvi_n - 1)\cdot(ip\_{n+1} - ip_n - 2) \\ {}+{} &mv_n \cdot(ip\_{n+1} - ni)\end{align*}$   |
| &nbsp; | 2    | memory pointer stays the same                              | $mp\_{n+1} - mp_n$                                                                                                  |
| &nbsp; | 3    | memory value stays the same                                | $mv\_{n+1} - mv_n$                                                                                                  |
|        |
| `.`    | 1    | instruction pointer increases by 1                         | $ip\_{n+1} - ip_n - 1$                                                                                              |
| &nbsp; | 2    | memory pointer stays the same                              | $mp\_{n+1} - mp_n$                                                                                                  |
| &nbsp; | 3    | memory value stays the same                                | $mv\_{n+1} - mv_n$                                                                                                  |
|        |
| `,`    | 1    | instruction pointer increases                              | $ip\_{n+1} - ip_n - 1$                                                                                              |
| &nbsp; | 2    | memory pointer stays the same                              | $mp\_{n+1} - mp_n$                                                                                                  |
| &nbsp; | 3    | —                                                          | $0$                                                                                                                |

##### Example Program
The processor table for the example program `++>,<[>+.<-]` was already derived when we derived the execution trace, as they are the same. The values in the processor table can be verified to match all above constraints.

| clk  | ip   | ci             | ni             | mp   | mv   | mvi       |
| :--- | :--- | :------------- | :------------- | :--- | :--- | :-------- |
| $0$  | $0$  | $a_+$          | $a_+$          | $0$  | $0$  | $0$       |
| $1$  | $1$  | $a_+$          | $a_>$          | $0$  | $1$  | $1$       |
| $2$  | $2$  | $a_>$          | $a_,$          | $0$  | $2$  | $2^{-1}$  |
| $3$  | $3$  | $a_,$          | $a_<$          | $1$  | $0$  | $0$       |
| $4$  | $4$  | $a_<$          | $a_\texttt{[}$ | $1$  | $97$ | $97^{-1}$ |
| $5$  | $5$  | $a_\texttt{[}$ | $13$           | $0$  | $2$  | $2^{-1}$  |
| $6$  | $7$  | $a_>$          | $a_+$          | $0$  | $2$  | $2^{-1}$  |
| $7$  | $8$  | $a_+$          | $a_.$          | $1$  | $97$ | $97^{-1}$ |
| $8$  | $9$  | $a_.$          | $a_<$          | $1$  | $98$ | $98^{-1}$ |
| $9$  | $10$ | $a_<$          | $a_-$          | $1$  | $98$ | $98^{-1}$ |
| $10$ | $11$ | $a_-$          | $a_\texttt{]}$ | $0$  | $2$  | $2^{-1}$  |
| $11$ | $12$ | $a_\texttt{]}$ | $7$            | $0$  | $1$  | $1$       |
| $12$ | $7$  | $a_>$          | $a_+$          | $0$  | $1$  | $1$       |
| $13$ | $8$  | $a_+$          | $a_.$          | $1$  | $98$ | $98^{-1}$ |
| $14$ | $9$  | $a_.$          | $a_<$          | $1$  | $99$ | $99^{-1}$ |
| $15$ | $10$ | $a_<$          | $a_-$          | $1$  | $99$ | $99^{-1}$ |
| $16$ | $11$ | $a_-$          | $a_\texttt{]}$ | $0$  | $1$  | $1$       |
| $17$ | $12$ | $a_\texttt{]}$ | $7$            | $0$  | $0$  | $0$       |
| $18$ | $14$ | $0$            | $0$            | $0$  | $0$  | $0$       |

Let $n$ be the row number which for this table is always equal to the $clk$ column value. The you can verify that for all rows, all constraint polynomials hold. For example
- $clk\_{n+1}-clk_n - 1 = 0$
- $mv_n\cdot(mv_n \cdot mvi_n - 1) = 0$ and $mvi_n\cdot(mv_n \cdot mvi_n - 1) = 0$
- If $ci_n = a_+$ then $mv\_{n+1} - mv_n - 1 = 0$
- If $ci_n = a_\texttt{]}$ then $(mv_n\cdot mvi_n - 1)\cdot(ip\_{n+1}-ip_n-2)+mv_n\cdot(ip\_{n+1}-ni)$

**Exercise**: Verify that the above four constraints apply for the row pairs (1,2), (10,11), and (15,16).

The deselector expression does a lot of the heavy lifting here. It allows us to make conditional arithmetic constraints such as saying "*if instruction is $a_>$ then ...*". This is achieved by choosing the deselector expression such that it evaluates to zero for all but one instruction.

For the $a_+$ instruction we have the deselector expression
$$\prod\_{op' \neq a_>}(ci - a\_{op'}) \\ = (ci - a_-)\cdot(ci - a_>)\cdot(ci - a_<)\cdot(ci - a_\texttt{[})\cdot(ci - a_\texttt{]})\cdot(ci - a_.)\cdot(ci - a_,)$$

This expression evaluates to $0$ if the instruction is anything else than $a_+$ because one of the factors will be zero. And since the the transition polynomials must all evaluate to zero when the table values are plugged in, we can ignore all the constraints except the one whose deselector is not zero for this instruction. This is what allows us to describe the arithmetization in a conditional way such that we can say "*if $ci = a_+$ then ...*".

The deselector is also what makes the $C_2$ consistency constraint unnecessary. The $C_2$ would ensure that the instruction in the current instruction (`ci`) register is a valid instruction. If the `ci` register does not hold a valid instruction, all the deselector expressions evaluate to a non-zero value and all transition constraints are active. Since the transition constraints for different instructions are mutually exclusive, this ensures that no invalid instruction will be accepted in `ci`.

#### Memory Table
The memory table is formed from three register values sorted first by memory pointer, then by cycle. Memory table consists of three rows: `clk`, `mp`, and `mv`. That is: cycle count, memory pointer, and memory value.

Note that the index $n$ in the following equations refers to row number in the instruction table and not to cycle count. Only in the processor table do the index and the cycle count correspond, because only the processor table is sorted by the cycle count. In this section, $n$ is the row number after sorting the instruction table.

Only the processor table has consistency and boundary constraints. So we only consider transition constraints on the memory table.

| id    | description                                                                                                                                                                                                                        | polynomial                                                          |
| :---- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------ |
| $M_0$ | Memory pointer increases by one or by zero. Brainfuck only allows the memory pointer to change by one in each cycle.                                                                                                               | $(mp\_{n+1}-mp_n-1)\cdot(mp\_{n+1}-mp_n)$                             |
| $M_1$ | If memory pointer does not increase and the memory value *does* change, then the cycle count must only increase by one.                                                                                                            | $(mp\_{n+1} - mp_n - 1)\cdot(mv\_{n+1}-mv_n)\cdot(clk\_{n+1}-clk_n-1)$ |
| $M_2$ | If memory pointer increases by 1, then memory value must be set to zero. This is because Brainfuck memory is initialized to zero and an increase in the $mp$ value represents the 1st cycle that this memory location is accessed. | $(mp\_{n+1} - mp_n)\cdot mv\_{n+1}$                                   |

The transition constraint $M_1$ ensures that memory access is consitent: When a value is re-visited it cannot have changed in the interim. $M_2$ deals with the correct initialization of memory.

##### Example Program
The processor table for the example program `++>,<[>+.<-]` was presented above. The rows in the memory table are a permutation of rows of the processor table, but the transition constraints on the memory table only deal with three registers, `clk`, `mp`, and `mv`, so the memory table only needs three columns.

| clk  | mp   | mv   |
| :--- | :--- | :--- |
| $0$  | $0$  | $0$  |
| $1$  | $0$  | $1$  |
| $2$  | $0$  | $2$  |
| $5$  | $0$  | $2$  |
| $6$  | $0$  | $2$  |
| $10$ | $0$  | $2$  |
| $11$ | $0$  | $1$  |
| $12$ | $0$  | $1$  |
| $16$ | $0$  | $1$  |
| $17$ | $0$  | $0$  |
| $18$ | $0$  | $0$  |
| $3$  | $1$  | $0$  |
| $4$  | $1$  | $97$ |
| $7$  | $1$  | $97$ |
| $8$  | $1$  | $98$ |
| $9$  | $1$  | $98$ |
| $13$ | $1$  | $98$ |
| $14$ | $1$  | $99$ |
| $15$ | $1$  | $99$ |

The transition constraints for the memory table are
- $(mp\_{n+1}-mp_n-1)\cdot(mp\_{n+1}-mp_n)$
- $(mp\_{n+1}-mp_n-1)\cdot(mv\_{n+1}-mv_n)\cdot(clk\_{n+1}-clk_n-1)$
- $(mp\_{n+1}-mp_n)\cdot mv\_{n+1}$

**Exercise**: Verify that the above three constraints apply for the row pairs with clock cycle (0, 1) (18,3) and (9, 13).

#### Instruction Table
The instruction table has three columns: The instruction pointer `ip`, the current instruction `ci`, and the next instruction `ni`. The rows are formed by first concatenating the entire program with the execution trace and then sorting the resulting rows by instruction pointer. So the instruction table is always longer than the processor table by the size of the program.

| id    | description                                                                              | polynomial                                    |
| :---- | :--------------------------------------------------------------------------------------- | :-------------------------------------------- |
| $I_0$ | Instruction pointer increases by 0 or 1.                                                 | $(ip\_{n+1} - ip_n - 1)\cdot(ip\_{n+1} - ip_n)$ |
| $I_1$ | If instruction pointer is unchanged, then <br>the current instruction is also unchanged. | $(ip\_{n+1} - ip_n - 1)\cdot(ci\_{n+1}-ci_n)$   |
| $I_2$ | If instruction pointer is unchanged, then <br>the next instruction is also unchanged.    | $(ip\_{n+1}-ip_n-1)\cdot (ni\_{n+1} - ni_n)$    |

Since the entire program is included and the program occupies a contiguous part of the instruction memory, starting from 0, the instruction pointer can only increase by 0 or 1 for each row. Given this requirement is met, we can use $(ip\_{n+1} - ip_n - 1)$ to check if the instruction pointer has changed in the next row. If the instruction pointer has not changed, then both the current instruction and next instruction must be unchanged in the next row.

The intuitive interpretation of these constraints is that a program cannot change during its execution. When the program returns to an instruction pointer inside a for-loop, the value of the instruction cannot have changed. That also applies to the "next instruction" value, as this holds any jump destination.

##### Example Program
The instruction table for the example program `++>,<[>+.<-]` is formed by concatenating the program with the processor table and sorting for instruction pointer and then clock cycle. Each instruction in the program will be repeated $m+1$ times in this table where $m$ is the number of times the instruction is executed.

| ip   | ci             | ni             |
| :--- | :------------- | :------------- |
| $0$  | $a_+$          | $a_+$          |
| $0$  | $a_+$          | $a_+$          |
| $1$  | $a_+$          | $a_>$          |
| $1$  | $a_+$          | $a_>$          |
| $2$  | $a_>$          | $a_,$          |
| $2$  | $a_>$          | $a_,$          |
| $3$  | $a_,$          | $a_<$          |
| $3$  | $a_,$          | $a_<$          |
| $4$  | $a_<$          | $a_\texttt{[}$ |
| $4$  | $a_<$          | $a_\texttt{[}$ |
| $5$  | $a_\texttt{[}$ | $13$           |
| $5$  | $a_\texttt{[}$ | $13$           |
| $7$  | $a_>$          | $a_+$          |
| $7$  | $a_>$          | $a_+$          |
| $7$  | $a_>$          | $a_+$          |
| $8$  | $a_+$          | $a_.$          |
| $8$  | $a_+$          | $a_.$          |
| $8$  | $a_+$          | $a_.$          |
| $9$  | $a_.$          | $a_<$          |
| $9$  | $a_.$          | $a_<$          |
| $9$  | $a_.$          | $a_<$          |
| $10$ | $a_<$          | $a_-$          |
| $10$ | $a_<$          | $a_-$          |
| $10$ | $a_<$          | $a_-$          |
| $11$ | $a_-$          | $a_\texttt{]}$ |
| $11$ | $a_-$          | $a_\texttt{]}$ |
| $11$ | $a_-$          | $a_\texttt{]}$ |
| $12$ | $a_\texttt{]}$ | $7$            |
| $12$ | $a_\texttt{]}$ | $7$            |
| $12$ | $a_\texttt{]}$ | $7$            |

The transition constraints for the memory table were
- $(ip\_{n+1} - ip_n - 1)\cdot (ip\_{n+1} - ip_n)$
- $(ip\_{n+1} - ip_n - 1)\cdot (ci\_{n+1} - ci_n)$
- $(ip\_{n+1} - ip_n - 1)\cdot (ni\_{n+1} - ni_n)$

**Exercise**: Verify that these three transition constraints hold for the row pairs where the $ip$ values are (1, 1); (5,5); and (10, 11).

#### Input and Output Table
The input table consists of all the values that are printed to standard out. The `,` instruction in Brainfuck reads a character from input and stores this in the memory cell that the memory pointer points to. The `.` instruction prints the value from the indicated memory cell to output.

The length of the input table is the number of reads encountered in the execution of the program. The length of the output table is the number of writes in the execution of the program.

There are no constraint polynomials defined for the base columns for these tables.

##### Example Program
The input table for the example program `++>,<[>+.<-]` consists of one row and one column whose value is the value entered by the user when the program was run. In the above examples we set this input value to `a = 97`.

| input value  |
| ---- |
| $97$ |

The output table for the example program consists of the values that were printed to standard out. These values are the two ASCII codepoints after what the user entered.

| output value  |
| ---- |
| $98$ |
| $99$ |

## Extension Columns
The correctness of each table is proved with the constraints defined above. But there is something missing: The constraints above ensure that each table is internally consistent, but we also need to prove that the tables all refer to the same program execution.

This is done by proving that each table is either a permutation or a subset of another table.

To prove that a specific table is a permutation or a subset, we extend the tables with more columns than the ones which correspond to register values. We call these new columns **extension columns**.

To link a table to another, both of the linked tables need one extension column. The value in this extension column is either a running product (for permutations) or running sum (for sublists). The terminal value of these extension column will agree for an honest STARK proof when defined as follows:

**Permuation Running Product** ($prp$)
$$ prp\_{n+1} = init\cdot \prod\_{i=0}^{n+1}\left( \beta - d\cdot clk_i - e\cdot mp_i - f \cdot mv_i \right) $$
The $\beta$, $d$, $e$, and $f$ values are chosen by the Fiat-Shamir public oracle. This running product is calculated in two tables that are permuations of each other.


This product is independent of the order in which the register values appear. The above expression calculates the values in the extension columns linking the memory table and the processor table. The $init$ value is an initialization value that is chosen by the prover and must be equal for both linked tables.

##### Example Program
Let's add the permutation running product ($prp$) that links the memory table and the processor table to the processor table. For this, we set $\beta = 3, d = e = f = 1$ to make the calculations simpler. We also assume that the user enters an `a` as the input. In a real proof, these values would be chosen by the Fiat-Shamir public oracle, and the calculations would take place in larger domain than the B field. We set $init = 7$.

| clk  | ip   | ci             | ni             | mp   | mv   | mvi       | prp                    |
| :--- | :--- | :------------- | :------------- | :--- | :--- | :-------- | :--------------------- |
| $0$  | $0$  | $a_+$          | $a_+$          | $0$  | $0$  | $0$       | 7                      |
| $1$  | $1$  | $a_+$          | $a_>$          | $0$  | $1$  | $1$       | 7                      |
| $2$  | $2$  | $a_>$          | $a_,$          | $0$  | $2$  | $2^{-1}$  | $18446744069414584314$ |
| $3$  | $3$  | $a_,$          | $a_<$          | $1$  | $0$  | $0$       | $18446744069414584342$ |
| $4$  | $4$  | $a_<$          | $a_\texttt{[}$ | $1$  | $97$ | $97^{-1}$ | $18446744069414584258$ |
| $5$  | $5$  | $a_\texttt{[}$ | $13$           | $0$  | $2$  | $2^{-1}$  | $18446744069414590684$ |
| $6$  | $7$  | $a_>$          | $a_+$          | $0$  | $2$  | $2^{-1}$  | $18446744069414546143$ |
| $7$  | $8$  | $a_+$          | $a_.$          | $1$  | $97$ | $97^{-1}$ | $18446744069414851567$ |
| $8$  | $9$  | $a_.$          | $a_<$          | $1$  | $98$ | $98^{-1}$ | $18446744069414851567$ |
| $9$  | $10$ | $a_<$          | $a_-$          | $1$  | $98$ | $98^{-1}$ | $18446744072360704225$ |
| $10$ | $11$ | $a_-$          | $a_\texttt{]}$ | $0$  | $2$  | $2^{-1}$  | $18446743754179754593$ |
| $11$ | $12$ | $a_\texttt{]}$ | $7$            | $0$  | $1$  | $1$       | $3467583127008$        |
| $12$ | $7$  | $a_>$          | $a_+$          | $0$  | $1$  | $1$       | $18446705926000187233$ |
| $13$ | $8$  | $a_+$          | $a_.$          | $1$  | $98$ | $98^{-1}$ | $457720972765056$      |
| $14$ | $9$  | $a_.$          | $a_<$          | $1$  | $99$ | $99^{-1}$ | $18395937041437663105$ |
| $15$ | $10$ | $a_<$          | $a_-$          | $1$  | $99$ | $99^{-1}$ | $5741194161392097408$  |
| $16$ | $11$ | $a_-$          | $a_\texttt{]}$ | $0$  | $1$  | $1$       | $9586652100225931044$  |
| $17$ | $12$ | $a_\texttt{]}$ | $7$            | $0$  | $0$  | $0$       | $12634263021116362185$ |
| $18$ | $14$ | $0$            | $0$            | $0$  | $0$  | $0$       | $765976425698632571$   |
|      |      |                |                |      |      |           | $5425144832537830614$  |

Let's also add the matching extension column to the memory table. The $init$ value and the $\beta, d, e, f$ values must be the same as those used in the processor table.

| clk  | mp   | mv   | prp                    |
| :--- | :--- | :--- | :--------------------- |
| $0$  | $0$  | $0$  | $7$                    |
| $1$  | $0$  | $1$  | $7$                    |
| $2$  | $0$  | $2$  | $18446744069414584314$ |
| $5$  | $0$  | $2$  | $18446744069414584342$ |
| $6$  | $0$  | $2$  | $18446744069414584195$ |
| $10$ | $0$  | $2$  | $18446744069414585203$ |
| $11$ | $0$  | $1$  | $18446744069414574619$ |
| $12$ | $0$  | $1$  | $18446744069414691043$ |
| $16$ | $0$  | $1$  | $18446744069413303657$ |
| $17$ | $0$  | $0$  | $18446744069435074945$ |
| $18$ | $0$  | $0$  | $18446744069086734337$ |
| $3$  | $1$  | $0$  | $5573449728$           |
| $4$  | $1$  | $97$ | $18446744052694235137$ |
| $7$  | $1$  | $97$ | $1688755267584$        |
| $8$  | $1$  | $98$ | $18446568438866755585$ |
| $9$  | $1$  | $98$ | $18616838069846016$    |
| $13$ | $1$  | $98$ | $16454742395941060609$ |
| $14$ | $1$  | $99$ | $18198000992000704501$ |
| $15$ | $1$  | $99$ | $9661223678353835339$  |
|      |      |      | $5425144832537830614$  |
 
Both tables have a terminal value for the $prp$ column of $5425144832537830614$.
 
As required by the protocol and the laws of mathematics, both tables have the same terminal value exactly because the rows in the memory table is a permutation of those in the processor table. Had the rows in the processor table not matched those in the memory table, the running product in the two tables would have been different with a probability of $1 - \frac{1}{2^{64}}$. Stated differently: It would have been possible for the prover to cheat through brute force, by making on average $2^{63}$ proofs. Since we are targeting a higher security than this, all extension columns are actually calculated in an extension of the prime field. Elements in this extension field can be uniquely represented by a tuple of three base field elements, so the security for this check becomes considerably higher than $2^{128}$.
 
**Running Evaluation** ($re$)
For tables that are not permutations of each other but rather one being a sublist of the other, we cannot use the permutation running product as defined above.

Instead we use the running sum.

$$ re_n = \sum\_{i=0}^n\delta^{n-i} mv_i $$
This sum depends on the order in which the $mv_i$ values appear, so it can be used to verify that one table contains rows that are an (order-preserving) sublist of another table.

##### Example Program
Let's expand the output table with its running evaluation extension column. The $\delta$ (delta) in the equation above is one of the prover's challenges, and is picked by the Fiat-Shamir random public oracle. To make the calculations simple here, we set it to 2 but in a real program it is a random extension field element which can be expressed as $\mathbb{F_B}^3$ where $\mathbb{F_B}$ is the (base) prime field defined by the prime $2^{64} - 2^{32} + 1$.

| output value | re                  |
| :----------- | :------------------ |
| $98$         | $98$                |
| $99$         | $98 \cdot 2 + 99 = 295$ |

Had the prover cheated with the order and switched the rows around, they would have calculated this instead

| output value | re                  |
| :----------- | :------------------ |
| $99$         | $99$                |
| $98$         | $99 \cdot 2 + 98 = 296$ |


This shows that the running evaluation guarantees to preserve the order with a security that is the size of the field in which the challenge is picked and in which the extension column values are calculated.


But how does this work in the processor table? How are the extension column values that link the processor table to the output table calculated?

The $i^{th}$ row of the linked column in the processor table is calculated as 
$$eval_n = \begin{cases}
    eval_{n-1},                    & \text{if } ci_n \ne a_, \\\\
    \gamma\cdot eval_{n-1} + mv_n, & \text{if } ci_n = a_, \\\\
\end{cases}$$
where $a_,$ is the `,` (read) instruction and $mv\_{n+1}$ is the (n+1)$^{th}$ memory value, i.e. the value that was read in the execution of this `,` instruction.

This piecewise function can be arithmetized in a similar way that the instructions are arithmetized: with a deselector polynomial that only activates a transition requirement if the current instruction is a `,` (read) instruction: $\prod\_{op' \neq op}(ci - a\_{op'})$.

### Transition Quotients
Extension columns, like base columns, also have transition constraints. These transition quotients for extension columns do no more than validate the integral calculation of the running product or sum. They are formed on the basis of transition constraints that link two adjacent rows of the table where the extension columns have been added.

### Terminal Quotients
The terminal values of the extension columns are public. The prover includes the terminal values in the proof stream and the verifier checks that a low-degree quotient can be formed from both linked extension columns by dividing out the zerofier of the terminal, which is a linear equation with root in the last element in the group generated by the $\omicron$ generator, that is: the generator that forms the cyclical subgroup over which the tables are interpolated. If the terminal values do not match, at least one of these divisions will **not** result in a low-degree polynomial as the division would be $\frac{P(x)}{x-r}$ where $P(x)$ does not have a root in $x = r$. Only if $P(x)$ has a root in $x = r$ does $\frac{P(x)}{x-r}$ result in a low-degree polynomial. The FRI protocol ensures that the submitted codeword is of low degree.

### Boundary Quotients
A cell value $perm_i$ of the extension column of a permutated table is calculated recursively as $perm\_{n+1} = perm_n\cdot \alpha\cdot\sum_i c_i\cdot reg_i$ where the $reg_i$ values are cell values of the other rows, and the $c_i$ values are challenges chosen by the verifier through Fiat-Shamir. And where $perm_0 = init$ where $init$ is a value chosed by the prover which must be randomly sampled from a uniform distribution to achieve zero-knowledge. The $init$ values of the permutated columns must be the same for both colums, otherwise an adversary could pick a desired terminal value, get the challenges and calculate the column values from the desired terminal value and find a derived initial value. To thwart this attack, we calculate $\frac{f(x) - g(x)}{x - 1}$ where $f(x), g(x)$ represent the permutation-column values. The division by zero only results in a low-degree polynomial if $f(x), g(x)$ share the initial value in their first row, which is represented by the value of the polynomials in $\omicron^0=1$ (omicron to the power of zero). $\omicron$ (omicron) is the generator of the cyclic subgroup over which $f(x), g(x)$ were interpolated.


### Marrying Processor Table and Instruction Table
The height of the instruction table is that of the processor table **plus** the length of the program (the number of instructions). We need to prove that a subset of the rows in the instruction table are a permutation of the processor table, and that a subset of the rows in the instruction table are the program. **And** we need to prove that these sets are disjoint and that their union constitute all rows in the instruction table. To see exactly how this is done, you should have a look at [Alan Szepieniec's tutorial](https://aszepieniec.github.io/stark-brainfuck/) that does a better job describing permutation and evaluation arguments.

## Further Reading
As mentioned above, Alan Szepieniec's tutorial does a better job at describing how tables are linked to each other. If you have an easier time reading code than equations, you can have a look at either the [Python implemenation](https://github.com/aszepieniec/stark-brainfuck/) or the [Rust implementation](https://github.com/Neptune-Crypto/twenty-first/tree/master/twenty-first/src/shared_math/stark/brainfuck) the latter being a part of the [twenty-first](https://github.com/Neptune-Crypto/twenty-first) cryptography library that we have published.

If you're thinking that Brainfuck is a useless programming language and can't see the point of any of this, you can have a look at our more serious attempt at designing a STARK VM: [Triton VM](https://github.com/TritonVM/triton-vm).

---

[^1]: It's of course possible though that we could have done a better job. Maybe we could do with fewer registers if we had come up with different constraint polynomials.

[^wikipediaonbfdialects]: According to the [Wikipedia article](https://en.wikipedia.org/wiki/Brainfuck#Portability_issues) on Brainfuck, the type used for the cell values is not well-defined, so different dialects of Brainfuck use different types for this. Therefore, our choice of values in the interval $[0, 2^{64} - 2^{32} + 1)$ is just another dialect.

[^cyclicalsubgroup]: A cyclical subgroup is constructed from a generator by taking increasing powers of the generator, $g$ until $g^N = 1$. The generator is an element in the prime field, so a number between $1$ and $p-1$. $N$ is called the order of the subgroup and it equals the number of elements in this subgroup. Let $p = 17$. Then $2$ is a generator for a cyclical subgroup of order $8$ since $2^i = \\{1, 2, 4, 8, 16, 15, 13, 9, 1, 2, ...\\}$. The prime we use in this tutorial is $p=2^{64}-2^{32}+1$, and it has subgroups of order $2^i$ for all $i \geq 0$ and $i \leq 32$.

[^3]: In the associated implementation all the values in the base rows are prime field elements in the prime field defined by $p=2^{64}-2^{32}+1$, denoted $\mathbb{F}_p$ and all the extension rows are elements in an extension field to this prime field. An extension field is equivalent to how complex numbers are an extension to real numbers. The concrete extension field that is used for the extension columns is ${\mathbb{F}_p[x]}/{\langle x^3-x+1 \rangle}$.
