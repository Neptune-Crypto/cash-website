+++
title = "Announcing Triton VM"

[extra]
author = "Jan Ferdinand Sauer"
ogtype = "article"
ogimage = "recursive-4.png"
ogdescription = "Triton is a Turing-complete virtual machine with fast recursive zk-STARK verification as its primary design goal. Having access to fast, recursive verification of computational integrity changes how to think about computation in networked environments."
+++

# Announcing Triton VM

The source code for [Triton Virtual Machine is now open source](https://github.com/TritonVM/triton-vm). 🎉

Triton is a Turing-complete virtual machine with fast recursive [zk-STARK](https://neptune.cash/learn/stark-anatomy/) verification as its primary design goal.
While it is being developed as one of the centerpieces of the Neptune protocol, Triton VM can be used for whatever you want.
Having access to fast, recursive verification of computational integrity changes how to think about computation in networked environments.

Here are some example statements you can prove – in zero knowledge, and quickly verifiable – using Triton VM:
- every participant of a multiplayer game with hidden information acted according to the game's rules
- the order matching algorithm of the stock exchange treated everyone fairly
- this publicly available library contains an exploitable bug
- the provided picture shows a dog, and the (secret) model is this certain about that
- these two STARK proofs are both valid

## Recursive STARKs of Computational Integrity

Normally, when executing a machine – virtual or not – the flow of information can be regarded as follows.
The tuple of (`input`, `program`) is given to the machine, which takes the `program`, evaluates it on the `input`, and produces some `output`.

![](recursive-1.svg)

If the – now almost definitely virtual – machine also has an associated STARK engine, one additional output is a `proof` of computational integrity.

![](recursive-2.svg)

Only if `input`, `program`, and `output` correspond to one another, i.e., if `output` is indeed the result of evaluating the `program` on the `input` according to the rules defined by the virtual machine, then producing such a `proof` is easy.
Otherwise, producing a `proof` is next to impossible.

The routine that checks whether or not a `proof` is, in fact, a valid one, is called the Verifier.
It takes as input a 4-tuple (`input`, `program`, `output`, `proof`) and evaluates to `true` if and only if that 4-tuple is consistent with the rules of the virtual machine.

![](recursive-3.svg)

Since the Verifier is a program taking some input and producing some output, the original virtual machine can be used to perform the computation.

![](recursive-4.svg)

The associated STARK engine will then produce a proof of computational integrity of _verifying_ some other proof of computational integrity – recursion!
Of course, the Verifier can be a subroutine in a larger program.

Triton VM is specifically designed to allow fast recursive verification.
It achieves this by using hash functions that play nicely with the underlying proof system, and by specifying instructions tailored to verifying STARK proofs.
For example, have you ever heard of the instruction [`divine_sibling`](https://github.com/TritonVM/triton-vm/blob/master/specification/isa.md#hashing) before?
Well, neither had we – but it's tremendously useful.
Check out the [instruction set architecture](https://github.com/TritonVM/triton-vm/blob/master/specification/isa.md) for a complete picture.

## Triton VM and the Neptune Protocol

In Neptune, Triton VM will be used to keep the total proof size required for validating the entire chain constant.
Concretely, when a new block is generated, the proof of block validity contained therein also proves that the previous block, including _its_ proof, was correct.
Thus, no matter how long the chain is, a single proof validates the entire chain.

## Next steps

The [Triton VM repository](https://github.com/TritonVM/triton-vm) already contains code for the virtual machine itself, as well as rust code for generating and validating proofs of computational integrity.
The next step is to bring the Verifier into Triton VM, thus achieving recursion.

If you want to learn about the inner workings of the STARK engine, check out [this excellent tutorial](https://aszepieniec.github.io/stark-brainfuck/) by [Alan](https://github.com/aszepieniec), one of our co-founders.
Even though the virtual machine in the tutorial is a different, way simpler one, the inner workings of the STARK engine are almost identical.
In fact, the simpler instruction set architecture allows to focus on the proof system without too much distraction.

Lastly, we'd love to hear what you can use Triton VM for!
[Get involved](https://github.com/TritonVM/triton-vm) and feel free to [contact us](mailto:ferdinand@neptune.cash).
