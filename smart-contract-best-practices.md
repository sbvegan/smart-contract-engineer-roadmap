# Smart Contract Best Practices

This industry is still highly experimental so the security landscape is still evolving rapidly and new best practices are developed along with it. Smart contract programming requires a different engineering mindset. One more similar to hardware or financial services programming because of the high cost of failure and difficulty to make changes.

## General Philosophy

### Prepare for Failure

There will be errors and we will need to respond to bugs and vulnerabilities gracefully. These practices may help:

- Pause the contract when things are going wrong (circuit breaker)
- Manage the amount of money at risk (rate limiting, max usage)
- Have an effective upgrade path for bugfixes and improvements

### Stay up to Date

Keep track of new security developments.

- Check your contracts for any new bug as soon as it is discovered
- Upgrade to the latest version of any tool or library as soon as possible
- Adopt new security techniques that appear useful

### KISS

Complexity increases the likelihood of errors:

- Ensure the contract logic is simple
- Modularize code to keep contracts and functions small
- Use already written tools or code where possible (stand on the shoulders of giants)
- Prefer clarity to performance whenever possible
- Only use the blockchain for the parts of you system that require decentralization

### Rolling out

Its better to catch bugs early:

- Test thoroughly and add tests whenever new attack vectors are discovered
- Provide bug bounties starting from alpha testnet releases
- Rollout in phases, w/increasing usage and testing in each phase

### Blockchain Properties

Be aware of this pitfalls:

- Be extremely careful about external contract calls, which may execute malicious code and change control flow.
- Understand that your public functions are public, and may be called maliciously and in any order.
- The private data in smart contracts is also viewable by anyone.
- Kepp gas costs and the block gas limit in mind
- Be aware that timestamps are imprecise on a blockchain, miners can influence the time of execution of a tx within a margin of several seconds.
- Randomness is non-trivial on blockchain.

### Simplicity vs. Complexity

There are always tradeoffs.

#### Rigid vs. Upgradeable

The tradeoff between malleability and security. Malleability adds to complexity and increases the potential attack surfaces.

#### Monolithic vs. Modular

Monolithic systems provide all knowledge locally identifiable and readable; which helps in some cases, like optimizing code review efficiency.

#### Duplication vs. Reuse

## Development Recommendations

### External Calls

Calls to untrusted contracts can introduce risks or errors. If its unavoidable, follow these recommendations:

**Mark untrusted contracts**

Use helpful naming schemes to make this clear

```solidity
// bad
Bank.withdraw(100); // Unclear whether trusted or untrusted

function makeWithdrawal(uint amount) { // Isn't clear that this function is potentially unsafe
    Bank.withdraw(amount);
}

// good
UntrustedBank.withdraw(100); // untrusted external call
TrustedBank.withdraw(100); // external but trusted bank contract maintained by XYZ Corp

function makeUntrustedWithdrawal(uint amount) {
    UntrustedBank.withdraw(amount);
}
```

**Avoid state changes after external calls**

Assume that malicious code might execute. One particular danger is malicious code may hijack the control flow, leading to vulnerabilities due to reentrancy.


**Don't use `transfer()` or `send()`**

Those methods forward exactly 2,300 gas to the recipient and this was to prevent reentrancy vulnerabilities, but makes the assumption that gas costs are constant. It's recommended to use `call()` instead, *BUT* this does nothing to mitigate reentrancy attacks, so other precautions must be taken. [Checks-Effects-Interaction Pattern](https://docs.soliditylang.org/en/develop/security-considerations.html?highlight=check%20effects#use-the-checks-effects-interactions-pattern)

```solidity
// bad
contract Vulnerable {
    function withdraw(uint256 amount) external {
        // This forwards 2300 gas, which may not be enough if the recipient
        // is a contract and gas costs change.
        msg.sender.transfer(amount);
    }
}

// good
contract Fixed {
    function withdraw(uint256 amount) external {
        // This forwards all available gas. Be sure to check the return value!
        (bool success, ) = msg.sender.call.value(amount)("");
        require(success, "Transfer failed.");
    }
}
```

**Handle errors in external calls**

Solidity low-level call methods on raw addresses: `address.call()`, `address.callcode()`, `address.delegatecall()`, and `address.send()` don't throw an exception, but will return `false` if one is encountered.

Make sure to account for this by checking the return value.

```solidity
// bad
someAddress.send(55);
someAddress.call.value(55)(""); // this is doubly dangerous, as it will forward all remaining gas and doesn't check for result
someAddress.call.value(100)(bytes4(sha3("deposit()"))); // if deposit throws an exception, the raw call() will only return false and transaction will NOT be reverted

// good
(bool success, ) = someAddress.call.value(55)("");
if(!success) {
    // handle failure code
}

ExternalContract(someAddress).deposit.value(100)();
```

**Favor *pull* over *push* for external calls**

External calls can fail and to minimize the damage caused by such failures its often better to isolate each external call into its own transaction. This especially relevant for payments, where it is better to let users withdraw funds rather than push funds to them automatically. Avoid combining multiple ther transfers in a single transaction.

```solidity
// bad
contract auction {
    address highestBidder;
    uint highestBid;

    function bid() payable {
        require(msg.value >= highestBid);

        if (highestBidder != address(0)) {
            (bool success, ) = highestBidder.call.value(highestBid)("");
            require(success); // if this call consistently fails, no one else can bid
        }

       highestBidder = msg.sender;
       highestBid = msg.value;
    }
}

// good
contract auction {
    address highestBidder;
    uint highestBid;
    mapping(address => uint) refunds;

    function bid() payable external {
        require(msg.value >= highestBid);

        if (highestBidder != address(0)) {
            refunds[highestBidder] += highestBid; // record the refund that this user can claim
        }

        highestBidder = msg.sender;
        highestBid = msg.value;
    }

    function withdrawRefund() external {
        uint refund = refunds[msg.sender];
        refunds[msg.sender] = 0;
        (bool success, ) = msg.sender.call.value(refund)("");
        require(success);
    }
}
```

**Don't delegatecall to untrusted code**

`delegatecall` is used to call function from other contracts as if they belong to the caller contract.


## Sources

[Ethereum Smart Contract Best Practices](https://consensys.github.io/smart-contract-best-practices/development-recommendations/)
