# Beware of `msg.sender.code.length` for EOA Checks

Avoid using the condition `msg.sender.code.length` to check whether the caller is an Externally Owned Account (EOA). Although it seems logical at first glance - since only smart contracts have bytecode - it's not a reliable method and can be bypassed. Specifically, while a contract is in its constructor phase, its bytecode has not been deployed yet. As a result, `msg.sender.code.length` will return 0, making the contract appear as an EOA. 

Consider the following `OnlyEOA` contract, which restricts the `onlyEOA` function access to EOAs using `msg.sender.code.length == 0` statement:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.27;

contract OnlyEOA {
    error ONLY_EOA();

    mapping(address => bool) processedBy;

    function onlyEOA() public {
        require(msg.sender.code.length == 0, ONLY_EOA());

        processedBy[msg.sender] = true;
    }
}
```

Now, here's a contract that will impersonate itself as an EOA by calling the `onlyEOA` function during its constructor phase:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.27;

interface IOnlyEOA {
    function onlyEOA() external;
}

contract NotEOA {
    constructor(address _onlyEOA) {
        IOnlyEOA(_onlyEOA).onlyEOA();
    }
}
```

Using Foundry, we can write a test to demonstrate how this condition can be exploited:

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.27;

import {Test, console} from "forge-std/Test.sol";
import {OnlyEOA} from "../src/OnlyEOA.sol";
import {NotEOA} from "../src/NotEOA.sol";

contract OnlyEOATest is Test {
    OnlyEOA public onlyEOA;
    NotEOA public notEOA;

    function setUp() public {
        onlyEOA = new OnlyEOA();
    }

    function test_onlyEOA() public {
        notEOA = new NotEOA(address(onlyEOA));

        assertTrue(onlyEOA.processedBy(address(notEOA)));
    }
}
```

When you run the test (`forge test --match-contract OnlyEOATest --match-test test_onlyEOA`), the output confirms that the check is ineffective:

```
[PASS] test_onlyEOA() (gas: 98725)
Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 817.44µs
```

This proves that the `NotEOA` contract successfully bypassed the EOA check by calling `onlyEOA` in its constructor.

Additionally, as stated in the [OpenZeppelin Docs](https://docs.openzeppelin.com/contracts/3.x/api/utils#Address-isContract-address-):


>It is unsafe to assume that an address for which this function returns false is an externally-owned account (EOA) and not a contract.
> 
> Among others, isContract will return false for the following types of addresses:
> - an externally-owned account
> - a contract in construction
> - an address where a contract will be created
> - an address where a contract lived, but was destroyed

A safer method to ensure the caller is an EOA is, for example, to compare `msg.sender` with `tx.origin`, as only EOAs can be the origin of a transaction. However, be aware that this approach has its limitations in systems with multiple contracts interacting, so use it with caution.
