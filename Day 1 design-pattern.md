# Smart Contract Design Patterns
## 1. Factory Pattern
The Factory Pattern is used to create new contract instances programmatically. This can be useful for deploying multiple instances of a contract.

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract SimpleContract {
    uint public value;

    constructor(uint _value) {
        value = _value;
    }
}

contract Factory {
    SimpleContract[] public contracts;

    function createSimpleContract(uint _value) public {
        SimpleContract newContract = new SimpleContract(_value);
        contracts.push(newContract);
    }

    function getContracts() public view returns (SimpleContract[] memory) {
        return contracts;
    }
}

```

## 2. Proxy Pattern
The Proxy Pattern allows for the upgradeability of smart contracts by separating the logic from the storage.

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Storage {
    uint256 public value;
}

contract Logic {
    Storage public storageContract;

    constructor(address _storageAddress) {
        storageContract = Storage(_storageAddress);
    }

    function setValue(uint256 _value) public {
        storageContract.value = _value;
    }

    function getValue() public view returns (uint256) {
        return storageContract.value;
    }
}

contract Proxy {
    address public logicAddress;
    address public storageAddress;

    constructor(address _logicAddress, address _storageAddress) {
        logicAddress = _logicAddress;
        storageAddress = _storageAddress;
    }

    function setLogicAddress(address _logicAddress) public {
        logicAddress = _logicAddress;
    }

    fallback() external {
        (bool success, ) = logicAddress.delegatecall(msg.data);
        require(success, "Delegatecall failed");
    }
}

```

## 3. Circuit Breaker Pattern
The Circuit Breaker Pattern provides a way to pause the functionality of a contract in case of an emergency.

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract CircuitBreaker {
    bool private stopped = false;
    address private owner;

    constructor() {
        owner = msg.sender;
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "Not the contract owner");
        _;
    }

    modifier stopInEmergency() {
        require(!stopped, "Contract is stopped");
        _;
    }

    function toggleContractActive() public onlyOwner {
        stopped = !stopped;
    }

    function deposit() public payable stopInEmergency {
        // deposit logic
    }

    function withdraw(uint _amount) public stopInEmergency {
        // withdraw logic
    }
}

```

# 4. Checks-Effects-Interactions Pattern
The Checks-Effects-Interactions Pattern helps to avoid common security vulnerabilities like re-entrancy attacks.

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract ChecksEffectsInteractions {
    mapping(address => uint256) public balances;

    function deposit() public payable {
        balances[msg.sender] += msg.value;
    }

    function withdraw(uint256 _amount) public {
        require(balances[msg.sender] >= _amount, "Insufficient balance");

        // Check
        balances[msg.sender] -= _amount;

        // Effects
        (bool success, ) = msg.sender.call{value: _amount}("");

        // Interactions
        require(success, "Transfer failed");
    }
}

```
