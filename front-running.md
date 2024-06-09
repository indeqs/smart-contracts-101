# Front running

Transactions on Ethereum are grouped together in blocks which are processed on a semi-regular interval, 12 seconds.

Before transactions are placed in blocks, they are broadcasted to the mempool where block builders can then proceed to place them as is economically optimal, i.e

> Block builders/validators are financially incentivized to inlcude transactions in a block if the transaction pays a higher gas price as the validator earns part of this gas price. A higher gas price means more money for the validator

What's important to understand here is that the mempool is PUBLIC and thus anyone can see transactions before they're added in a block, giving them the power to frontrun a transaction by placing their own transaction executing the same, or a similar, action with a higher gas price. 

This will mean that the higher gas price transaction will be put first inside the block


```solidity
contract Auction {
    address highestBidder;
    uint256 highestBid = 1 ether;

    function bid() external payable {
        require(msg.value > highestBid, "Bid is too low");

        // Refund previous bidder

        if(highestBidder != address(0)) {
            payable(highestBidder).send(highestBid);
        }

        highestBidder = msg.sender;
        highestBid = msg.value;
    }
}
```


The function `bid()` can be frontrun by an individual making him get the 1 ether




### Front-Running Attacks
Front-running are significant concerns in decentralized finance (DeFi) as issues like sandwich attacks occur.

**Example:**
1. Alice places a buy order for a token at a certain price.
2. An attacker sees Alice's transaction and places their buy order with a higher gas fee.
3. The attacker's transaction gets included first in a block, buying up the tokens at the current price.
4. Alice's transaction gets processed next, but the token price has increased due to the attacker's purchase (economics 101: when supply goes down, price goes up provided all factors stay constant)
5. The attacker sells the tokens at the higher price, making a profit at Alice's expense.

### Sandwich Attacks
A sandwich attack is a more sophisticated form of front-running where the attacker places two transactions around the victim's transaction, effectively "sandwiching" it.

**How it Works:**
1. **Observation:** The attacker monitors the mempool and identifies a large buy order from a victim.
2. **Front-Run:** The attacker submits a buy order with a higher gas price, ensuring it gets included first in a block before the victim's order. This increases the token's price.
3. The victim's buy order gets processed after the attacker's buy order. This further drives the price of the token higher up cause 2 large buy orders have been executed!
4. **Back-Run:** Immediately after, the attacker submits a sell order with another high gas price, ensuring it gets included right after the victim's buy order.
5. **Profit:** The attacker sells the tokens at the higher price caused by the both his and victim's buy order, making a profit.

**Example in Solidity:**

```solidity
// Hypothetical smart contract for a decentralized exchange
pragma solidity ^0.8.0;

contract DEX {
    mapping(address => uint256) public balances;
    uint256 public price = 1 ether;

    function buy() public payable {
        require(msg.value > 0, "Send ETH to buy tokens");
        uint256 amount = msg.value / price;
        balances[msg.sender] += amount;
        // Simulate price impact by increasing the token price
        // In reality, market forces would drive this price up automatically
        price += (msg.value / 10 ether);
    }

    function sell(uint256 amount) public {
        require(balances[msg.sender] >= amount, "Insufficient balance");
        balances[msg.sender] -= amount;
        uint256 ethAmount = amount * price;
        payable(msg.sender).transfer(ethAmount);
        // Simulate price impact by decreasing the token price
        // In reality, market forces would drive this price down automatically
        price -= (ethAmount / 10 ether);
    }
}
```

In this example, an attacker could observe a large pending buy order, front-run it to buy tokens at the current price, wait for the victim's buy order to increase the price, and then sell the tokens at the inflated price.

### Mitigation Strategies
1. **Transaction Ordering Mechanisms:** Implementing mechanisms that obfuscate transaction order or batch process transactions can reduce front-running risks.
   
2. **Commit-Reveal Schemes:** Users first commit to a transaction without revealing details, and then reveal the transaction in a second step, making it harder for attackers to predict and front-run.

3. **Transaction Timing:** Randomly delay transactions or use time-locks to make it harder for attackers to predict transaction execution.

4. **Off-Chain Solutions:** Use off-chain order matching and only settle on-chain, which can hide the order details from the public mempool.

5. **Increasing Slippage Tolerance:** Users can set a higher slippage tolerance to prevent their transactions from being affected by minor price changes due to front-running, but this comes with the risk of accepting worse prices.