---
layout: post
title:  "Project Update"
date:   2024-05-09 15:25:33 -0700
categories: athena update
---

It’s been a couple of months since we announced the Athena project and a month since the last update. I’ve spent the past few weeks doing research into best practices and the state of the art in blockchain VM and zkVM design, tooling, programming language design, etc. across a number of ecosystems including EVM, Move, Solana, Bitcoin, NEAR, and Cairo (I previously wrote a little bit about some of the [initial experimentation](https://rettig.substack.com/p/blockchain-devex-battle)). I want to take the opportunity to update the community about how the Athena project is going: what has and has not changed from the goals and plans we originally announced, what’s been done, what we’re working on right now, and where we go from here.


# Goals

We previously shared [eight goals](https://spacemesh.io/blog/introducing-athena/) for Athena: incentive compatibility and security, simplicity, decentralization, scalability, ZK compatibility, avoiding vendor lockin, developer experience, and developer safety. I want to zoom in on a subset of those goals that I’m especially focused on right now and discuss some of the tradeoffs among them, and to add one new goal.


## Developer Experience

It’s difficult to measure developer experience since it’s by definition highly subjective. As one common example, some developers prefer frameworks, languages, and tools that are [highly opinionated](https://hackernoon.com/opinionated-or-not-choosing-the-right-framework-for-the-job-6x1u2ga0) and reduce the number of decisions they make; others prefer more degrees of freedom at the cost of higher complexity and more security risks. There’s no one-size-fits-all solution for all developers, teams, and use cases. Our goal as infrastructure designers and developers is therefore to pick a reasonable compromise and to build a system that allows both kinds of experiences to be built on top, to the extent possible.

One such choice we have to make with Athena is whether to align ourselves more closely with a more opinionated, blockchain-specific programming language and toolchain such as EVM or Move versus aligning more closely with a non-blockchain-specific language and toolchain, the most obvious of which is Rust. There are advantages and disadvantages to each choice, but I think it makes more sense to place ourselves firmly within the Rust ecosystem, especially if we’re able to do so with mainline Rust and LLVM (as opposed to a bespoke fork of Rust like [other projects](https://solana.stackexchange.com/questions/12530/whats-the-benefit-of-using-platform-tools-compared-to-rusts-upstream-bpfel-unk) have chosen). While languages and tooling in blockchain-specific ecosystems like EVM, Move, and Cairo have matured a lot, the reality is that they’ll never come close to the maturity and wide choices available of tooling available in a “global” developer ecosystem like Rust and LLVM, which is at least an order of magnitude greater (probably more).

The blockchain technical community is big and important, but it’s not as big or important as the broader universe of developers, and to appeal to and attract the largest possible number of developers we must design and deliver tools that are as familiar to developers as possible. We didn’t state so explicitly but I’ll do it now: one of the main goals for Athena is to attract millions of non-blockchain developers and we’ll be much better able to do that with Rust and LLVM than with Solidity, Move, or another bespoke programming language.

The plan is still for Athena smart contracts to be written in idiomatic, standard Rust, to include at least partial support for the Rust standard library, to be compiled using mainline Rust and a mainline LLVM target, and to support all existing LLVM tooling including compiler, debugger, optimizations, etc.


## Performance

Performance wasn’t on the original list of goals but it’s sort of implied under “scalability.” There are several dimensions to performance and ensuring that Athena is as performant as possible imposes several constraints on its design space. For one thing the target bytecode should as small as possible so that Athena programs don’t impose an undue storage burden on Spacemesh L1 miners. It should also be possible to compile programs using [`no-std`](https://docs.rust-embedded.org/book/intro/no-std.html) to product artifacts that are as small as possible.

Recall that Athena programs need to run in two different but related contexts: initially, during the optimistic phase, Executors will directly execute Athena programs (and Spacemesh Miners will as well in case they need to adjudicate a dispute), and later on Provers will run them through ZK circuits to generate succinct ZK proofs of faithful execution. It’s therefore important that Athena be as performant as possible in _both_ execution contexts, which is a unique challenge and a unique goal of this project.

While it’s not possible to perfectly and completely optimize for both of these competing goals simultaneously, we still think RISC-V is an excellent choice and serves both execution paths well. RISC-V is a simple, performant, well-designed instruction set that performs well compared to alternatives including Wasm and Solana rBPF. We need to do some additional benchmarking but the reports we’ve seen from others are [promising](https://github.com/koute/polkavm/blob/master/BENCHMARKS.md). In particular to optimize the direct execution path, RISC-V is a good choice because the instruction set is simple, is based on an actual hardware ISA (unlike Wasm), and by carefully choosing the RISC-V subset we support (in particular, a recent “embedded” variant that uses only 16 registers) we can expect very good performance. It should be possible to compile Athena code to RISC-V bytecode, to perform some initial optimizations (a process that happens offchain), and then to store the result on chain. Executors will subsequently take this RISC-V bytecode, promote it to native code and execute it at near-native speed.


## Succinctness

But that’s not all! Today there’s not [one](https://github.com/risc0/risc0), not [two](https://github.com/succinctlabs/sp1), but in fact [three](https://github.com/a16z/jolt) independent tools that support efficiently generating succinct proofs (i.e., ZK proofs) for vanilla RISC-V code (and others that [look promising](https://github.com/NilFoundation/zkLLVM)). These tools are at varying degrees of maturity but they’re maturing rapidly; we’ve tested them and studied their code and believe that they’ll work well for our use case. The fact that these projects are open source, that each is designed and implemented independently and that each is based on a different circuit design is reassuring.

While we’re not primarily focused on succinctness or proving efficiency at this stage it’s still important to ensure that Provers will be able to efficiently generate execution proofs in the future when we begin the transition from optimistic to ZK. While there are slight differences among the precise instruction set supported by each of these tools, and in particular among their use of syscalls, we’re confident that we can design a program format that can be proven by any of these tools with minimal translation.

While it’s possible to prove the execution of Ethereum transactions using these tools (and they’ve produced [proofs](https://github.com/risc0/zeth) of [concept](https://github.com/succinctlabs/sp1-reth) to this effect), doing so is inherently inefficient since it effectively requires writing a _separate_ Ethereum execution client, compiling that client to RISC-V, and then running it inside the ZK circuit (which means EVM smart contract code is running inside a VM that’s running inside a zkVM—yes, you read that correctly). This leads to one of the **boldest, most novel goals of Athena: to produce smart contract program code that can be run _natively_ inside a RISC-V execution circuit**, i.e., without adding an extra layer of indirection. As far as we’re aware Athena is the first ever attempt to do this and it should make proving Athena execution at least two orders of magnitude faster and cheaper than doing the equivalent with EVM.


## Platform Independence

It’s just as important to speak to the paths not taken. We took a good, long look at two ecosystems in particular: Move and Cairo. Both have a lot going for them and I have many good things to say about both. Starkware recently open sourced both the [current Cairo prover](https://github.com/starkware-libs/stone-prover) as well as an exciting, [next-generation prover](https://github.com/starkware-libs/stwo) based on a novel technology known as [Circle STARKs](https://eprint.iacr.org/2024/278). Cairo has matured rapidly from an [obscure domain-specific ZK language](https://perama-v.github.io/cairo/examples/first_application/) to something that looks and feels a lot [like modern Rust](https://www.cairo-lang.org/cairo-v2-6-0-is-out/). In fact, Cairo now looks quite a bit like Move and I doubt that’s an accident. In my estimation the Move language and VM are mature and exceptionally well-designed. Move addresses many of the shortcomings of EVM. The syntax for writing smart contracts in Move is concise and intuitive. The [resource and ownership model](https://move-language.github.io/move/structs-and-resources.html) (inspired by Rust itself) makes a lot of sense and, in particular, it means that common smart contract patterns can be expressed in code that’s an [order of magnitude simpler](https://medium.com/@kklas/smart-contract-development-move-vs-rust-4d8f84754a8f) than, say, in Solana.

Unfortunately unlike Cairo Move does not currently have good ZK support and this is a big reason we’re not considering it. We looked and were unable to find any successful attempts to succinctly prove execution of Move code efficiently. And while Cairo is designed to be ZK friendly it relies on a custom circuit and a custom language and VM and we believe [the future is general purpose circuits](https://a16zcrypto.com/posts/article/faqs-on-jolts-initial-implementation/#section--8) like those designed for RISC-V.

On top of these issues, platform independence and avoiding vendor lockin are big reasons to develop something new. While there are currently two dominant, well-funded projects in the Move ecosystem, support for Move is constrained to one small corner of the (already-small) blockchain ecosystem. Move also relies heavily upon [formal verification](https://github.com/Zellic/move-prover-examples) for correct execution, expertise that our team lacks. Cairo, while it has the backing of another well-funded project, is even more obscure than Move. If either Move or Cairo were to be abandoned or replaced by their respective patrons we’d be “left holding the bag” and unable to independently maintain the necessary infrastructure.

By contrast, Rust, LLVM, and RISC-V are universal standards, as universal as you can get in computing. We can confidently build Athena in the Rust ecosystem and know that we won’t be left holding anyone’s bags in the future.


## Compartmentalization

Something quickly became obvious while doing this research: with respect to our plans and goals for Athena and the tools we’ll use to achieve them, we’re standing on the shoulders of giants in a very big way. In particular, building something like Athena even two or three years ago would’ve cost $100M and required a huge team including compiler and programming language experts, cryptographers and ZK experts, protocol designers and mechanism design experts, etc. We have some of this expertise on our team but not all of it, and Spacemesh has nowhere near that many resources at its disposal. Nevertheless Athena is achievable today due to some massive breakthroughs and advances, and thanks to wonders of open source software and research. In particular ZK technology has made enormous advances in recent years and the emergence of the abovementioned tools that allow proving RISC-V code efficiently using a general-purpose circuit is a key enabling technology. (AI tools that have helped me get up to speed on a lot of these topics also feel like a superpower and deserve our gratitude!)

Given these limitations it’s as important to define what Athena is _not_ as what it is. Athena is _not_ a novel ZK circuit design. We’re not ZK experts and we’re not hand-writing circuits for Athena; instead, we’re black-boxing this aspect of the Athena design and relying on the work of others. Nor is Athena intended to reinvent the wheel with respect to a best-in-class blockchain account and data model. Experts have spent a lot of money and done a lot of the heavy lifting for us already, and as a result the Athena design will be heavily informed and inspired by the design of projects like Move in this regard.

We will rely on existing code and ideas as much as possible in the design and engineering process for Athena in order to limit its complexity, and we’ll give credit where credit is due for ideas and code that we utilize!


# What’s Been Done So Far

The deterministic VM, programmability, embedded computing, and zero knowledge space is big, much bigger than I anticipated when I began this research. The first few weeks of research were exciting but also exhausting because every day I learned about three new tools I needed to study and the list of topics to explore kept getting longer and longer. It took a few weeks but I did finally get through the entire queue. I want to share a snapshot of what I explored, what I learned, and trends that I see.

In rollups, I looked at the evolving design of projects like [Taiko](https://taiko.xyz/) which is planning to use not only RiscZero for proving but also SGX, and which is considering a [long list of other candidates](https://taiko.mirror.xyz/e_5GeGGFJIrOxqvXOfzY6HmWcRjCjRyG0NQF1zbNpNQ). I studied Optimism’s plan to [use ZK proving](https://github.com/ethereum-optimism/ecosystem-contributions/issues/61) to replace its current, [flawed](https://x.com/EdFelten/status/1783847133461860839) dispute resolution mechanism. I took a very close look at Arbitrum [Stylus](https://docs.arbitrum.io/stylus/stylus-gentle-introduction), which is Wasm-based, including its Rust SDK and the design that allows multiple, independent VMs to talk to one another, which is unique as far as I know. I studied how Stylus [validates](https://docs.arbitrum.io/stylus/stylus-gentle-introduction#activation) contract code before execution.

In the zkVM and ZK space I looked at existing and emerging zkEVMs including the ones from zkSync, Scroll, Polygon, and EF. As mentioned I spent a lot of time looking at [RiscZero](https://github.com/risc0/risc0/) and [SP1](https://github.com/succinctlabs/sp1/) and at their proof-of-concept Ethereum engines, [zeth](https://github.com/risc0/zeth) and [sp1-reth](https://github.com/succinctlabs/sp1-reth). I also looked into the [prover](https://blog.succinct.xyz/succinct-network/) [markets](https://www.risczero.com/bonsai) being [created](https://nil.foundation/proof-market) by the same projects, which we may want to consider using when Athena upgrades to a succinct design. I’m excited about the progress being made on [Lasso and Jolt](https://a16zcrypto.com/posts/article/introducing-lasso-and-jolt/), and about the potential of improvements like [Binius](https://vitalik.eth.limo/general/2024/04/29/binius.html). I looked at a novel tool called [Powdr](https://www.powdr.org/) for designing modular zkVMs, and attempts to do [Move in ZK](https://www.zkmove.net/). I studied recursive proving and techniques like batching and [continuations](https://www.risczero.com/blog/continuations).

In the VM space more generally I studied the Solana VM, the Move VM (Diem, Aptos, and Sui), the NEAR VM, the Stacks VM, and emerging projects on Bitcoin including ordinals, inscriptions, and runes. I developed, tested, and deployed code in each of these ecosystems, and in EVM, using modern tooling. I studied how each system addresses hard problems like state expiry/state rent. I explored wackier, more out-of-the-box VM designs including [Urbit](https://urbit.org/), which I was already familiar with, and [ao](https://ao.arweave.dev/), which I was not (no stone left unturned!). I studied proposed extensions to RISC-V including [CHERI-RISC-V](https://community.spacemesh.io/t/succint-proofs-of-risc-v-transactions-using-a-black-box-riscv-zkvm/416/5?u=lane) which would provide additional memory safety guarantees. Most useful of all, I spent a lot of time understanding the prior work done on [PolkaVM](https://forum.polkadot.network/t/announcing-polkavm-a-new-risc-v-based-vm-for-smart-contracts-and-possibly-more/3811/) in the Polkadot ecosystem: that project is about a year ahead of Athena and, while it has [different goals](https://github.com/koute/polkavm?tab=readme-ov-file#design-goals), it strongly validates many of the ideas proposed for Athena.

There are two important outcomes from all of this preliminary work. First, I can say with confidence that there’s nothing out there that ticks all of the boxes that Athena does (a developer-friendly VM based on a modern, non-domain specific programming language and toolchain that runs at near-native speeds and also natively targets an efficient zkVM). Second, I still believe that our proposed design is the best choice. In other words, I stand by everything we said in our previous [Athena announcement](https://spacemesh.io/blog/introducing-athena/).


# Stages

Athena is obviously a large, ambitious project. As I indicated at the top of this post the preliminary research phase is now complete and it’s time to start the next phase, which involves developing a proof of concept implementation. Ongoing work will proceed in four mostly independent, mostly parallelizable stages.


## Low Level Design

Basically everything I wrote above is low level design. This includes programming language and SDK design, core architecture including word size and register count, choice of instruction set and syscall regime including IO, bytecode artifact format, compiler and linker toolchain, and how to handle cross-contract calls/the call stack. It includes precompiles. It includes the strategy for compilation, interpretation (if appropriate), and execution more generally including verification of smart contract code and gas metering. It includes benchmarking all of these things.


## High Level Design

The high level design builds upon the low level design but also allows abstraction and “black boxing” of the low level VM. It includes the design of the account model, how code and state are stored and updated, and how they’re permissioned. This is likely to include concepts like account abstraction and resources.


## Mechanism Design

The mechanism design is basically the design of the rollup-like architecture that powers Athena. It includes each of the [Athena roles](https://spacemesh.io/blog/introducing-athena/)—relay, miner, executor, prover—and the incentives and game theory that ensures that the entire system runs in a secure and efficient fashion.


## Succinctness

Succinctness involves the “ZK rollup” design that Athena will transition to over time. It involves questions including which proving system we use, how to parameterize the various tradeoffs involved in proving (required compute and memory, proving time, proof size), and how to handle edge cases like what happens if a proof is missing or after a reorg.

