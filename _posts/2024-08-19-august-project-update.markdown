---
layout: post
title:  "August Project Update"
date:   2024-08-19 19:25:33 -0700
categories: athena update
permalink: /2024/08/19/august-project-update.html
---

I’m about to take off for a short summer holiday, but I owe you an Athena project update before that. In the past few weeks we finished the third phase, Blockchain Integration, and kicked off work on the fourth phase. The goal of this phase is to launch an Athena testnet.


## Tldr

In the last phase we successfully added [several host functions](https://github.com/athenavm/athena/blob/main/vm/hostfunctions/src/lib.rs) that allow Athena programs to read and write state, call other programs, and send coins. After that we began working on the very concrete goal of launching a barebones Athena testnet to allow users to begin testing what’s been built so far, while we continue to improve Athena and work on the remaining phases (mainly rollup design and ZK) in parallel. We made some important progress on upgrading the Athena Rust toolchain. Another Spacemesh developer has joined the project, and he made some big improvements to the FFI bindings which will make it much easier to integrate Athena into go-spacemesh. Finally, while rewriting the first, existing Spacemesh template, the single sig wallet template, in Athena, we realized there are still a few missing features which we’re currently working on.


## Toolchain

The Rust toolchain has been a headache since the beginning of the Athena project, as I [previously wrote about](https://rettig.substack.com/i/147359339/thing-rust-toolchain). The first toolchain release was based on Rust 1.78.0 and included a [fairly large patch](https://github.com/athenavm/rustc-rv32e-toolchain/blob/v0.2.0/patches/rust.patch) for compatibility with our RV32E instruction set. It only supported linux-amd64. I previously tried to upgrade to a more recent Rust release but kept running into problems due to [other](https://github.com/athenavm/rustc-rv32e-toolchain/issues/1#issuecomment-2156233834) [people’s](https://github.com/athenavm/rustc-rv32e-toolchain/issues/6) [bugs](https://github.com/athenavm/rustc-rv32e-toolchain/pull/8#issuecomment-2294022934). While Rust and LLVM feature experimental support for RISC-V in general, and RV32E specifically, “experimental” unfortunately means that not everything works out of the box on every platform/architecture.

Since then we’ve made several improvements to the toolchain, and this work is ongoing. Firstly, we succeeded in upgrading to Rust 1.80.0 which includes some nice [new language features](https://blog.rust-lang.org/2024/07/25/Rust-1.80.0.html). The patch is smaller, but it also includes a new, [small LLVM patch](https://github.com/athenavm/rustc-rv32e-toolchain/blob/v0.3.0/patches/llvm.patch) to work around yet another toolchain issue. I’m pretty sure I know how to remove this patch, and work on this is [ongoing](https://github.com/athenavm/rustc-rv32e-toolchain/pull/8). Secondly, with the [latest release](https://github.com/athenavm/rustc-rv32e-toolchain/releases/tag/v0.3.0), we’ve successfully added macOS support (only Apple Silicon for now).

One of the things that makes working on the toolchain so hard is that it takes > 30 minutes to compile locally, and up to two hours to compile when using Github Actions runners. This makes feedback loops incredibly slow: it takes hours to see the effects of a small change to the configuration. So we’ve also been working on finding faster runners that are still affordable. We successfully added support for [RunsOn](https://runs-on.com/), which is great for Linux runners, but have been unable to find better runners for Windows or Mac. If you are aware of a solution, please let us know.


## Missing Pieces

One of the great things about having a very concrete goal like integrating Athena into go-spacemesh and launching a testnet is that it immediately reveals things that are still missing. The first step in the integration is rewriting the existing Spacemesh templates for Athena.

As a recap, programs (smart contracts) in Spacemesh are split into two parts: a template and a program. (The terminology isn’t finalized and we’re open to suggestions for improvements.) You can think of a template like a class definition in an object oriented programming language—or, in more concrete terms, like an uninstantiated Rust struct. The act of placing a new template on chain is called “deploying.” Once this is done, a user can send a spawn transaction to spawn an instance of a template that they control, i.e., that contains their state. The first and most important template is the single sig wallet template, and the only initialization state associated with this template is the public key that controls it.

Templates in Spacemesh today are implemented natively in Go (which is why we sometimes refer to them as precompiles). They’re implemented in multiple pieces and the code is fairly complicated. Templates will be much cleaner in Athena: a template is just a single, ordinary Rust struct. And thanks to account abstraction, templates control their own serialization/deserialization, authentication, etc.

The first Athena template, the single sig wallet template, is more or less done, and the bulk of the work in this phase has been filling in the missing pieces in Athena around it to make it work. This includes a “spawn” host call, and the ability to save a serialized program instance to chain and restore it on a subsequent call. Getting this working without the use of a custom toolchain required some fairly complex Rust procedural macros, something else I [wrote about recently](https://rettig.substack.com/i/147861802/thing-its-just-rust). We also needed to add the ability to call into an Athena binary in [multiple places](https://github.com/athenavm/athena/issues/73)—i.e., to add a form of function dispatch.


## Wallet Template

As mentioned, the wallet template is the first canonical Athena program. It’s the analogue of an Ethereum “externally owned account” (EOA), i.e., an account controlled by a single keypair, under the Spacemesh model of account abstraction. The vast majority of transactions flowing through Athena will pass through this template, or another template modeled after it, for a long time, so it’s important that we get it right.

The wallet template implements four methods: spawn, deploy, send, and call. These are the core primitives for the Athena VM transaction and account model. Spawn takes a public key and spawns a new wallet program owned by that key, as mentioned above. Deploy deploys a new template, i.e., new code to the blockchain. Send sends coins to another account, and call allows the wallet owner to call a function on another program. For more on this design, see the [Athena design document](https://community.spacemesh.io/t/proposed-athena-design/433).

You can find the prototype code for the single sig wallet template [here](https://github.com/athenavm/athena/blob/vmsdk/examples/wallet/program/src/main.rs).


## Testnet

As previously mentioned, the concrete goal of this phase is to launch an initial, prototype Athena testnet so that we can begin dogfooding the things we’ve been designing and building for months. While a number of things are still up in the air, the outlines of the testnet are now taking shape. It’s likely that the first testnet will feature only a single template, the single sig wallet template described above. We probably won’t have a “deploy” feature and won’t allow users to deploy their own templates in the initial version. It’ll probably look and feel a lot like the single sig wallet template that’s live on the Spacemesh mainnet today, i.e., it’ll just be a cryptocurrency where you spawn a wallet and send coins to someone else’s wallet. The important thing, however, is that all of this will be happening on Athena, so it’ll function as an end to end test of everything we’ve built so far.

There will be multiple generations of testnets and we’ll fix bugs and add functionality over time. Once we’ve tested the single sig wallet, we’ll likely roll out a multisig wallet, and then we’ll switch on a “deploy” feature that will allow users to deploy their own programs. That’s when the fun will really begin.

We’re very hopeful that we can get the initial testnet running by the end of September. Please bear in mind that, while the testnet represents a major step forward on the Athena roadmap and on the goal towards launching Athena in production on mainnet, even after the testnet is up and running, there will still be a lot of work required to get Athena into production on mainnet.


## Host Integration

The other missing piece of the puzzle is integrating Athena into go-spacemesh in order to launch the testnet. For now, we’ll implement Athena as an alternative VM, next to the existing “genesis VM.” This means that it’ll be possible to run go-spacemesh with either VM, and for the purposes of the testnet, we’ll run it with Athena switched on. In other words, the testnet will run Athena as its only, layer one VM.

This isn’t how Athena will work when we ultimately roll it out on Spacemesh in production. As we originally described, Athena will be implemented as a “sovereign rollup” that runs on top of the existing Spacemesh layer one VM. Athena transactions will be sequenced together into bundles and the existing Spacemesh VM and miners will provide data availability to Athena.

But it’ll take time to finalize and implement that design, and in the meantime we’ll be able to have fun building and testing applications on the Athena testnet.


## What’s Next

There are still a number of important tasks in flight in this phase. We have a prototype of the single sig wallet template, but it still needs to be peer reviewed, tested, and merged. We have a prototype of the “spawn” host call working, but this also needs peer review and testing. We need to finalize the v0.3.0 toolchain, including hopefully removing the LLVM patch. Then we need to finish the go-spacemesh integration. As mentioned, we have a second developer working on the project now, and we’ll soon have a third, so we expect that development will speed up, but all of this will still take some time.

