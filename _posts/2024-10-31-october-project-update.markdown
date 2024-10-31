October was an exceptionally productive month and we made great progress on the Athena project, something which is immediately obvious on the [Athena releases page](https://github.com/athenavm/athena/releases). We merged a remarkable 84 pull requests in the past month. Read on to learn more about what we've accomplished, what we're working now, the current project status, and what comes next.

# Tldr

I can now say with confidence that we have everything we need on [the Athena side](https://github.com/orgs/athenavm/projects/2/views/1) to complete the initial integration work with go-spacemesh, and to launch the initial Athena testnet. This month we completed the last missing pieces of the puzzle, including implementing [method selectors](https://github.com/athenavm/athena/pull/147), the [`MaxSpend`](https://github.com/athenavm/athena/issues/128) and [`Verify`](https://github.com/athenavm/athena/issues/129) interface methods, as well as finalizing the [Athena payload](https://github.com/athenavm/athena/issues/131) and [transaction encoding](https://github.com/athenavm/athena/issues/172).

The initial [`go-spacemesh` integration work](https://github.com/spacemeshos/go-spacemesh/pull/6380) is also finished, and we're just working on getting the tests to pass. We don't expect this to take more than a few more days, and once that's done, there will be a code review process and then we'll launch the initial testnet.

# Testnet

While the completion of this initial integration work, and the launch of the first Athena testnet, is of course a huge milestone and should be celebrated, it's important to remind the community that the initial testnet will be extremely limited in terms of functionality, and that there's still a large amount of work to be done before Athena is ready for a mainnet launch.

The initial testnet will include only a single template, the [single sig wallet template](https://github.com/athenavm/athena/blob/main/examples/wallet/program/src/main.rs), and will only allow simple coin transfers. The primary goal of this initial testnet is just to test that everything is working end-to-end: that you can join and sync the testnet, create, sign, and broadcast an Athena transaction, and that state updates are happening correctly.

If all goes well with the initial phase of the testnet, we plan to roll out a number of additional features in further phases, such as event logs, better gas metering, multisig, and cross-contract calls. Most importantly, we'll add the ability for users to deploy their own template code.

For a sneak peak of what we plan to add, check out the recently-updated [Athena project milestones](https://github.com/athenavm/athena/milestones).

# VM

In addition to the testnet-critical features described above, we made a huge amount of progress cleaning up and improving the Athena codebase over the past month, and implementing other features that will be required for future testnet iterations. Some notable improvements include:

- Massive [cleanup of tests](https://github.com/athenavm/athena/pull/122)
- Improving [object serialization](https://github.com/athenavm/athena/pull/139)
- Improving [memory management](https://github.com/athenavm/athena/pull/150) and fixing a number of memory-related bugs
- Removing [compiled test programs](https://github.com/athenavm/athena/pull/159)
- Adding support for the [`CALL` host call to return data](https://github.com/athenavm/athena/pull/175)
- Adding our first precompile: [`ed25519` signature verification](https://github.com/athenavm/athena/pull/182)

# Integration

The main blocker for launching the initial testnet is finishing the [`go-spacemesh` integration work](https://github.com/spacemeshos/go-spacemesh/pull/6380). This work initially proceeded slowly due to the need for a lot of context switching: we'd make some progress, then realize that we needed to add something else to Athena to facilitate the integration work. Adding that would take a few days, then we'd switch back to the intregration work. But the work is moving much more quickly now that we have everything we need on the Athena side.

I mentioned [last month](https://www.athenavm.org/2024/09/30/september-project-update.html#scoping) that we weren't sure whether we wanted to attempt to deliver a production-ready integration, or whether we instead wanted to write "throwaway code" in the interest of shipping a testnet as soon as possible. After further discussion we decided on the latter course of action. We're now working on Athena in a parallel go-spacemesh branch that we don't intend ever to merge into production. We made this decision for three reasons (and please note that these apply to the _host integration_ work for Athena in go-spacemesh, _not_ to the main VM, which we do expect eventually to be used in production):

1. Expediency, as discussed. It will allow us to deliver a testnet much more quickly than if we had to thoroughly review everything, write thorough tests for everything, and generally do everything in production-ready fashion.
2. Uncertainty. A lot of the details about the later phases of Athena, most notably the rollup design and the ZK design, are still up in the air at this point in time. We don't feel that we can write final, production-ready Athena host code until more of these questions have been answered.
3. Once we have more confidence about these questions, we intend to rewrite the host code anyway. Having done this once already will undoubtedly result in much better design and implementation the second time around.

Since we chose to perform the integration work as quickly as possible, this necessarily means making as few changes as possible to the existing go-spacemesh data structures and logic. While still faster than the alternative, the integration work has taken longer than expected because it required deeper changes to go-spacemesh than we initially anticipated.

The Spacemesh mainnet VM, known as the "genesis VM" or "genvm", is designed and implemented in a way that's fundamentally incompatible with many aspects of the Athena design. The implementation makes a lot of assumptions that aren't valid in the context of Athena, such as that templates can interact with the blockchain state in an extremely limited way, and that state changes from one transaction don't have much impact upon later transactions. Refactoring the related code has turned into a big project, but the work is nearly done and there is a light at the end of the tunnel.

# What's Next

We're working diligently to finish this integration work. As mentioned above, this requires getting existing tests to pass fully, and a code review process. This will take a few days. In parallel, we'll begin putting together the testnet infrastructure, and we'll prepare other parts of the infrastructure for the testnet launch, including the node API, explorer, and web wallet. What's more, in parallel to the testnet effort, we'll continue working on improvements on the VM side to bring more features online in future testnets, as described above.

# Updates and Contributions

As always, we encourage you to follow the project on [Github](https://github.com/athenavm) and [X](https://x.com/hashtag/athenavm), and to join the conversation in the `#athena-vm` channel [on our Discord](https://chat.spacemesh.io/).

Athena is being built fully in the open and the codebase is fully open source and permissively licensed. If you're interested in contributing, take a look at the list of issues tagged [Help Wanted](https://github.com/athenavm/athena/issues?q=is%3Aissue+is%3Aopen+label%3A%22help+wanted%22) and [Good First Issue](https://github.com/athenavm/athena/issues?q=is%3Aissue+is%3Aopen+label%3A%22good+first+issue%22).
