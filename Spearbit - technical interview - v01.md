## Missing check for contract existence

**Severity**: High 

Context: 
[`Implementation.sol#L13,L19`](https://github.com/spearbit-audits/writing-exercise/blob/3b7afa03976f98d758bf222d51546c0446080cdf/contracts/Implementation.sol) 

"The low-level functions `call`, `delegatecall` and `staticcall` return `true` as their first return value if the account called is non-existent, as part of the design of the EVM. Account existence must be checked prior to calling if needed." 

https://docs.soliditylang.org/en/v0.8.10/control-structures.html#error-handling-assert-require-revert-and-exceptions

Any wallet logic relying on these external contract calls might fail and user's funds could be lost.

**Recommendation**:

Implement a contract existence check before calling low-level functions.

*Option 1:*
Implement an isContract() function (ref. OpenZeppelin's [`Adress.sol#L36`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/Address.sol#L36)) before making  low-level calls.  Beware of its limitations.

````solidity
/**

* @dev Returns true if `account` is a contract.

*

* [IMPORTANT]

* ====

* It is unsafe to assume that an address for which this function returns

* false is an externally-owned account (EOA) and not a contract.

*

* Among others, `isContract` will return false for the following

* types of addresses:

*

* - an externally-owned account

* - a contract in construction

* - an address where a contract will be created

* - an address where a contract lived, but was destroyed

* ====

*

* [IMPORTANT]

* ====

* You shouldn't rely on `isContract` to protect against flash loan attacks!

*

* Preventing calls from contracts is highly discouraged. It breaks composability, breaks support for smart wallets

* like Gnosis Safe, and does not provide security since it can be circumvented by calling from a contract

* constructor.

* ====

*/
function isContract(address account) internal view returns (bool) {

// This method relies on extcodesize/address.code.length, which returns 0

// for contracts in construction, since the code is only stored at the end

// of the constructor execution.

return account.code.length > 0;

}
````

***

*Option 2:* 
Extend the current `require(success)` statement with an added contract check, with something similar to this?

````solidity
require(success && a.code.length > 0)
````

***

*Option 3:* 
Similar to "Don't Roll Your Own Crypto" -> "Don't Roll Your Own Proxy Implementation", but start from OpenZeppelin contracts?