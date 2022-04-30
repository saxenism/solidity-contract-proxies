# solidity-contract-proxies
Everything you need to know about *upgrading your smart contracts*

# Migration or Social Yeet (CT Lingo)

You keep all your deployed contracts immutable and non-upgradable. So, when you actually need to modify your deployed contract's logic or add some functionality, you have to deploy a new contract and ask the users to start making their calls to this newly deployed version of your protocol's smart
contract address.

This is the truest form of upgrading your contracts, without compromising on the trustless (no onlyAdmin functions) and decentralisation facets of web3. However, as is amply clear pulling this type of an upgrade isn't really a breeze.

# Proxies

## Proxy Terminologies

1. The Implementation Contract
* Which has all our code of our protocol. When we upgrade, we launch a brand new implementation contract.

2. The Proxy Contract
* Which points to which implementation is the "correct" one, and routes everyone's function calls to that contract

3. The User
* The make calls to the proxy

4. The admin
* This is the user(or group of users/voters) who upgrade to new implementation contracts.

## Storage Variables

The cool thing about proxies and delegate call is that all our storage variables are going to be stored in the proxy contract and not in the implementation contract. Therefore, when you deploy a new contract and your proxy starts pointing to this new contract, you don't have to migrate the data from the old contract as it is already in the proxy contract.

## Issues with proxies

### 1. Storage Clashes

When we do `delegateCall` from the proxyContract, we do the logic of implementationContract inside the proxyContract.

In Solidity storage layout begins at position 0 and increments for each new state variable. A proxy contract and its delegate/logic contracts share the same storage layout!

Here is an example to illustrate the problem. ProxyA defines two state variables, facetA and owner .
```
contract ProxyA {
  address facetA;  
  address owner;
  constructor() public {
    owner = msg.sender;
    facetA = 0x0b22380B7c423470979AC3eD7d3c07696773dEa1;
  }
  fallback() external payable {
    address facetAddress = facetA;
    assembly {
      ... code omitted for simplicity
    }
  }
}
```

FacetA declares one state variable.
```
contract FacetA {
  address user;
  function getUser() external view returns(address) {
    return user;
  }
  function setUser(address newUser) external {
    user = newUser;
  }
}
```

ProxyA delegates function calls to FacetA. The problem is that, when delegating, ProxyA and FacetA share the same storage layout. The state variable facetA is at position 0. And The state variable user is also at position 0. So if the setUser function is called it will set user and facetA to the newUser value, which is obviously not the intention, the intention just being to set user only.

So, basically, you can only ever append variables to your implementation contract and not really change or re-order the old ones.

Now,ideally if you called `setValue` using delegateCall from your proxy contract, it should set the value of `differentValue` but it sets the value 

### 2. Function Selector Clashes
 
 Function Selector: A 4 byte hash of a function name and function signature that define a function.
 
 Now it is possible that a function in the implementation contract has the same function selector as an admin function in the proxy contract. Think of a scenario where `function getPrice` in the implementationContract has the same Function Selector as `function destroyProxy`. Definitely would not be good.
 
 For example, the following two functions have the exact same function selector
 ```
 function collate_propagate_storage(bytes16) external{}
 function burn(uint256) external {}
 ```
   
 Contract to verify this statement is as follows and also can be found on this link: https://gist.github.com/saxenism/02af4e7a7fdb42801157571a3dab2c05
```
// SPDX-License-Identifier: MIT

pragma solidity >=0.7.0 <0.9.0;

contract ProxyExperimentation {

    function getFunctionSignature(string memory signature) internal pure returns (bytes4) {
        return bytes4(keccak256(bytes(signature)));
    }

    function checkFunctionSelectorSimilarity() external pure returns (bool) {
        bytes4 functionSelector1 = getFunctionSignature("collate_propagate_storage(bytes16)");
        bytes4 functionSelector2 = getFunctionSignature("burn(uint256)");

        bool success = (functionSelector1 == functionSelector2);
        return success;
    }

    function verifyOtherFunctionDisimilarity() external pure returns (bool) {
        bytes4 functionSelector1 = getFunctionSignature("collate_propagate_storage(bytes16)");
        bytes4 functionSelector2 = getFunctionSignature("burnn(uint256)");

        bool result = (functionSelector1 == functionSelector2);
        return result;
    }
}
```
## Proxy Patterns

### 1. Transparent Proxy Pattern

In this pattern, the admin can call only call the admin functions in the proxy contract and the users can only call the functions in the implementation contract. Admin functions are the functions that govern the upgrades.

This way, as an admin or as a user you can't mix up functions from different contracts with the same function selector and no problems should occur.
As a side note, now the admin cannot participate in their own DeFi protocol :P

### 2. Universal Upgradable Proxies

Admin Only functions are kept in the implementation contract itself, instead of the proxy. 

The advantage here is, if that happens, and two functions have the same function selector, then the Solidity compiler will let us know.
Also, since there is one less read that we have to do, we save on gas costs.

So, it is necessary here that you implement the upgradation functions in the implementation contract, because otherwise you would be stuck and we get back to the YEET method

### 3. Diamond Pattern

Allows for multiple implementation contracts. Is probably the best method to implement upgradable contracts, since it allows you to make granular changes/upgrades, but it can get really complex. So to use this, you need to be really really good at smart contract development.
