---
eip: <to be assigned>
title: ERC: Identity Proxy Contract
author: David Ammouial (@davux) <dammouia@everis.com>
status: Draft
type: Standards Track
category: ERC
created: 2019-06-04
---

## Simple Summary
Identity contract

## Abstract
This ERC describes the interface used for identity management: creation of an identity, update of information related to an identity, and action on behalf of an identity.

## Motivation
There is a need to standardize the interface for contract-based digital identities to be used, so as to allow for interoperability between implementations.

## Definitions

**Owner**

The owner of an identity (i.e. Proxy contract) is an Ethereum address recognized by the Proxy contract to originate transactions that the Proxy contract will forward. While the address of a Proxy contract is permanent, owners may be added and removed as needed.

## Specification

This EIP defines an interface called `Proxy`, which provides the following functions:

**constructor**

The constructor is used to instantiate a new identity (i.e. Proxy contract). It takes a `firstOwner` address of that proxy.

```js
constructor(firstOwner) public
```

**forward**

Used to instruct the smart contract to execute a given function (in the form of its bytecode `data`) and/or transfer a certain `value` of ethers to a given `destination` smart contract.

```js
function forward(address destination, uint value, bytes data) public;
```

**isOwner**

Returns whether a given address is allowed to execute `forward` on the smart contract.

```js
function isOwner(address) public returns (bool);
```

**addOwner**
```js
function addOwner(address newOwner) public;
```

Used to add a `newOwner` address as an owner of the smart contract.

**renounce**

Used to give up ownership of the smart contract.

```js
function renounce() public;
```

In addition, this EIP defines the following events:

**OwnerAdded**

This event is emitted everytime a `newOwner` address is added to the contract.

```js
OwnerAdded(address indexed newOwner)
```

**OwnerRemoved**

This event is emitted everytime a `formerOwner` address is removed from the contract.

```js
OwnerRemoved(address indexed formerOwner)
```

## Rationale

For digital identity to be self-sovereign, it needs to be able to rely on a trusted, decentralized backbone. Ethereum blockchain is a satisfying component for that requisite.

## Backwards Compatibility
This EIP doesn't introduce any known backwards compatibility issues. However, it does a similar job as other ERCs such as ERC-725 and others, and there are plans to integrate the most recent developments into this EIP.

## Implementation
KayTrust Provider web application and KayTrust Wallet mobile application provide an implementation of the Proxy contract.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).