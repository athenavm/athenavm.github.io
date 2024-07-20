---
layout: post
title:  "July Project Update"
date:   2024-07-20 16:25:33 -0700
categories: athena update
---

We’re overdue for another Athena project update. Lots has happened over the past month. I was hoping that the current phase, Blockchain Integration, would be finished in time for this update. We’re not quite there yet, but we’re close.


# Tldr<a id="tldr"></a>

The first two project updates appeared at the completion of the first two phases: [Initial R\&D](https://www.athenavm.org/athena/update/2024/05/09/project-update.html), and [VM Prototype](https://www.athenavm.org/athena/update/2024/06/14/june-project-update.html). These phases are now visible in the [project README](https://github.com/athenavm/athena?tab=readme-ov-file#progress), and there’s a [project board](https://github.com/orgs/athenavm/projects/1) tracking this phase. This phase isn’t quite finished, largely due to travel for EthCC. The good news is that I gave the [first talk on Athena](https://ethcc.io/archive/Athena-Rises), had a lot of good conversations about the project, and received a lot of good feedback.

This phase is going well and the first few host functions are in place—including the hard ones! The FFI, which was the hardest part of this phase, is complete and [has been tested](https://github.com/athenavm/athena/blob/main/ffi/ffitest/src/lib.rs). We have basic gas metering and the ability to read and write account state. Most importantly, we have our [first prototype smart contract](https://github.com/athenavm/athena/blob/main/tests/recursive_call/src/main.rs)! It performs an end to end test that includes reading and writing account state, and calling back into itself recursively. This means that Athena is no longer just a RISC-V VM: with the addition of these features it’s a blockchain VM that now runs smart contracts that can call one another.


# FFI<a id="ffi"></a>

Our goal is for Athena to compile to a library that can be used in _any_ blockchain node implementation. The first of these will, of course, be go-spacemesh. But Athena is written in Rust and go-spacemesh is, obviously, written in Go. Wut do?

Enter FFI. FFI stands for [foreign function interface](https://en.wikipedia.org/wiki/Foreign_function_interface). It’s the means by which programs written in one language can interface _in process_ with a program written in a different language (i.e., without communicating over an API). The Athena FFI, known as [athcon](https://github.com/athenavm/athena/blob/main/ffi/athcon/athcon.h) (for “Athena connector”), lays out a common language (the technical term for this is [ABI](https://en.wikipedia.org/wiki/Application_binary_interface)) spoken both by Athena itself as well as by any host node that interfaces with it. It includes basic types such as account address, status codes, revision numbers, capabilities, and messages (an internal representation of a transaction). It also includes the _host interface,_ which lays out the set of host functions that allow Athena and the programs running on it to interface with the blockchain (see next section).

It took a long time to design this interface since it needs to be simple, generic, and as blockchain-agnostic as possible. We intend for Athena to work not only in Spacemesh but in the context of other blockchains as well! Abstracting away the underlying specificities of Spacemesh, or of any blockchain, into a clean API is not a simple exercise.

Athcon lays out a _host_ and a _VM._ The host is the blockchain node implementation, the thing that knows about accounts, addresses, balances, and code. The VM is Athena, i.e., the thing that _runs_ that code in the context of the host. Today, there’s only one host in the Spacemesh ecosystem, go-spacemesh, but in the future there will hopefully be many full node implementations written in many languages, and it’s important that any of them can run Athena. By the same token, for now Athena is the only full VM in the Spacemesh ecosystem, but we could imagine a future with multiple VMs. Just as athcon plugs into any host, it should also plug into any future VM. (Athcon is based on an Ethereum ecosystem project known as [EVM-C](https://evmc.ethereum.org/), the “EVM Connector”, which provides a similar interface between multiple Ethereum node implementations and multiple Ethereum VMs including EVM and Ewasm.)

Note that, while the purpose of the FFI is to make Athena usable in any programming language, in fact designing the Athena FFI was a useful exercise regardless of other languages. This is because a C interface is by definition constraining, using only C types and C function syntax as it does. When you build a component of a program in the same language, you can be lazy: you tend to pass around more data than you need, you may not think thoroughly about memory management or the lifecycle of particular data elements, you may expose internal methods and properties unnecessarily, etc. And you can make changes and updates as often as you like, without needing to worry about compatibility issues.

By contrast, the FFI needs to be extremely slim. It needs to be rigorous and well-defined, and it can use only basic C types and functions since it needs to work in any language and any environment. What’s more, you need to be extremely thoughtful about memory and lifecycle management, since you don’t get this for free the way you do when working with native types and data structures. And you can’t easily make changes since this will cause downstream incompatibilities.

By building the Athcon interface, I realized that, like designing any API, SDK, or data interchange format, wrapping your functionality in a clean FFI is probably always a good exercise. As one example of why this matters regardless of language or execution context, many of the Athena tests—which are of course also written in Rust—use the FFI for testing. Those tests can be cleaner and simpler than they would be without the FFI. Athcon is the [narrow waist](https://www.oilshell.org/blog/2022/02/diagrams.html) of Athena.


# Host Functions<a id="host-functions"></a>

What separates any old VM from a blockchain execution environment, or a smart contract engine? The answer is a set of host functions (plus gas metering, for which see below). “Host function” is a term from embedded programming. It refers to functionality provided by the “host”—which in our case is the node, i.e., go-spacemesh—to the execution environment, i.e., the running smart contract. Without host functions, a smart contract is just a simple program that cannot read from or write to the blockchain. In the same way that system calls (“syscalls”) allow a program running on your computer to do interesting things like read from and write to files on the hard drive, send and receive data over the network, display information on the screen, etc., host functions allow the running program to interface with the blockchain: to read and write contract state, to deploy a new smart contract, to send coins, to call into a another smart contract, etc.

One of the main goals of this phase of the Athena project is to implement an initial, straightforward set of host functions. This will allow simple smart contracts to run on Athena. The first such smart contract, which is a test that writes some data to storage and then reads back the same data, can be found [here](https://github.com/athenavm/athena/blob/bf587defaa1e0816ab3d4417ab85a4382280f086/tests/host/src/main.rs) and it looks like this:

```rust
#![no_main]

athena_vm::entrypoint!(main);

pub fn main() {
  // use 32-bit words
  let mut key = [2u32; 8];
  let mut key2 = [2u32; 8];

  // expected output value

  // [1u8; 32] i.e. 32 x 1-bytes
  // we convert [u8; 32] into [u32; 8] where each u32 is a 4 byte chunk
  // 0x01010101 == 16843009

  let value = [16843009u32; 8];

  // result will be written to key, overwriting input value
  unsafe { athena_vm::host::write_storage(key.as_mut_ptr(), value.as_ptr()) };
  assert_eq!(key, [0u32; 8], "write_storage failed");

  // result will be written to key, overwriting input value
  unsafe { athena_vm::host::read_storage(key2.as_mut_ptr()) };
  assert_eq!(value, key2, "read_storage failed");
  println!("success");
}
```

The host functions here are the `athena_vm::host::write_storage` and `athena_vm::host::read_storage` which, as the names suggest, allow a running smart contract to write a storage item, which will be stored permanently on the blockchain, and to read a storage item, respectively. For now, this code is pretty low level and requires the use of `unsafe` Rust, requires passing raw pointers, etc., but this will become cleaner and easier as Athena matures. We’ll provide an SDK that hides these details. For a more complicated example, see this [recursive host call test](https://github.com/athenavm/athena/blob/19f2ff6f7a4bae22f4c84f98d03271b3ec418443/tests/recursive_call/src/main.rs) using Fibonacci.

You can see the full, [initial set of host functions](https://github.com/athenavm/athena/issues/28) we’ll implement. This list will grow over time as Athena matures. Three have already been implemented; the hardest of these was the [`call` host function](https://github.com/athenavm/athena/issues/5), which allows a running smart contract to call out to another, external smart contract. This is tricky because it introduces recursion: the VM needs to pause the current execution frame, construct a new VM message, pass this over the FFI boundary into the host, then the host needs to call back into the VM to perform the call, the results of which are unwound through the same process in reverse. In a sense, implementing `call` is really the capstone of this entire phase since it requires testing all of the other blockchain-related functionality (host functions, the host-VM interface, message passing, etc.) end to end. I expect the remaining host functions won’t take very long, and then we can declare this phase, too, completed.


# What About Accounts?<a id="what-about-accounts"></a>

The astute reader may have noticed that we’re talking about reading and writing account state, but we haven’t actually announced the Athena account model yet. Isn’t this contradictory? Well, yes and no.

No, because we can actually abstract a lot of this complexity away from Athena. This is another upside of the Host-VM pattern described above. Details such as how an account is implemented under the hood are the concern of the host, not of Athena. Athena uses a host-provided “host function” to request that data be written to, or read from, an account (and as far as Athena is concerned, an account is just an opaque bytestring). Athena doesn’t actually need to know anything about how an account is implemented or how the host fulfills this request (or fails to). The goal is for Athena to be as unopinionated about its _execution context_ as possible. In the context of the blockchain, this means being agnostic as to which chain it’s running on, whether it’s running at layer one or layer two, etc. This both limits complexity and also ensures that Athena will run lots of places.

Yes, because there are limits to how far we can take this abstraction (and because all [abstraction boundaries are leaky](https://www.joelonsoftware.com/2002/11/11/the-law-of-leaky-abstractions/)). This works pretty well for simple functions, but it may not work so well when we get to more complex functions. Even for simple functions, we’ve made some assumptions. Reading and writing state currently assumes that state is a key-value store where keys and values are each 256-bit words, and that an account can only access its own state. That seems like a reasonable starting point because it’s [how EVM works](https://www.evm.codes/#54?fork=cancun), but we might decide to take things in a different direction with Athena in the future.

Another good example is deploying new programs (smart contracts). The Spacemesh layer one account model is based on templates, which are like classes, and individual accounts, which are instances of those templates. This works differently than it does in EVM. While we haven’t finalized the Athena account model, we’re leaning towards using the Spacemesh L1 account model in Athena, which means Athena _may_ need to be more opinionated about things like how new code is deployed. Note that Athena also supports features and capabilities that can be switched on and off, so even if we add support for Spacemesh-style templates, it doesn’t mean that Athena won’t work in an execution environment without these.

We’re also seriously considering adding [Move-style resources](https://move-language.github.io/move/structs-and-resources.html) and [abilities](https://move-book.com/reference/abilities.html) to Athena. That’s another example of something that Athena might need to understand.


# Gas Metering<a id="gas-metering"></a>

The other missing piece of the puzzle is gas metering. We need to be able not only to execute smart contract code safely and securely, and to allow smart contracts to interact with the blockchain via host functions, but we of course also need to perform gas metering to ensure that running smart contracts don’t consume more resources than they’re able to pay for.

Since one of the main goals of this phase of Athena is to keep things simple, for now we’re doing braindead simple gas metering. Each instruction has an identical cost, and we simply meter each executed instruction as we go. In practice, it’s of course not actually the case that every instruction has the same cost! Complex instructions, especially cryptographic operations, as well as instructions that write to permanent storage, are more expensive and should charge more gas. See, for example, [this list of EVM opcodes](https://www.evm.codes/) and the associated gas fees, many of which are dynamic and depend on, e.g., the amount of data read or written. We’ll implement more sophisticated gas metering in the future.

This sort of metering is pretty straightforward in an interpreter, which is how Athena works for now. In the future, if we decide to move to compiled code, we’ll need a more sophisticated approach such as injecting gas metering into the bytecode to be compiled. Fortunately our friends in the Polkadot ecosystem have made a lot of progress towards [even more sophisticated](https://hackmd.io/@pepyakin/By7783Wo5) forms of gas metering, something we’re playing close attention to.

But this simplistic gas metering is good enough for now.


# Talk<a id="talk"></a>

The other thing that happened this month was the EthCC conference week in Brussels. I attended and gave [a talk on Athena](https://ethcc.io/archive/Athena-Rises), the very first such talk (of hopefully many). I also had the opportunity to meet folks from the RiscZero team, and to meet with other RISC-V stakeholders. As expected and as I confirmed in person last week, the awareness and popularity of RISC-V is growing, especially as a better engine for blockchain programmability. I was pleasantly surprised how many people were aware of it, probably in large part due to marketing efforts on the part of RiscZero and other stakeholders. I pitched the Athena design, both the low-level VM design as well as the “[enshrined](https://tezos.stackexchange.com/questions/5835/rollup-qa-why-are-tezos-enshrined-rollup-better-than-ethereum-smart-contract) [rollup](https://www.reddit.com/r/ethereum/comments/vrx9xe/comment/if7auu7/?context=3)” design, to a number of smart founders and developers throughout the course of the week. I asked all of them to poke holes in the design, and none poked any very large holes, which is reassuring and validating. (The biggest questions involved how we intend to prove Athena execution in ZK. There are indeed a number of outstanding questions about this phase of Athena, but these are problems for a later day.)

I’ll be attending, and hopefully speaking about Athena at, Solana Breakpoint in Singapore in September, as well as Ethereum Devcon in Bangkok in November. If you intend to be at either event, let me know!


# What’s Next<a id="whats-next"></a>

The rest of this month is all about finishing up those remaining host functions, then packaging up what’s been done so far as a dynamic library so go-spacemesh can use it. We’ve already done this with dependencies like [spacemesh-sdk](https://github.com/spacemeshos/spacemesh-sdk) and [post-rs](https://github.com/spacemeshos/post-rs), both of which are written in Rust, and which [smcli](https://github.com/spacemeshos/smcli) and [go-spacemesh](https://github.com/spacemeshos/go-spacemesh/) depend on, respectively. So I don’t expect this part to take too long, although linking against foreign dependencies can [sometimes be difficult](https://github.com/spacemeshos/smcli/blob/develop/Makefile), especially when it comes to supporting different platforms.

After that, we move onto the fun work of integrating the prototype into go-spacemesh and launching it on a testnet so we can start playing with it! Back to work.
