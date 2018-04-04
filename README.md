# Formal Verification of Ethereum Contracts (Yoichi's attempts)

This is the start page about my efforts around smart contract verification.

## Goal

The goal is to establish a method how to verify a smart contract so that
no surprises happen after their deployment.

## Talks

* gave a talk in DEVCON 1
* Berlin, 2016-11-1; gave [a talk](https://yoichihirai.com/deedtalk.pdf) in Berlin Ethereum Meetup ([video](https://www.youtube.com/watch?v=cCUGMAnCh7o&list=PLaM7G4Llrb7wPiT2G75tj2JQr8qg6P5hi&index=3))
* Paris, EDCON, 2017-02-17 [slide](https://yoichihirai.com/edcon-yoichi-hirai.pdf) [video](https://www.youtube.com/watch?v=P98SZ-PznaU)
* Cambridge, ARM Research Summit, 2017-09-13 [slide](https://yoichihirai.com/cambridge2017.pdf)

## Links

* _every update comes here_: a [gitter channel](https://gitter.im/ethereum/formal-methods)
* an [Isabelle/HOL repo](https://github.com/pirapira/eth-isabelle)
* [Solidity](https://gitter.im/ethereum/solidity/)
* a [Coq repo](https://github.com/pirapira/evmverif) and a [screen cast](https://youtu.be/Mzh4fyoaBJ0?list=PL9oaY6Y4QxRZybj86eGItGVApxLXVIXHz)
* [FP Ethereum community](https://github.com/fp-ethereum/fp-ethereum)

## The rest of the page

* What are formal methods
* [Verification of EVM bytecodes](https://github.com/pirapira/ethereum-formal-verification-overview/blob/master/README.md#path-a-verification-of-evm-bytecodes-currently-followed)
* [Verification of Solidity Programs](https://github.com/pirapira/ethereum-formal-verification-overview/blob/master/README.md#path-b-verification-of-solidity-programs)
* [A Safe Programming Language](https://github.com/pirapira/ethereum-formal-verification-overview/blob/master/README.md#sideway-a-safe-programming-language)
* [Dr-Y's contract analyzer](https://github.com/pirapira/ethereum-formal-verification-overview/blob/master/README.md#sideway-dr-ys-contract-analyzer)
* [Transaction visualization](https://github.com/pirapira/ethereum-formal-verification-overview/blob/master/README.md#sideway-transaction-visualization)
* [Resources](https://github.com/pirapira/ethereum-formal-verification-overview#resources)


## What are formal methods

What are formal methods?  The word "formal" here is about looking at the shape, not at the meaning, of mathematical proofs.

For a long time, mathematical proofs were read, understood and then checked.
In some cases you can do calculations like "a + b - b = a" by just looking at the form, saying something like "this cancels that".
Still, in the usual practice of mathematics, you ought to be able to explain what is going on.  You need to understand.

In the first half of 20th century, rigid languages appeared where proofs can be checked by machines.
[You can see examples here](http://us.metamath.org/mpegif/mmtheorems40.html#mm3944s).
Maybe the computers do not understand the meaning, but they can check the proofs.

When you translate mathematical texts into such machine-readable proofs, you are "formalizing" mathematics.
Recent decades saw a tantalizing progress in this area. The [Flyspeck project](https://code.google.com/archive/p/flyspeck/wikis/AnnouncingCompletion.wiki) and [CoqFiniteGroups](https://gforge.inria.fr/projects/coqfinitgroup/) formalized big results from the past centuries.
The [Homotopy Type Theory](https://homotopytypetheory.org/) was formalized from its very early stages.

So far this was about mathematics.  The take away is, you can obtain infinitely many truths at one shot.
The equality "a + b - b = a" is true for any natural numbers.
By the way, here we have a smart contract, whose input is actually only finitely many.  Can we do something about this?


## Path a: Verification of EVM bytecodes (currently followed)

One way is to look at the EVM bytecodes.  They are executed on a simple virtual machine.  The rules of the virtual machine is well understood by different Ethereum clients which usually match (otherwise they fix the difference with uttermost priority).
The current attempt in Coq is in [evmverif](https://github.com/pirapira/evmverif/) repository, and there is a [screen cast](https://youtu.be/Mzh4fyoaBJ0?list=PL9oaY6Y4QxRZybj86eGItGVApxLXVIXHz).

### Obstacles

* ~~Coq proofs are currently [lengthy](https://github.com/pirapira/evmverif/blob/master/coq/example/managed_account_with_accumulators.v#L405).~~ Isabelle/HOL made the proofs much shorter.
    - I believe Isabelle/HOL provides much easier user experience (because it has a well-polished machine word library).  I've ported the Coq attempt [into Isabelle/HOL](https://github.com/pirapira/eth-isabelle).
* At the bytecode level, it's harder to see what the code is doing and what to expect.
    - One solution is to make the Solidity compiler [annotate the bytecode](https://github.com/ethereum/solidity/issues/1178) with the expected properties at specific code location.
* At the bytecode level, Solidity array access looks like storage access with Kaccek hashes, and somehow we need to assume no collisions.  If I assume `∀ a, b. keccak(a) = keccak(b) -> a = b`, using the pigeon hole argument, I can prove `0 = 1` and everything.
    - I can locally say `sorry` or `admit` to indicate that I give up proofs whenever I have to prove `keccak(a) != keccak(b)` because `a != b`.  Then the proofs would be incomplete and I cannot call my results "theorems", but I'm happy that way.  Anyway the compiler assumes no hash collisions without logical justification (but empirical).
	- The module system of Coq provides [a clever workaround](https://github.com/bitemyapp/ledgertheory/blob/master/CryptoHashes.v).

### Steps on Path a

1. ~~try Isabelle/HOL to see if proofs are really 5 times easier there (less than a week)~~
    - By the way, about the trustworthiness, I would put Isabelle/HOL at least as good as Coq because Isabelle/HOL is based on a simpler deduction system (a deduction system is not a piece of software; it is a bunch of inference rules usually written in LaTeX)
2. ~~choose Coq or Isabelle/HOL (overnight)~~ I chose Isabelle/HOL over Coq.
3. ~~verify `Deed` and some other simple bytecode programs against simple properties (6-10 days)~~ This is finished and there is a [report](https://yoichihirai.com/deed.pdf).
4. ~~translate the definitions into Lem (2 weeks; almost done except the Keccak hash)~~ done.
5. ~~develop a method how to verify assertions between opcodes (this is [evmverif#5](https://github.com/pirapira/evmverif/issues/5)) (a week; there is a [milestone](https://github.com/pirapira/eth-isabelle/milestone/1) for this item)~~
6. ~~cover all opcodes (3 days): this is already in progress. A [milestone](https://github.com/pirapira/eth-isabelle/milestone/2).  mostly done,~~ [but LOGx does nothing right now](https://github.com/pirapira/eth-isabelle/issues/94)
7. ~~test the new EVM against the standard VM tests (4 weeks; started, now parsing the tests)~~ mostly done. [but VM tests involving multiple contracts are still skipped](https://github.com/pirapira/eth-isabelle/issues/95)
8. ~~[build a simple Hoare logic](https://github.com/pirapira/eth-isabelle/milestone/4)
9. combine the Hoare logic with an invariant tracking technique
9. develop Ethereum ABI
9. implement the packaging ABI
9. extend the model so that it can run state-tests
10. implement verification-condition generation -- the desired post-condition and invariant annotations at JUMPDESTs would generate the formulas to prove.
11. implement a wallet (might be between 8. and 9.)
12. build decompilation of EVM bytecode into a recursive function in a theorem prover
13. try to automate the process of verification / finding vulnerabilities

### Personal Account of the Struggle

* My EVM in Isabelle/HOL says "your memory contents might change after you CALL another account", and I'm like "no, that's paranoid."
* The verification of Deed contract finishes.  I break the contract to see what happens, then the proofs still work.  I had a bug in my JUMPI implementation.
* I try to verify the Deed contract again, I cannot prove the safety property anymore, and I find that one precondition was missing about `active` flag.
* I added gas to the EVM and run the Deed proofs again, it now takes overnight.  I forgot to `time` it.
* The OCaml extraction and the Isabelle/HOL extraction were using different endianness in the hash implementation.

## Path b: Verification of Solidity programs

Another way would be to verify Solidity sources somehow,
not looking at the EVM bytecode.

### Obstacles

* The only way to know the meaning of a Solidity program is to compile it into bytecodes.
* The Solidity compiler is changing fast and the verification tools would need to follow the changes
* The verification tools tend to believe a different behavior from that of actually deployed bytecode

### Steps on this path

The steps that have to be taken:

1. Know Solidity program's meaning without checking the bytecode.  For this, one way is to write a Solidity interpreter in Coq or in an ML language.  Another way is to compile a Solidity program into WhyML or F*.
2. Check the above against the reality.  We need a big set of tests to compare the above translation against the bytecode.
3. Translate the ML implementation into a proof assistant (or we rely on Why3, in that case we need to trust the SMT solvers)
4. Try to assert properties in a shallow-embedded way and write proofs
5. Automate the process, maybe by deep-embedding some kind of Hoare logic


## Sideway: Formal Verificaton of Proof-of-Stake Protocols

* Vitalik's Casper with dynamic validator sets https://medium.com/@pirapira/a-mechanized-safety-proof-for-pos-with-dynamic-validators-17e9b45faff4#.94adc75cm
* Vlad's Casper until Consensus Safety https://medium.com/@pirapira/formal-methods-on-another-casper-8a75f6e02073#.qdps9277i
* Vitalik's Capser with static validator set https://medium.com/@pirapira/formal-methods-on-some-pos-stuff-e309775c2ab8#.vaaa77dut


## Sideway: a safe programming language

This alone does not verify smart contracts, but I have a chance to make it static analyzer friendly.

The [bamboo compiler](https://github.com/pirapira/bamboo) is now producing snippets of bytecode for the empty contract (still lots to be done).

The language has
* no loops or recursions
    - this makes life much easier for static analyzers
    - gas consumption is stabler
    - users would need to call it again and again
* persistent program counter
    - this discourages multiple workflows in a single contract
    - reentrancy does not jump to far away in the source code, but reentrancy arrives around the last sentence executed
* protection against reentrancy
* explcit storage setting at every exit

It is inspired by an existing language but I will talk about it when it's actually working.


## Sideway: Dr-Y's contract analyzer

The [contract analyzer](https://github.com/pirapira/dry-analyzer) needs an overhaul in the functionality and the UX.
I want to see the meaning of each basic block, and how the execution can jump around the basic blocks.


## Sideway: transaction visualization

A quick hack [to visualize the dataflow in a transaction](https://github.com/pirapira/vmtrace_enricher).
This is useful after a surprise.

## Resources

* [oyente](https://github.com/ethereum/oyente)
* [ethereum-lem](https://github.com/mrsmkl/ethereum-lem)
* [EVM in K framework](https://github.com/kframework/evm-semantics)
* [Scilla: a Smart Contract Intermediate Level LAnguage](https://arxiv.org/pdf/1801.00687.pdf)

### An Incomplete List of Papers I Saw

* [Online Detection of Effectively Callback Free Objects with Applications to Smart Contracts](https://arxiv.org/pdf/1801.04032.pdf)
    - most Ethereum transactions so far are "effectively call back free"
