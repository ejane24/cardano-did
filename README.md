# Cardano DID method (Draft)

## Introduction
Cardano is a public blockchain platform. It is open-source and decentralized, with consensus achieved using proof of stake. It can facilitate peer-to-peer transactions with its internal cryptocurrency, Ada. Cardano DID method is a proposed implementation of DID for the Cardano Blockchain and provides a concrete specification for Cardano-specific DID. This method is valid for both the Cardano mainnet and testnet.

## Cardano DID Scheme
The ABNF notation for DIDs used in the Cardano DID method is as follows:

```
cardano-did = "did:cardano:" + vKeyHash
vKeyHash = cardano-cli address key-hash --verification-key-file <vKey file>
```

### Address Generation

Cardano DID is intended to use the same key pair as an account/wallet on the Cardano Blockchain. It is possible to create a Cardano DID address without considering any wallets related to the key pair. When generating a new DID based on a Cardano blockchain account, one can execute either of the following procedures;

1. Generate a new Cardano blockchain key pair according to [Creating keys and addresses](https://developers.cardano.org/docs/stake-pool-course/handbook/keys-addresses/)
2. Use keys from an existing Cardano blockchain account.

### Examples

The following are valid Cardano DIDs:

```
did:cardano:d167e80867ac557471f19763408029b8db1a4fc88656e544877c6bf7
did:cardano:16b03e96f3a50cf7704a79390f84f65b527e62773f5d078df9b2b7f5
```

The following are not valid Cardano DIDs:

```
did:cardano:61758b7824094606971dc5134860c9dea36e63bdc1e6837b01a439 (length error)
```

## DID Document

### Representation

In the Cardano DID method, a DID document is represented in the JSON-LD format. When represented in JSON-LD format, the `@context` property must have the following entry:

```
"@context": "https://www.w3.org/ns/did/v1"
```

### Properties

A Cardano DID document must have the following top-level properties:
* `id` - A Cardano DID address
* `verificationMethod`
* `authentication`

There can also be the following optional top-level properties:
* `controller`
* `assertionMethod`

#### Verification Method
Verification Method is represented by a `verificationMethod` property. Since the Cardano blockchain account uses private (signing) and public (verification) key pairs, `verificationMethod` property must have at least one entry. the `publicKeyHex` field should be filled with the CBOR hex value in the verification key file. An example of this property is given below;

```
"verificationMethod": [{
  "id": "did:cardano:vkeyHash",
  "type": "Ed25519VerificationKey2018",
  "controller": "did:cardano:vkeyHash",
  "publicKeyHex": "CBORHex"
}]
```

## CRUD (Operations)

### Create (Register)

In order to register a Cardano DID, the account must transmit a transaction on the blockchain containing the following metadata;

```
{
  "931": {
    "target": "did:cardano:vKeyHash",
    "document": {
      "@context": "https://www.w3.org/ns/did/v1",
      "id": "did:cardano:d167e80867ac557471f19763408029b8db1a4fc88656e544877c6bf7",
      "controller": "did:cardano:vKeyHash"
      "verificationMethod": [{
        "id": "did:cardano:vkeyHash",
        "type": "Ed25519VerificationKey2018",
        "controller": "did:cardano:vkeyHash",
        "publicKeyHex": "CBORHex"
      }],
      "authentication": [
        "did:cardano:d167e80867ac557471f19763408029b8db1a4fc88656e544877c6bf7"
      ],
      "assertionMethod": [
        "did:cardano:d167e80867ac557471f19763408029b8db1a4fc88656e544877c6bf7"
      ]
    }
  }
}
```

This transaction must be signed by the signing key corresponding to the verification key associated with the Cardano DID.

### Read

Reading a DID document can be achieved using one of the various Cardano blockchain explorers or by using Cardano DB-Sync with the following query;

```
SELECT * from tx_metadata 
inner join tx_out 
  on tx_metadata.tx_id=tx_out.tx_id
where
  key='931'
and
  '\x'||substring(json->'target' from 13)=tx_out.payment_cred
order by tx_out.tx_id desc
limit 1;
```

This returns the most recent valid version of the DID document.

### Update (Replace)

In order to update a document, simply sumbit a new transaction in the same format as the Create operation with the new updated document. This must be done by the controller account.

### Deactivate (Revoke)

In order to revoke a DID document, simply trasmit a new transaction with the following metadata;

```
{
  "931": {
    "target": "did:cardano:vKeyHash"
  }
}
```

## Security and Privacy Considerations

### Sybil Attack
An attacker must solve a computationally hard problem in order to forge a private key of the corresponding account in order to register a DID document for an arbitrary DID.

### Unauthorized Update
An attacker must solve a computationally hard problem in order to forge a private key of the corresponding account in order to update a DID document for an arbitrary DID.


## Current Drawbacks
Many wallets on the cardano blockchain have multiple addresses and hence public/private keys. This method only takes into account a single key pair and as such a unique DID could be created for each address associated with an account.
