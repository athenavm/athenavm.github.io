---
layout: post
title:  "June Project Update"
date:   2024-06-14 16:25:33 -0700
categories: athena update
permalink: /2024/06/14/june-project-update.html
---

It’s been a while since the last Athena project update. A lot of progress has been made since then so it seems about time for another update.

## Tldr

In the last update I introduced [four phases](https://www.athenavm.org/athena/update/2024/05/09/project-update.html#stages) to the Athena project. The first phase, the initial research phase, was finished around the time the last update was published, a month ago. The second phase, the low level design and implementation phase, is [now done](https://github.com/athenavm/athena) as well. While there were plenty of hiccups, this phase went much more quickly and smoothly than expected due to the magic of open source software (more on this below). We now have a working VM that fully implements our target instruction set, RV32IM (as well as the “EM” variant, more on this below as well). And, yes, it runs Doom ([almost](https://youtu.be/5rnGihyBPS0)!).

I think it makes sense to add another phase to the Athena roadmap, the phase we’re now moving into: the integration phase. This is where the fun really begins. The VM can run programs but it doesn’t yet do anything blockchain- or Spacemesh-related. In other words, those programs can’t do interesting things like read or write account state, send coins, etc.. The next phase involves adding blockchain-specific functionality such as gas metering, host functions, and cross-contract calls. It also involves defining [an interface](https://github.com/athenavm/athena/blob/45c7db1e307205ae93f780ac6ce94fd17928ac97/ffi/athcon/athcon.h) and “packaging up” Athena into a format that will allow go-spacemesh to interface with it (since Athena is written in Rust and go-spacemesh is written in Go).

At the end of this third phase, with luck, we’ll have a functional prototype blockchain VM running on a testnet and it’ll be possible to deploy and run simple smart contracts. Here’s what the overall Athena roadmap looks like and where we stand today:

- Phase 0: Initial Research (complete)

- Phase 1: VM Prototype (complete)

- Phase 2: Blockchain integration (in progress)

- Phase 3: Mechanism design

- Phase 4: Succinctness

Read on for more details on what’s been done and what comes next.


## Open Source Magic

As mentioned above, the second phase of Athena came together much more quickly than I anticipated. This is due to the magic of open source software and of permissionless innovation. We’re standing on the shoulders of giants, and by leaning into existing code we were able to pull together a fully-functional VM that meets our needs very quickly rather than reinventing the wheel. Two projects in particular deserve our gratitude.

The first is [SP1](https://github.com/succinctlabs/sp1) from Succinct Labs. You may recall SP1 from the [previous update](https://www.athenavm.org/athena/update/2024/05/09/project-update.html#whats-been-done-so-far) where I described it as a general-purpose, RISC-V zkVM that we intend Athena to target when we reach the ZK phase. It turns out that we were able to extract the SP1 RISC-V VM (without any of the ZK proving components) and use it as our [core VM](https://github.com/athenavm/athena/tree/main/core). We’re grateful for the work done by the Succinct Labs on this important public goods infrastructure, and we’re especially grateful that their code is extraordinarily clean and well structured.

The second is [PolkaVM](https://github.com/koute/polkavm), a project in the Polkadot ecosystem that shares many goals with Athena, including to build a modern, secure, performant RISC-V blockchain VM. Athena’s Rust toolchain (more below) is based on the PolkaVM Rust toolchain. (Unlike Athena, however, natively targeting ZK execution is not one of PolkaVM’s goals.)

It goes without saying that Athena is fully [open source](https://github.com/athenavm) and [permissively licensed](https://github.com/athenavm/athena?tab=readme-ov-file#license) under the same terms as these projects!


## Rust Toolchain

One of our goals for Athena is to use mainline Rust so that developing an Athena application simply means developing in the Rust language that developers are already familiar with, with the tools they’re already familiar with. We want Athena code to be compatible with all existing Rust tooling, including IDEs, debuggers, profilers, and indeed the entire, powerful, mature [LLVM toolchain](https://llvm.org/).

Under the hood the Athena VM is based on the RISC-V ISA. One of the nice things about RISC-V is that it’s modular: there are [many variants](https://en.wikipedia.org/wiki/RISC-V#Design) but the core instruction set is simple and fixed. RISC-V also offers the option of using 32 registers (the standard RV32I base) or 16 registers (RV32E, where “E” stands for “embedded”). There are multiple benefits to choosing RV32E, including the fact that it makes cross-compliation to underlying hardware easier (since most physical machines running Athena code implement amd64 which uses 16 registers). Another reason to prefer RV32E is that, in order to prove Athena execution in zero knowledge, Athena programs will need to be [wrapped in low-level runtime code](https://community.spacemesh.io/t/succint-proofs-of-risc-v-transactions-using-a-black-box-riscv-zkvm/416) that handles things like system calls and gas metering. By using only 16 registers and reserving the rest for this runtime code, we expect the integration task to be simpler.

While experimental support was [recently added](https://github.com/llvm/llvm-project/commit/3ac9fe69f70a2b3541266daedbaaa7dc9c007a2a) to LLVM (and thus to Rust, which depends on LLVM) for RV32E, this support is incomplete and, in particular, mainline Rust doesn’t yet include a `riscv32em` target. For now we therefore have no choice but to maintain [our own Rust toolchain](https://github.com/athenavm/rustc-rv32e-toolchain) for Athena as a fork of Rust with [a few small patches](https://github.com/athenavm/rustc-rv32e-toolchain/blob/main/patches/rust.patch) (creating this toolchain was by far the hardest part of this phase of Athena). For now, Athena users will need to download this toolchain before compiling and running Athena code, a task that is fully automated by the Athena tooling. We’re hopeful that this requirement will disappear in the fullness of time as full RISC-V support is added to Rust, and we’ll help advocate for this as well.


## Try It Out

You can already compile and run programs on Athena (with the caveat that, as mentioned above, these programs don’t do anything blockchain-specific yet). For now it’s only guaranteed to work on Ubuntu 22.04 or newer; it may work on other UNIX and Linux variants but is untested. It wouldn’t require a lot of work to get it running on macOS; we could use [some help](https://github.com/athenavm/rustc-rv32e-toolchain/issues/4) with this. We hope to eventually add Windows support but this will take time.

All you need to do is run the following commands to download and install Athena and its toolchain:

    > curl -L https://install.athenavm.org | bash && athup

You can then create a barebones test application and run it with the following commands:

    > cargo athena new fibonacci
    > cd fibonacci/script
    > cargo run

Here’s what a complete, barebones Athena application looks like. [As promised](https://www.athenavm.org/athena/update/2024/05/09/project-update#developer-experience)—it’s just Rust! You can expect to see more sophisticated examples soon as we add the blockchain-related functionality described above.

    #![no_main]
    athena_vm::entrypoint!(main);

    pub fn main() {
        println!("Hello, world!");
    }


## What’s Next

As mentioned above, the next phase is the “blockchain integration” phase where we add blockchain-specific functionality to the Athena VM. This includes things like reading and writing account state, deploying new programs/contracts, sending coins, gas metering, and cross-contract calls. It’ll also involve integrating the Athena codebase into the go-spacemesh codebase and launching a testnet that runs the new VM.

This phase of the design is [shaping up here](https://community.spacemesh.io/t/athena-blockchain-design-and-integration/424), for those who are curious to follow along or contribute. You can also follow progress on [this project board](https://github.com/orgs/athenavm/projects/1/views/1) and [this pull request](https://github.com/athenavm/athena/pull/21).


## Get Involved

The Athena project lives on GitHub and there are a number of open issues tagged as [Help Wanted](https://github.com/athenavm/athena/issues?q=is%3Aissue+is%3Aopen+label%3A%22help+wanted%22) and [Good First Issue](https://github.com/athenavm/athena/issues?q=is%3Aissue+is%3Aopen+label%3A%22good+first+issue%22). Contributions are welcome! Whether you’re an experienced Rust or VM developer, or are curious and want to learn, there’s lots you can do to help. If you’re not already on our Discord, please [join using this link](https://discord.gg/spacemesh) and join the active conversation in the #athena-vm channel.


## Ecosystem

To restate the obvious: Athena is first and foremost being designed and built by the Spacemesh team as the Spacemesh VM. As we [mentioned before](https://spacemesh.io/blog/introducing-athena/), Athena features a novel design that’s somewhere between a traditional L1 VM and a L2 rollup: a subset of Spacemesh nodes, known as Executors, are responsible for maintaining the Athena state on a day to day basis, relieving Spacemesh L1 miners of this responsibility. However, Spacemesh L1 miners are aware of Athena and, when necessary, can step in to adjudicate disputes that may arise under the optimistic design.

However, over the long term, we hope and expect that Athena will grow beyond the single, existing Spacemesh mainnet. In the same way that there are now dozens (if not hundreds) of production EVM-based blockchains, we’re intentionally designing and building Athena in such a way that it can be useful for lots of other projects and communities. A simple, secure, RISC-V-based VM that’s natively performant and can also natively target ZK is a powerful primitive with lots of potentially exciting use cases. We had to design Athena because, 15 years after the launch of the first blockchain, there is still no such tool today! While we like Move, for example, the language, VM, account model, and other aspects of the Move VM are tightly coupled (rather than modular) so it unfortunately [wasn’t an option](https://www.athenavm.org/athena/update/2024/05/09/project-update#platform-independence) for Spacemesh.

As Athena matures and use cases beyond Spacemesh mainnet emerge, we expect this to benefit the Spacemesh project and ecosystem as well. It’ll lead to additional resources and to more developers working on Athena, which will in turn benefit Spacemesh. It’ll also mean that smart contracts and tooling developed for Spacemesh will work out of the box with other chains that adopt Athena. It’s a win-win.

A great example here is [libp2p](https://libp2p.io/), which powers our networking stack. Historically, most blockchains implemented a bespoke networking stack. We initially did the same at Spacemesh. However, over time, we began to understand the challenges associated with maintaining a bespoke stack and the benefits of adopting a global standard became clear. At the same time, libp2p matured as a project, and we saw that it made sense to adopt infrastructure that has the support of Protocol Labs, Ethereum, and [many other projects](https://github.com/libp2p/go-libp2p?tab=readme-ov-file#notable-users). One way to think of Athena is as the libp2p of the VM world. This is the reason Athena lives in its own [Github organization](https://github.com/athenavm) and has its own [domain name](https://www.athenavm.org/): just as libp2p grew out of the IPFS and Protocol Labs ecosystem, we expect that as Athena matures it’ll grow beyond just the Spacemesh ecosystem and we want to make it as friendly as possible for other projects to adopt it and contribute to Athena.
