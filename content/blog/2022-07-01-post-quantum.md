+++
title = "Neptune and Post-Quantum Security"

[extra]
author = "Alan Szepieniec"
+++

# Neptune and Post-Quantum Security

Neptune claims to offer post-quantum security. But quantum computers don't exist yet, at least for as far as we know. So how can we substantiate this claim if we don't have a quantum computer to verify it?

The short answer is that quantum computers do not consist of magic. They obey laws of physics. The analysis of quantum algorithms presupposes the correctness of our understanding of quantum physics, the [Schrödinger equation](https://en.wikipedia.org/wiki/Schr%C3%B6dinger_equation) in particular. We can study, analyze, and predict the behavior of quantum systems by using the equations that describe the quantum physics, just like we can study, analyze, and predict the trajectories of projectiles and celestial bodies using the laws of Newtonian physics.

In a more computational language: we can predict ahead of time what the running time of some classical algorithms is going to be, even if that running time is so enormous that no practical experiment can verify that prediction within our lifetimes. So too can we predict the running time of a quantum algorithm ahead of time, even though we don't have quantum computers yet to verify it.

All cryptographic security[^1] comes from the assumed hardness of computational tasks, meaning that *by assumption* no algorithm can solve the task in a reasonable period of time. Post-quantum security simply means that this hardness assumption extends to quantum computers as well.

## Computers and Quantum Computers

A computer is a piece of the universe that transforms information in a controlled manner. It is *controlled* in the sense that the operational mechanics have been purified by design to isolate the process of information transformation from the disturbing effect of unknown factors. The result is that the state evolves according to simple laws. While the computer operator could in principle derive the desired output from calculating with these laws of evolution, it saves effort to trick the universe into calculating it instead; the computer is the tool that enables this outsourcing.

Computers can be built out of any physical phenomenon that behaves according to sufficiently expressive physical laws. Modern computers are built out of transistors, but transistors are by no means the only brick that fits this description. Computers can and have been made out of steam, marbles, dominoes, and even mischiefs[^2] of rats in dynamic mazes. But all of these ways of building computers turn out to be equivalent in the following mathematical sense. You can use computer A to simulate the physical evolution of computer B and compute its output. This is known as the *physical Church-Turing thesis*: all physically realizable computers are equivalent. Things get interesting when you study the overhead cost of computing this simulation. For all the named constructions of computers, the overhead of simulation is *polynomial* – in layman terms, this means that not only is simulation *possible*, but it is also *efficient*.

What happens if we use quantum physics as the operational principle for a computer? According to Schrödinger's equation, we need an *exponential* number of complex numbers to completely describe the state of a quantum system. In the early eighties Richard Feynman postulated that quantum physics was indeed not efficiently simulatable by classical computers, but that quantum computers should be able to simulate quantum physics efficiently. 

The natural next question is, beyond simulating quantum physics, what other tasks can quantum computers do more efficiently than classical ones? In 1994 Peter Shor came up with an [efficient quantum algorithm](https://en.wikipedia.org/wiki/Shor%27s_algorithm) for a family of natural problems occurring in cryptography, namely period-finding. It turns out that both the integer factorization problem and the discrete logarithm problem (in any finite group, including elliptic curves) can be solved with black box access to a period-finder. Shor's quantum algorithm therefore breaks cryptography that relies on the hardness of factorization or discrete logarithms!

What other *natural* problems can quantum computers solve efficiently that classical computers can't? There are tons of *contrived* problems that fit this description, but in terms of practical problems that affect cryptography, Shor's algorithm and its brother [Simon's algorithm](https://en.wikipedia.org/wiki/Simon%27s_problem) (another period-finder) are the only ones on the list that we know of. [Grover's algorithm](https://en.wikipedia.org/wiki/Grover%27s_algorithm), variants thereof, and various quantum walks are proposed from time to time to attack cryptography, but these algorithms are quantum analogues of brute force and the net result is a reduction in complexity from very exponential running time to concretely less work but still exponential running time, and easy to defend against simply by increasing the key length.

## Cryptography and Proofs

Cryptography is the science of protecting information, and in a narrower sense it is the science of protecting information with hard computational problems. Integer factorization and the discrete logarithm are two such problems: it is easy to multiply two prime numbers but hard to factorize their product; and likewise it is easy to exponentiate a given group element to obtain another but hard to compute the exponent given the two group elements. But these are by no means the only hard problems used in cryptography. We also assume the hardness of distinguishing block ciphers like AES from random permutations, the hardness of finding collisions or preimages of hash functions such as SHA3, et cetera.

Some hard problems, such as finding short vectors in lattices, finding solutions to systems of multivariate polynomials, or finding isogenies between elliptic curves, are even rich enough in structure to generate public key cryptosystems. These *post-quantum* cryptosystems represent drop-in replacements for cryptosystems that rely on the hardness of factorization and discrete logarithms, because there is no characterization in terms of period finding that would enable an efficient quantum attack using Shor's algorithm. Stronger still: we do not know of any quantum algorithms that can solve them efficiently. While quantum computers generally do offer *some* speedup, the speedup for these problems is negligible.

We cannot preclude the future discovery of quantum algorithms that attack these problems efficiently. But such a discovery would constitute a massive breakthrough. Given the human capital devoted to studying post-quantum cryptography – the [books](https://www.amazon.com/Post-Quantum-Cryptography-Daniel-J-Bernstein/dp/3642100198), the [conferences](https://www.pqcrypto.org/conferences.html), the [theses](https://lirias.kuleuven.be/retrieve/525268), the [standardization projects](https://csrc.nist.gov/projects/post-quantum-cryptography) – I tend to judge the likelihood of such a breakthrough as low. But more importantly, *if* we are working under the assumption that a big cryptanalytic breakthrough is possible, why would it be limited to post-quantum hard problems? It might as well undermine the security of AES or SHA3 or the elliptic curve discrete logarithm problem against classical computers.

In order to use any cryptography at all, we have to rely on some hardness assumption. Then what's the point of proofs? Generally speaking, we cannot prove the security of cryptosystems[^3]. However, proofs in cryptography achieve at least three important things.
 - The proof establishes that a successful attacker (assuming one exists) can be used to solve the hard problem(s). Therefore, the assumption that a successful attacker exists, is incompatible with the assumption that the problem is hard.
 - The proof quantifies the security degradation induced by the above reduction. Specifically, this degradation tells us how the attacker's complexity relates to the complexity of the best known algorithm for the hard problem.
 - The proof modularizes by abstracting away concrete primitives. As a result, if an attack is discovered on the primitive in question, it can be replaced with another equivalent primitive without affecting the proof. 

## Effect of Quantum Computers on Cryptography

The previous discussion motivates the following recipe for upgrading a cryptosystem to withstand attacks from quantum computers.

1. To account for Shor's algorithm, identify all primitives that are vulnerable to period-finding. Typically this includes public key cryptosystems based on the elliptic curve discrete logarithm problem as well as factorization-based public key cryptosystems such as RSA and Paillier. These primitives need to be replaced with post-quantum counterparts.

2. To account for Grover's algorithm (and similar quantum brute force strategies), double the key length and hash output length. In fact, doubling is overkill because the complexity of quantum brute force algorithms is not quite the square root of their classical counterparts, owing to the need to implement the attacked primitive in a reversible manner.[^4]

3. Ensure that the security proof is still valid in the context of quantum attackers. Recall that the typical cryptographic security proof involves building a solver for the hard problem by using the assumed attacker as a subroutine. As a result, this constructed solver must ensure that the attacker has the same *quantum* interface to the attacked cryptosystem, even though the cryptosystem itself defines classical algorithms. After all, the quantum attacker can simulate these classical algorithms, or any classical algorithm for that matter.

Point number (3) is the most intricate. There are contrived security proofs that are valid assuming the attacker is classical, but invalid if he is not. However, the rule rather than the exception over the past few years has been that classically-valid proof techniques have a quantumly-valid counterpart.

## What about Neptune

Believe it or not, *the only[^5] cryptographic primitive upon which Neptune relies is a secure hash function*. It is a special kind of hash function because it is *[arithmetization-oriented](https://asdm.gmbh/arithmetization-oriented-ciphers/)* – in a nutshell, this property means that it works well with zero-knowledge proofs. There is no indication that arithmetization-oriented hash functions might be more susceptible to a quantum attack than traditional ones. The output size of the hash function used in Neptune is set to accommodate for future attacks, quantum or classical.

As for the security proofs, Neptune as a whole doesn't really have one – but then neither does Bitcoin. Part of the problem is to first define what is meant by "security", which is a tricky, multi-faceted thing. Another part of the problem is that researchers and peer-reviewers prefer works that is innovative, new, and surprising. A security proof for Bitcoin would hardly qualify for any of those categories.

That being said, specific components in Neptune can and do have security proofs. It is [known](https://eprint.iacr.org/2019/834), for instance, that the BCS transform for tranforming an IOP into an interactive STARK retains security even against quantum attackers. Likewise, the Fiat-Shamir transform, which turns an interactive proof into a non-interactive one, [is](https://arxiv.org/abs/1902.07556) [known](https://eprint.iacr.org/2019/262.pdf) to preserve security against quantum attackers.[^6] All of these results are relative to a *quantum random oracle*, which is an idealization of a hash function. This idealization would be proven to be inadequate if an attack on it were discovered.

To be fair, many components in Neptune remain without security proof, and these remain outstanding research-level tasks. However, the point is that there is no reason to assume quantum computers will make any difference. To the contrary: there is evidence to support the assertion that quantum computers *will not* make a difference.

## Conclusion

 - We can predict ahead of time what quantum computers will and will not be capable of.
 - Broadly speaking, only cryptosystems involving integer factorization or discrete logarithms are affected. These primitives must be replaced with post-quantum ones.
 - Neptune does not rely on integer factorization or discrete logarithms.
 - Many components in Neptune do not have security proofs (yet), but then all a security proof can do is reduce to *assumptions* anyway.
 - Quantum computers are *very* unlikely to be able to attack Neptune at a dramatically lower cost than classical computers are.

[^1]: I use the term *cryptographic security* narrowly, excluding information-theoretical security. I would classify the latter type of security as ... well, *information-theoretic*.

[^2]: A *mischief* is the term of venery for rats. A *herd* of cows, a *pack* of wolves, a *school* of fish, a *pride* of lions, a *murder* of crows, and, apparently, a *mischief* of rats.

[^3]: Specifically, we *can* prove the security of information-theoretical cryptosystems like the one-time pad. However, information-theoretical cryptosystems are generally impractical because of the large key size. We focus on practical cryptography, which trades off the large key against a reliance on hard problems.

[^4]: A weird quirk of quantum physics and quantum computers in particular is the difficulty of forgetting information. As a result, all quantum algorithms must be implemented in a reversible way, which guarantees that no information is lost.

[^5]: This statement refers only to the consensus rules. Other cryptographic primitives can be used on top of Neptune, for instance to enable paper wallets or lightning.

[^6]: The referenced papers pertain only to three-pass interactive arguments. A generic theorem pertaining to protocols with arbitrarily many rounds remains outstanding, but will surely soon be proven.
