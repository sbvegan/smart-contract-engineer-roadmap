# Smart Contract Best Practices

This industry is still highly experimental so the security landscape is still evolving rapidly and new best practices are developed along with it. Smart contract programming requires a different engineering mindset. One more similar to hardware or financial services programming because of the high cost of failure and difficulty to make changes.

## Prepare for Failure

There will be errors and we will need to respond to bugs and vulnerabilities gracefully. These practices may help:

- Pause the contract when things are going wrong (circuit breaker)
- Manage the amount of money at risk (rate limiting, max usage)
- Have an effective upgrade path for bugfixes and improvements

## Stay up to Date

Keep track of new security developments.

- Check your contracts for any new bug as soon as it is discovered
- Upgrade to the latest version of any tool or library as soon as possible
- Adopt new security techniques that appear useful

## KISS

Complexity increases the likelihood of errors:

- Ensure the contract logic is simple
- Modularize code to keep contracts and functions small
- Use already written tools or code where possible (stand on the shoulders of giants)
- Prefer clarity to performance whenever possible
- Only use the blockchain for the parts of you system that require decentralization

## Rolling out

Its better to catch bugs early:

- Test thoroughly and add tests whenever new attack vectors are discovered
- Provide bug bounties starting from alpha testnet releases
- Rollout in phases, w/increasing usage and testing in each phase

## Blockchain Properties

Be aware of this pitfalls:

- Be extremely careful about external contract calls, which may execute malicious code and change control flow.
- Understand that your public functions are public, and may be called maliciously and in any order.
- The private data in smart contracts is also viewable by anyone.
- Kepp gas costs and the block gas limit in mind
- Be aware that timestamps are imprecise on a blockchain, miners can influence the time of execution of a tx within a margin of several seconds.
- Randomness is non-trivial on blockchain.

## Simplicity vs. Complexity

There are always tradeoffs.

### Rigid vs. Upgradeable

The tradeoff between malleability and security. Malleability adds to complexity and increases the potential attack surfaces.

### Monolithic vs. Modular

Monolithic systems provide all knowledge locally identifiable and readable; which helps in some cases, like optimizing code review efficiency.

### Duplication vs. Reuse

## Sources

[Ethereum Smart Contract Best Practices](https://consensys.github.io/smart-contract-best-practices/development-recommendations/)
