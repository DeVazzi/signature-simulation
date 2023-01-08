---
eip: <to be assigned>
title: Humanly readable offline signatures
description: A procedure for making a proposed to be signed buffer of typed structured data humanly readable.
author: Tal Be'ery (@talbeerysec), Roi Vazan (@)
discussions-to: <URL>
status: Draft
type: Standards Track
category: Interface
created: 2023-01-08
requires (*optional): 712
---

## Abstract

Abstract is a multi-sentence (short paragraph) technical summary. This should be a very terse and human-readable version of the specification section. Someone should be able to read only the abstract to get the gist of what this specification does.

## Motivation
The use case of Web3 off-chain signatures intended to be used within on-chain transaction is gaining traction and being used in multiple leading protocols (e.g. OpenSea) and standards (EIP-2612: Permit Extension for EIP-20 Signed Approvals), mainly as it offers a fee-less experience.
Attackers are known to actively and successfully abuse such off-chain signatures, leveraging the fact that users are blindly signing off-chain messages, since they are not humanly readable. 
While EIP-712 originally declared in its title that being ”humanly readable” is one of its goals, it did not live up to its promise eventually and EIP-712 messages are not humanly readable by an average user (see example below).

In this proposal we offer a secure and scalable method to bring true human readability to EIP-712 messages by leveraging their binded smart contracts.


## Specification

   EIP-712 already formally binds an off-chain signature to a contract, with the "verifyingContract" parameter. We suggest adding a “view” function ("stateMutability":"view") to such contracts, that returns a human readable description of the meaning of this specific off-chain buffer.

Using this function, wallets can submit the proposed off-chain signature to the contract and present the results to the user, allowing them to enjoy an “on-chain simulation equivalent” experience to their off-chain message.

This function will have a well known name and signature, such that there is no need for updates in the EIP-712 structure.


   /**




    * @dev Returns the expected result of the offchain message.


    */




   function evalEIP712Buffer(bytes[] memory buffer) public view virtual returns (string memory) {


      ...


   }

## Rationale


The proposed solution solves the readability issues. The incentives for keeping the descritption as accurate as possible are alligned, as the responsibility for the description is now owned by the contract, that:
knows the message meaning exactly (and probably can reuse the code that handles this message when received on chain)
Natively incentivized to provide the best explanation to prevent a possible fraud
Not involving a third party that needs to be trusted 
Maintains the fee-less customer experience as the added function is in “view” mode and does not require an on-chain execution and fees.
Maintains Web3’s composability property
### Alternative solutions 
#### Third party services:
Currently, the best choice for users is to rely on some 3rd party solutions that get the proposed message as input and explain its intended meaning to the user. This approach is:
Not scalable: 3rd party provider needs to learn all such proprietary messages
Not necessarily correct: the explanation is based on 3rd party interpretation of the original message author
Introduces an unnecessary dependency of a third party which may have some operational, security, and privacy implications.

#### Domain binding

Alternatively, wallets can bind domain to a signature to only accept EIP-712 message if it comes from a web2 domain that is included in the message, however this approach has the following disadvantages:
It breaks Web3’s composability, as now other dapps cannot interact with such messages
Does not protect against bad messages coming from the specified web2 domain, e.g. when web2 domain is hacked
Some current connector, such as walletConnect do not allow wallets to verify the web2 domain authenticity 

## Backwards Compatibility

All EIPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The EIP must explain how the author proposes to deal with these incompatibilities. EIP submissions without a sufficient backwards compatibility treatise may be rejected outright.

## Test Cases

Test cases for an implementation are mandatory for EIPs that are affecting consensus changes.  If the test suite is too large to reasonably be included inline, then consider adding it as one or more files in `../assets/eip-####/`.

## Reference Implementation

An optional section that contains a reference/example implementation that people can use to assist in understanding or implementing this specification.  If the implementation is too large to reasonably be included inline, then consider adding it as one or more files in `../assets/eip-####/`.

## Security Considerations

### The threat model:
The attack is facilitated by a rogue web2 interface (“dapp”) that provides bad parameters for an EIP-712 formatted message that is intended to be consumed by a legitimate contract. Therefore, the message is controlled by attackers and cannot be trusted, however the contract is controlled by a legitimate party and can be trusted. 

The attacker intends to use that signed EIP-712 message on-chain later on, with a transaction crafted by the attackers. (if the subsequent on-chain transaction was to be sent by the victim, then a regular transaction simulation would suffice)    

The case of a rogue contract is irrelevant, as such a rogue contract can already facilitate the attack regardless of the existence of the  EIP-712 formatted message.

Having said that, a rogue contract may try to abuse this functionality in order to send some maliciously crafted string in order to exploit vulnerabilities in wallet rendering of the string. Therefore wallets should treat this string as an untrusted input and handle its renderring it as such. 

### Analysis of the proposed solution

The explanation is controlled by the relevant contract which is controlled by a legitimate party. The attacker must specify the relevant contract address, as otherwise it will not be accepted by it. Therefore, the attacker cannot create false explanations using this method.
Please note that if the explanation was part of the message to sign it would have been under the control of the attacker and hence irrelevant for security purposes. 

Since the added functionality	to the contract has the “view” modifier, it cannot change state and  harm the existing functionalities of the contract

## Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).

