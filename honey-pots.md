Over 250 ETH have been collected by the below honeypots of similar kind


# King of the hill

### Variable shadowing of the owner ensures it is never reassigned


```solidity
contract Ownable {
    address public owner;

    constructor() {
        owner = msg.sender;
    }

    modifier onlyOwner() {
        require(msg.sender == owner);
        _;
    }
}


contract Throne is Ownable {
    address public owner;
    uint256 public largestStake = 0;

    function Stake() payable external {
        if(msg.value > largestStake) {
            owner = msg.sender;
            largestStake = msg.value;
        }

    function withdraw() external onlyOwner {
        address(msg.sender).transfer(address(this).balance);
    }
    }
}
```


# Multiplicator

### Abuses global variable semantics of this.balance to ensure an impossible conditional branch


```
function multiplicate() payable external {
    if(msg.value >= this.balance) {
        address(msg.sender).transfer(this.balance + msg.value);
    }
}
```


# Gift Box

### Relies on abusing etherscan block explorer that does not display 0 value internal transactions that allows the owner to trap deposited ether in the contract


```solidity
bool passHasBeenSet = false;

function setPass(bytes32 hash) payable external {
    if(!passHasBeenSet && (msg.value > 1 ether)) {
        hashPass = hash;
    }
}

function getGift(bytes pass) external {
    if (hashPass == keccak256(pass)) {
        address(msg.sender).transfer(this.balance);
    }
}

function setHidden() external {
    passHasBeenSet = true;
}

```