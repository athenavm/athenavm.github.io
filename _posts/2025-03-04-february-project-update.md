---
layout: post
title: "February 2025 Project Update"
date: 2025-03-04
categories: updates
---

This month’s update breaks away from the usual pattern, as we’ve done more design and less development work this month.
Specifically, we’ve fleshed out Athena’s initial featureset - the “Minimum Viable Product”.

We’ve mapped the potential solutions on the following axes:

![design axes](/assets/2025_03_04/image1.png){: style="display: block; margin: 0 auto; width: 50%;" }

Eventually we’ve settled on a solution that sacrifices some decentralization of the Athena rollup to maximize simplicity
(minimizing time to market), without hurting upgradeability to the Athena’s final form.

## The Spacemesh-Athena Stack

![high level design](/assets/2025_03_04/image2.png)

You may remember this graphic from previous updates. Today we’re going to focus on the 3 main roles involved in the
eventual Spacemesh stack:

![individual components](/assets/2025_03_04/image3.png)

**Smeshers** are what we already have on the network today. They’re in charge of running base-layer consensus:
constructing blocks of base-layer transactions and taking part in consensus protocols to enshrine these blocks, and the
order of transactions in them, in history.

**Relays** will be a new way to participate in the Spacemesh ecosystem. They’ll maintain their own mempool of Athena
transactions and create bundles of those transactions to be included in base-layer blocks. Relays cover the storage cost
of including their bundles in blocks and they get paid within Athena. The margin between the base layer storage fee and
the fee they collect from transactions is their profit.

**Executors** will be responsible for Athena transaction execution. Since Athena will start out as an optimistic rollup,
they will have to post some stake and gain execution fees from Athena transactions.

Executors will eventually have their own eligibility mechanism (like Hare and Tortoise have today). Relays will need to
maintain a gossip network to propagate transactions and have a fee-sharing mechanism. These are just a few examples of
the complexities involved in building a decentralized system like this.

## Forward Looking Simplifications

To be able to ship Athena as quickly as possible, while keeping the path open to the end-game described above, we’ve
decided to sacrifice some of the initial decentralization of Athena itself (base layer remains fully decentralized).

The launch version of Athena will have a single relay that can collect storage fees within Athena to cover the base
layer storage fee. This allows us to delay implementing the fee-sharing solution and relay-coordination, while also
allowing us to expose a single relay endpoint for sending transactions to the relay with no need for an additional
peer-to-peer network. Spacemesh will operate this initial relay. Users can still choose to submit Athena transactions
using their own relay, covering the base-layer storage fee themselves - contributing to the permissionlessness of the
network. So nobody is able to censor transactions, but using the Spacemesh-operated relay is a mere convenience.

There will also be a single pre-authorized executor node on launch day, operated by Spacemesh. This allows us to delay
implementing the executor eligibility mechanism as well as on-chain dispute resolution and slashing. We will still
provide the software for anyone to easily run a “shadow executor” and validate the output of the single executor.
Executors perform a deterministic algorithm, as they execute transactions in the order they appear in blocks generated
by the smeshers (which include bundles submitted by relays). They then publish the resulting global state. If the single
executor “cheats” and reports the wrong state, the shadow executor will detect this. While there won’t be an on-chain
malfeasance proof or monetary slashing, if Spacemesh is caught cheating it will ruin our reputation and achieve nothing.

![simplified individual components](/assets/2025_03_04/image4.png)

The rest of the design remains as initially planned and there’s a clear path for us to upgrade from the initial version
to the ultimate end-game.

## Looking Ahead

We plan to start prototyping the individual components (the relay and the executor) as soon as we release atx-merge and
node-split. We hope to have the basic minium implementation ready as fast as possible, run a devnet and iterate from
there adding more features and security guardrails iteratively.

## Contributing

Athena is being built fully in the open, and the codebase is fully open source and permissively licensed. If you’re
interested in contributing, take a look at the list of issues tagged `Help Wanted` and `Good First Issue` on
[GitHub](https://github.com/athenavm).

---

Thank you for your continued support and interest in Athena. As always, we welcome your feedback and encourage you to
experiment with the devnet. Please to follow the project on [GitHub](https://github.com/athenavm) and
[X](https://x.com/hashtag/athenavm), and to join the conversation in the
[#athena-vm](https://discord.com/invite/yVhQ7rC) channel on our Discord to stay up to date with our developments.

Stay tuned for more updates next month!
