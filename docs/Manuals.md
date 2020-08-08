# Identity Contracts

## Introduction

This section is intended to guide you through all the functionalities offered by the Identity contracts. First, lets recap the types of contracts and how they are related and finally explore what we can do with them.

## Considerations

In the current state contracts are available for ethereum networks.

## General assumptions

* Device: This document supposes a user/entity accessing the contract through any **device** like a computer, mobile phone, tablet, IoT device, remote server. Later in this document the term *device* refers to any of the mentioned here.

* User: The end user who creates its identity either through the IndetityManager or direclty by deploying an instance of the proxy smart contract.

## Types of contracts

Two contracts are used in order to create and manage identities.

### Proxy contract

The Proxy smart contract **represents the Identity for a specific person**. This contract Allows to:

* Set one or more ethereum addresses as the owner/owners of a proxy instance.

* Allows to forward any message to any other contract in the network where the proxy contract has been deployed.

* You are totally free to deploy a proxy contract instance without depending of any other contract. When it is done you can set an address as the owner of that contract.

* Another way to deploy a proxy instance is by using an Identity Manager contract. The identity manager is explained in the next section.

* Forward any message to any other contract. This is the main method in this contract.

Interested to see all the details about the Proxy contract?. Please refer to the following [documentation](http://developer.kaytrust.id/Specs/Proxy-Contract-ERC)

### Identity Manager contract

This contract is aimed to be used as a layer of abstraction when making actions related to your identity. Later in this document called IM. This contract is not owned or administered by anyone.

#### Authorization levels in the Identity Manager Contract

The following authorization levels are currently supported in the Identity Manager contract:

Those levels can be applied to any device with a configured private key.

* fw: Allows the device to forward messages through the IdentityManager contract.
* auth: If assigned to a certain device you can create an offline logic that includes querying to the IM contract if the device has authentication capabilities; it allows developers perform authentication processes.
* devicemanager: Allows to add a new device from which the user can access to its identity.
* admin: A user with this capability is able to transfer its proxy identity to a new IdentityManager.

#### Available actions in the Identity Manager

Those actions can be categorized in three groups

* Identity lifecycle thorugh the Identity manager
  * Identity creation
    * [reference](/Manuals/diagrams/identity-lifecycle/IM_identity_creation)
  * Register/remove devices to interact with their identities which live on the blockchain
    * [reference](/Manuals/diagrams/identity-lifecycle/IM_add_remove_device)
  * Transfer identity management to a new IdentityManager
    * [reference](/Manuals/diagrams/identity-lifecycle/IM_migration_to_new_IM)
  * [Identity recovery](/Manuals/IM/IM_recover_identity)
* Administrative actions
  * Add capabilities to a device
  * Remove capabilities to a device
  * Check capabilities a certain device
Find reference for each of those capabilities [here](/Manuals/diagrams/administration/IM_identity_administration)
  
* Forward messages through the IdentityManager: The identity Manager serves as a proxy to deliver messages to any existing contract in the netwwork when the Identity Manager contract lives.
  * [reference](/Manuals/diagrams/forward/forward)

#### Identity Manager contract deployment

* The IdentityManager contract can have many flavours. In this version we use solidity in order to make it deployable on ethereum networks.

* **As a developer who want to use the [Kaytrust solution](http://developer.kaytrust.id/) you don not need to deploy the IdentityManager contract**, instead simply consume its methods though our available Kaytrust SDKs:
  * [Kaytrust Java SDK](https://dev.azure.com/everis-peru/KayTrust/_git/KT-SDK-Java)
  