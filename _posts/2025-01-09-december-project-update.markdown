# December Project Update

As we reflect on the final month of 2024, we wanted to share the progress made on Athena during December. Despite the shortened month due to the holiday season, we managed to achieve some significant milestones.

## Multisig Wallet Ported to Athena

The multisig wallet contract from the go-spacemesh genesis VM has been successfully ported to Athena ([PR #6528](https://github.com/spacemeshos/go-spacemesh/pull/6528)). This wallet allows transactions to be signed by multiple keys, offering enhanced functionality and flexibility for users.

## Deployment Method and Testnet Testing

We’re excited to announce that the `DEPLOY` method has been implemented and tested on the testnet. This enables the deployment of new smart contracts, marking a critical step forward for Athena.

Here we deploy the new multisig template. It received an address calculated from the hash of the code: `atest1rq5c736k86szfe50mslwukxrh457xxjl6klsl2q7ay5pv`:
![]({{site.baseurl}}/assets/tx_deploy.png)

And it is spawned at address `atest1qqqqqqqg0khlwxhgcyr9ja5qn2fwx64d8jxah7sxmytpp`:
![]({{site.baseurl}}/assets/tx_spawn_multisig.png)


## Enhancing VM Reliability

A major focus this month was improving the reliability of the VM. During testing, we identified and resolved several vulnerabilities that could crash the VM when executing crafted programs. While this is a solid start, there are still areas where improvements are needed, and we will continue to address these issues to make the VM robust against potentially malicious code.

## Optimizing Contract Binary Size

Another highlight is the significant reduction in contract binary size. For instance, we trimmed the basic wallet contract from 148 kiB to 48 kiB by eliminating the need for the ELF symbol table section. This improvement is part of [PR #264](https://github.com/athenavm/athena/pull/264). There’s still a lot to be done to further minimize binary sizes. This is important because the size of a contract template affects its cost as well as general efficiency of the VM.

## Performance Enhancements

We’ve also made strides in improving the performance of smart contracts. By optimizing the data transfer between the VM and guest programs, we halved the gas costs for the singlesig wallet. You can check out the details in [PR #267](https://github.com/athenavm/athena/pull/267).


## Looking Ahead

### VM Robustness

Our top priority remains making the VM more reliable and robust. As it will execute user-deployed contracts, potentially malicious, ensuring it doesn’t crash under any circumstances is critical. Work on this front will continue in the coming weeks.

### Public Testnet Launch

We’re gearing up to release the first public testnet in January! This testnet will allow users to spawn contracts, send funds, and deploy new smart contracts. Please note that this is a proof-of-concept (PoC) integration replacing the genesis VM on L1. It is far from the final solution, and we’re actively working on finalizing the design.

We want to set expectations: the testnet may not be fully reliable due to potential bugs, and nodes could crash. We appreciate your understanding and look forward to your feedback as we refine Athena further.

### PROXY Method Implementation

Lastly, work is ongoing to implement the `PROXY` method, which will enable contracts to call other contracts, unlocking powerful composability features.

## Contributing

As always, we encourage you to follow the project on [Github](https://github.com/athenavm) and [X](https://x.com/hashtag/athenavm), and to join the conversation in the `#athena-vm` channel [on our Discord](https://chat.spacemesh.io/).

Athena is being built fully in the open and the codebase is fully open source and permissively licensed. If you're interested in contributing, take a look at the list of issues tagged [Help Wanted](https://github.com/athenavm/athena/issues?q=is%3Aissue+is%3Aopen+label%3A%22help+wanted%22) and [Good First Issue](https://github.com/athenavm/athena/issues?q=is%3Aissue+is%3Aopen+label%3A%22good+first+issue%22).

---

Thank you for your continued support and interest in Athena. We wish you all a happy and prosperous New Year. Stay tuned for more updates in January!
