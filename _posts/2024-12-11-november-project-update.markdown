# November Athena update

November was all about finishing up the remaining initial integration work, and shipping the initial, alpha testnet.

## Tldr

A month ago, the initial integration work had just been completed and we were just working on getting the tests to pass. That work was completed in early November. The rest of the month was spent on three parallel work streams: 1. code reviewing the initial integration work, 2. launching the first testnet, and 3. adding support for Athena transactions and the Athena testnet to the Spacemesh web wallet.

As of today, the second and third are done, and we've made great progress on the first.

## Testnet

The launch wasn't without a few hiccups, but our devops team successfully launched the first Athena testnet in mid-November. As mentioned many times before, this initial testnet only allows two types of transactions: spawning a wallet and sending coins. We're excited to report that these transactions are running successfully on the testnet.

We are working on adding support for more kinds of transactions, in particular, deploying new programs. This work work is very much R&D and requires doing some breaking changes in the process. This is why we haven't released a public testnet yet. We plan to create a public network once we moreless stabilize the format of transactions.

## Wallet and Explorer

A testnet isn't just the invisible, backend infrastructure—Athena itself, the go-spacemesh integration, the testnet configuration, etc. It also needs a frontend. Our frontends are the [Spacemesh web wallet](https://wallet.spacemesh.io/) and the [Testnet explorer](https://explorer-devnet-athena.spacemesh.network/overview).

These needed to be updated to support the Athena transaction format, which is a bit different than the OG Spacemesh transaction format. This work was done in November, and you can now make Athena testnet transactions using the web wallet, and see these transactions in the explorer.

The explorer can already parse and show Athena transactions executed on the testnet. Below are a few screenshots from the explorer showing TXs and accounts

#### Spawn transaction

![]({{site.baseurl}}/assets/tx_spawn.png)

#### Spend transaction

![]({{site.baseurl}}/assets/tx_spend.png)

![]({{site.baseurl}}/assets/account_received_spend.png)

#### Coinbase receiving rewards for smeshing

![]({{site.baseurl}}/assets/account_rewards.png)

## What's Next

While the initial integration work is done and the testnet is running, much work remains to continue to develop Athena to be more useful and production-ready. The code review of the integration work is ongoing, and the code has already been cleaned up considerably. The next major feature that's being added to Athena is the ability for users to deploy their own programs. This will make Athena much more useful and much more interesting. We hope to include this in the next testnet release.

You can track progress towards future releases on the [Github milestones](https://github.com/athenavm/athena/milestones) page.

## Contributing

As always, we encourage you to follow the project on [Github](https://github.com/athenavm) and [X](https://x.com/hashtag/athenavm), and to join the conversation in the `#athena-vm` channel [on our Discord](https://chat.spacemesh.io/).

Athena is being built fully in the open and the codebase is fully open source and permissively licensed. If you're interested in contributing, take a look at the list of issues tagged [Help Wanted](https://github.com/athenavm/athena/issues?q=is%3Aissue+is%3Aopen+label%3A%22help+wanted%22) and [Good First Issue](https://github.com/athenavm/athena/issues?q=is%3Aissue+is%3Aopen+label%3A%22good+first+issue%22).
