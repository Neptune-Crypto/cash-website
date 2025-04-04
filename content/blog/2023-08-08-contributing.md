+++
title = "How to Contribute"

[extra]
author = "Thorkil Værge"
ogtype = "article"
ogdescription = "Neptune seeks to deliver scalable private money to the world with an as fair a launch as possible. We need help from developers to make it stand on its own legs."
ogimage = "beavers-collaborating.png"
+++

![](beavers-collaborating.png)
<center>Open source software is about collaboration: anyone can contribute to build a better Neptune.</center></br>

# How to Contribute
Neptune is a fully open sourced project, which means anyone can contribute.

The [Neptune Core client](https://github.com/Neptune-Crypto/neptune-core/) is written in Rust, as is the [Triton virtual machine](https://triton-vm.org), and the [underlying cryptography library](https://github.com/Neptune-Crypto/twenty-first). Triton VM is a standalone repository since it [can be used outside of the context of a blockchain](https://neptune.cash/blog/announcing-tvm/).

We are offering bounties to anyone who makes significant contributions that help Neptune reach mainnet.

The [newly launched forum](https://talk.neptune.cash/) is a good place to start talking about Neptune and Triton VM.

## Getting Started
The easiest place to start is probably in the client: Even if you don't have any programming experience, you can still help by downloading and running the client and reporting any problems or submitting a feature request that you might have. You can do this [by opening an issue in the client's repository](https://github.com/Neptune-Crypto/neptune-core/issues). The neptune-dashboard makes interacting with the software easy. For now we have only tested the software on Linux distros.

On the other hand, if you are a developer that uses Windows or macOS, making sure it can compile in that environment would be very a welcome contribution.

## For Non-developers
Even if programming is not for you, there are many ways in which you can contribute: You can help with communication, fundraising, community building, graphics design, or testing.

## For Developers
If you want to start contributing to the client development, you can look for [GitHub issues tagged with "good first issue"](https://github.com/Neptune-Crypto/neptune-core/issues?q=is%3Aissue+is%3Aopen+label%3A%22good+first+issue%22) or look for "TODO"s in the code. To get an overview of the structure of the client's codebase, you can read the [developer snapshot](https://github.com/Neptune-Crypto/neptune-core/blob/689db5cab8d673b3b11d930e03442ba2fbf6d0be/developer_docs/documentation_snapshots/20221011.md) which details design principles, pitfalls, and the overall structure.

Also, if you're confused: try writing documentation. We can correct your understanding if necessary, and you can help shrink the learning curve for others.

### Towards Mainnet
The main missing components are consensus logic that can run in the virtual machine: transaction logic, block logic, and recursive proof verification. We could use help with both the writing of this logic in a high-level language as well as its compilation to Triton VM assembly. We've written a large part of this logic directly in Triton VM assembly, TASM for short. This process is [slow and tedious](https://neptune.cash/blog/alphanet/#observations) though: writing the program for removal record integrity check, a part of transaction validity logic, took us half a day in Rust. Translating it manually to assembly took a month. A good compiler would be able to significantly speed up this process.

An overview of the remaining tasks to get to mainnet can be found [here](https://github.com/Neptune-Crypto/neptune-core/issues/10).

It's possible to write all the needed programs directly in TASM but we think that a compiler to automate this translation is worth the effort. If you disagree we would be thrilled to see algorithms implemented in TASM. So don't let our assumptions hold you back.

### Compiler
There are two compiler projects ongoing:
1. [OmniZK](https://github.com/greenhat/omnizk) which compiles from Rust via WebAssembly (WASM) to Triton VM assembly (TASM)
2. [tasm-lang](https://github.com/TritonVM/tasm-lang) which compiles from Rust to a custom AST to TASM

Currently, the exact way forward for the compiler project is unclear. Going through WASM seems like a good idea as we might get a lot of optimizations for free, but how do we utilize the [Triton-VM specific instructions](https://triton-vm.org/spec/instructions.html#hashing) if we have to go through WASM? You can fork and build on top of any of these compiler projects.

If you're up for the task and you think it's the right approach, you're also very welcome to start a new compiler project. If you do, you probably want to make use of the [standard library for TASM](https://github.com/TritonVM/tasm-lib) that we have built. Using the standard library is a good idea in any case, as it ensures consistent data structure and prevents you from having to reinvent the wheel.

### The Recufier
Given a program and an input, Triton-VM can produce an output, just like any other VM. But if you run the VM's prover, it can also generate a *proof* that the program and input matches the output — that the rules of the virtual machine were obeyed when executing the program. And if you run the VM's verifier, these proofs can be verified faster than it would take to rerun the program.

Fast verification is a big deal, but the holy grail of verifiable computation is to run the verifier in the VM itself, thus generating a proof that another proof was valid. This is the recursive verifier, or *recufier* as we call it. The verifier is currently a program that only exists as Rust code. It hasn't yet been compiled to or written in TASM. If you want to help on that problem, see [this GitHub issue](https://github.com/TritonVM/triton-vm/issues/215). Or contribute to one of the compiler projects until it's capable enough to generate the TASM of the verifier.

## Bounties
We want to reward contributions that take us closer to mainnet launch. For this reason, we reserve a part of the premine for developer bounties. For reference, the maximum limit of Neptune coins that will ever be brought into existence is 42.000.000, and the total size of the premine is 831.600 as defined by [Neptune's tokenomics](../tokenomics/). We allocate 60.000 coins for bounties.
- 20.000 for compilers translating from Rust to TASM
- 20.000 for consensus logic, bug discoveries, and client development
- 20.000 for porting existing compilers like [gcc](https://gcc.gnu.org/) and [rustc](https://www.rust-lang.org/) to TASM, such that compilation can be done in Triton VM producing cryptographically verified binaries

The reward is at the discretion of the Neptune founders. All significant contributions that are used by the Neptune team and that fall into any of the categories above will be eligible for a reward. The bounties can be awarded as long as coins still remain in the pot and can be allocated from the premine. If all bounties in a specific category are not rewarded prior to launch, they can be moved to other projects, such as developing layer-2 protocols like [Lightning](https://neptune.cash/blog/scalable-privacy-problem/#lightning) and [public smart contracts](https://neptune.cash/blog/data-availability/).

## Take Initiatives!
Open source projects are about collaboration. If you see something you think can be improved, go for it! We cannot predict or steer all the ways in which this project can grow, so it's up to the community to see things that we cannot.

A truly decentralised blockchain must stand on its own legs. This is why the founding company will eventually dissolve and hand control over to the community. Without a community to support Neptune, its vision cannot be realized. Help us build a more perfect blockchain.
