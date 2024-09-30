---
layout: post
title:  "September Project Update"
date:   2024-09-30 13:06:33 -0700
categories: athena update
permalink: /2024/09/30/september-project-update.html
---

The month of September flew by in a blur. While there wasn't as much steady progress on Athena during September due to travel and time off, we nevertheless achieved some significant milestones and continue to make good progress towards the next big milestone: launching the initial, proof of concept Athena testnet.

## Tldr

Everything we need to launch the initial, proof of concept Athena testnet is now in place on the Athena side. We merged a [major update](https://github.com/athenavm/athena/pull/74) that includes the [remainder](https://github.com/athenavm/athena/issues/75) of the [initial set of host functions](https://github.com/athenavm/athena/issues/28), a [SDK](https://github.com/athenavm/athena/issues/54) and [procedural macros](https://rettig.substack.com/p/technical-challenges-in-athena-part?open=false#%C2%A7thing-its-just-rust) used for writing Athena programs, and some other nice features such as the ability for a program to have [multiple entry points](https://github.com/athenavm/athena/issues/73).

The other big piece of news this month is that two additional developers have joined the Athena project. It took a few weeks but they're now up to speed and have begun to make [major contributions](https://github.com/athenavm/athena/pulls?q=is%3Apr+is%3Aclosed+-author%3Alrettig+) to Athena.

We also have a final proof of concept Spacemesh wallet template [implemented in Athena Rust](https://github.com/athenavm/athena/blob/main/examples/wallet/program/src/main.rs). For now it implements only two of the proposed [four wallet methods](https://community.spacemesh.io/t/proposed-athena-design/433#type-2-wallet-call-5), `spawn` and `send`, but this is sufficient for the initial testnet.

If work on the [Athena Rust code](https://github.com/athenavm/athena) appears to have slowed, it's only because we now have everything we need for the testnet, and we're totally focused on finishing the initial integration with go-spacemesh in order to launch the testnet.

## Integration

Up to now, all of the work on Athena occurred in an hermetic, self-contained fashion. To the extent that we've tested the host API, we did so using a very simplistic [mock host implementation](https://github.com/athenavm/athena/blob/6bdceaab5cf3bf9f44c42ecfda3abb4ec08002f2/interface/src/lib.rs#L393-L418). As I [mentioned last month](https://www.athenavm.org/2024/08/19/august-project-update.html#testnet), the goal of this phase of the project is to package up everything that's been done so far and get it running on a proof of concept Athena testnet. Doing so requires integrating the Athena VM into go-spacemesh (the host).

This work is off to a good start, but it's a big project. First of all, it requires rewriting the existing Spacemesh VM code to replace the monolithic "genesis VM" and make it modular. The initial portion of this work is [complete](https://github.com/spacemeshos/go-spacemesh/pull/6180), but there's a lot of work remaining to expand the existing Spacemesh VM API to encompass all of Athena's functionality. Secondly, it requires expanding the Spacemesh account model to make it compatible with Athena: for instance, in addition to what's already present in the Spacemesh account model, Athena accounts contain storage items.

## Database

For now, in the interest of launching the Athena testnet as soon as possible, we'll implement this in a straightforward, naive fashion: storage items will just be stored as an array or map under the Spacemesh account data type, using the existing account database table. However, it's highly likely that we'll need to redesign and rewrite this portion of the integration in the future since the IO associated with state access is one of the major bottlenecks on blockchain throughput for projects like Ethereum, and also to add support for light client proofs. We're inspired by the pioneering prior work being done by projects like [Erigon](https://erigon.tech/) and [Monad](https://docs.monad.xyz/technical-discussion/execution/monaddb) in this respect, and as with the rest of Athena, we'll make use of existing, mature designs and implementations wherever possible, rather than reinventing the wheel.

## Scoping

In complete transparency, we don't know exactly how much work remains to finish this initial integration work. In addition to the changes to the account model, just described, we'll likely need to change a bunch of other aspects of go-spacemesh including the node API, the transaction format, and the pipeline that constructs and processes blocks and transactions. (Fortunately, Athena does _not_ impact _most_ of go-spacemesh, including things like networking and consensus.)

go-spacemesh is a complex, largely monolithic codebase and Athena integration requires making deep changes. There's also a key question of how much time and effort we want to invest in doing things "the right way," and producing an integration that has a chance of eventually making it into production, versus writing throwaway code in the interest of shipping the Athena testnet as soon as possible. There's a big difference between maintaining a parallel Athena branch of go-spacemesh versus merging changes one by one into go-spacemesh in production. The most important part of this phase is finishing this scoping and design, a task that's underway as we speak.

Now that we have multiple core developers working on Athena and the integration, we also have to prioritize the work relative to [other critical ongoing tasks](https://x.com/teamspacemesh/status/1839657165155406004).

In the best case, I expect this integration work to take a few days to a few weeks. In the worst case, it could take weeks to months. We'll have much more clarity on the remaining steps and timeframe in the near future, and we'll continue to keep the community updated.

## What's Next

Next month is all about scoping, planning, and integration, integration, integration. We hope to share more concrete plans on the launch of the initial, proof of concept Athena testnet soon. Watch this space!
