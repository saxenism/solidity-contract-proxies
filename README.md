# solidity-contract-proxies
Everything you need to know about *upgrading your smart contracts*

# Migration or Social Yeet (CT Lingo)

You keep all your deployed contracts immutable and non-upgradable. So, when you actually need to modify your deployed contract's logic or add some functionality, you have to deploy a new contract and ask the users to start making their calls to this newly deployed version of your protocol's smart
contract address.

This is the truest form of upgrading your contracts, without compromising on the trustless (no onlyAdmin functions) and decentralisation facets of web3. However, as is amply clear pulling this type of an upgrade isn't really a breeze.

# Proxy Terminologies

1. The Implementation Contract
* Which has all our code of our protocol. When we upgrade, we launch a brand new implementation contract

2. The Proxy Contract
* Which points to which implementation is the "correct" one, and routes everyone's function calls to that contract

3. The User
* The make calls to the proxy

4. The admin
* This is the user(or group of users/voters) who upgrade to new implementation contracts.
  
