Verifiable Credential Revocations
==================

**Specification Status:** ****Draft****

**Status Date**: September 24, 2020

**Last Revised**: October 19, 2020

**Latest Draft:**
  [workdaycredentials/specifications/revocations](https://github.com/workdaycredentials/specifications/revocations)
<!-- -->
**Editors:**
~ [Rory Martin](mailto:rory.martin@workday.com) (Workday)
~ [Gabe Cohen](mailto:gabe.cohen@workday.com) (Workday)
~ [Bjorn Hamel](mailto:bjorn.hamel@workday.com) (Workday)
~ [Lionello Lunesu](mailto:lio.lunesu@workday.com) (Workday)

**Participate:**
~ [GitHub repo](https://github.com/workdaycredentials/specifications)
~ [File a bug](https://github.com/workdaycredentials/specifications/issues)
~ [Commit history](https://github.com/workdaycredentials/specifications/commits/master)

------------------------------------

## Abstract

The [Verifiable Credentials Specification](https://w3c.github.io/vc-data-model) makes approximately twenty references to _revocations_ without defining much about how they should look or work. The specification, however, does give a few guidelines for revocations:
- Revocation by the issuer should not reveal any identifying information about the subject, the holder, the specific verifiable credential, or the verifier.
- Issuers can disclose the revocation reason.
- Issuers revoking verifiable credentials should distinguish between revocation for cryptographic integrity (for example, the signing key is compromised) versus revocation for a status change (for example, the driver’s license is suspended).
- A verifier verifies the authenticity of the presented verifiable presentation and verifiable credentials. This should include checking the credential status for revocation of the verifiable credentials.
- Issuers are urged to not use mechanisms, such as credential revocation lists that are unique per credential, during the verification process that could lead to privacy violations.
- If the credential supports revocation, use a globally-distributed service.
- Designing revocation APIs that do not depend on submitting the ID of the credential. For example, use a revocation list instead of a query.

These guidelines have led us to the following proposal for a standard data format to create Verifiable Credential Revocations. This specification also outlines considerations and concerns when evaluating different hosting models.

## Status of This Document

This document is in ****DRAFT**** form. In its current state, it is a representation of the Workday Credential Platform with limited input from others in the community.

## Terminology

Term | Definition
:--- | :---------
[Decentralized Identifier (DID)](https://w3c.github.io/did-core/) | A unique ID string and PKI metadata document format for describing the cryptographic keys and other fundamental PKI values linked to a unique, user-controlled, self-sovereign identifier in a target system (i.e. blockchain, distributed ledger).
Credential | A [W3C Verifiable Credential](https://w3c.github.io/vc-data-model/) — cryptographically secure, privacy respecting, and machine-verifiable data
Holder | The entity that has in their possession and control a Verifiable Credential
Issuer | The entity that issues Verifiable Credentials and has the ability to create Revocations
Verifier | The entity that requests data and checks a Credential's status


## Revocations

Revocations are documents that indicate a _revoked_ status of Verifiable Credentials. Revocations are a mechanism by which an issuer can update the _status_ of a Credential after issuance. Revoking a Credential results in the claims no longer being accepted as valid, regardless of the validity of any digital signatures.

Following the guidelines laid out in the Verifiable Credentials Data Model we have derived the following Revocation Data Model:

### Data Model

The Revocation data model is divided into two parts: encrypted — the revocation itself, and unencrypted — the revocation metadata. We always encrypt the `revocation` object to satisfy some of the privacy constraints and concerns outlined in the VC Spec. The only requirement of data to be unencrypted is the Revocation ID because we generate it using a one-way hash function. A key derivation function (KDF) is used to encrypt the Revocation, with the Credential ID being the input to the KDF. The logic behind this is that Credential IDs are unknown until the moment a Credential is shared. This is secure because once shared, the Verifier may wish to check the Credential's status — and only once a Credential is shared will a Verifier have the requisite information to lookup and decrypt a Revocation.

#### Type

To tie in to the [VC Data Model's credentialStatus](https://w3c.github.io/vc-data-model/#status) property this scheme is referred to as `RevocationService2020`.

#### Unencrypted Data

Field | Description
:--- | :---------
`id` | The Revocation Identifier. A [SHA-256](https://tools.ietf.org/html/rfc4634#section-4.1) hash of the Issuer DID and Credential ID. This effectively creates a namespace for the issuer, who has control over the Credential ID generation. This ensures that there cannot be a conflict even if two issuers use the same Credential ID. It is not recommended for an Issuer  to reuse a Credential ID, as all Credentials would be revoked by a single revocation. An alternate ID scheme could be (Issuer ID + "/" + Base58(SHA-256(Credential ID)), which isn't as privacy preserving for the issuer, but is verifiable by public nodes, which can see that the ID and proof are generated by the same identifier. An additional scheme allows the Identifier to be random, reducing the chance of collision and preventing attacks which attempt to calculate and claim the Identifier.
`type` | A URI property providing semantic meaning for the document (e.g. a link to this specification).
`version` | A [Semantic Version](https://semver.org/) for the data model to support backwards compatibility. 
`revokeRevealValue` | An opaque, cryptographically random value, that is the pre-image of the revocation commitment in the Credential being revoked.
`proof` | A [Linked Data Proof](https://w3c-ccg.github.io/ld-proofs/) providing authenticity from the Issuer of the Credential of the Revocation.

::: note Risk of Proof
  A proof may either be unencrypted or encrypted. If unencrypted it is recommended that the proof is not stored publicly, but rather acts as an argument to the logic that would allow a revocation to be written to a data store.

  If stored on a ledger, depending on its consensus mechanism, it may be possible to omit the `proof` field from the value recorded on ledger while still including it as a transient argument of the write request in order to determine the validity of payload.

  We recommend that the proof should never be publicly visible, since this reveals data about the issuer of the revocation.
:::

#### Encrypted Data

Field | Description
:--- | :---------
`issuerId` | The Identifier of the Issuer of the Credential being revoked.
`credentialId` | The [UUID-4](https://tools.ietf.org/html/rfc4122) Identifier of the Credential being revoked. This is **essential** — any predictible Credential ID scheme would be at risk of de-anonymization.
`revoked` | An [RFC-3339](https://tools.ietf.org/html/rfc3339) compliant date-time value indicating when the Credential was revoked.
`reasonCode` | An integer enumeration value giving information on why the Credential was revoked (e.g. "cryptographic key compromised"). This could tie in to a standard such as a [CRLReason](https://tools.ietf.org/html/rfc5280#section-5.3.1e).

### Implementation Considerations

As mentioned above, the crux of the security is that both the key and the payload are obfuscated (hash and encryption, respectively) by a randomly generated value, which is only known to those that have held the credential. Having such a value as a key greatly reduces the risks of brute force attacks, which could attempt to expose revocation data by repeatedly querrying until a hit is found. Additionally, because the data **should be** stored on a public service, the data itself needs to be obfuscated. As mentioned above, the entire revocation, with metadata, is to be encrypted using a password-based encryption scheme such as [PBKDF2](https://www.ietf.org/rfc/rfc2898.txt) with a sufficient key length, and number of salt bits and iterations.

Verifiers can use the `revokeRevealValue` to ensure the revocation matches the revocation commitment in the Credential being revoked. When obtaining revocations from a public ledger, this ensures the revocation was done by the Issuer of the Credential, since only the Issuer would have access to the pre-image of the revocation commitment. To this end, we add a `revoke_commitment` to the Credential Data Model `credentialStatus` object.

### Examples

In the examples provided below we show both unencrypted and encrypted variations of our implementation. Encrypted examples set the `revocation` property to a [Base58](https://tools.ietf.org/id/draft-msporny-base58-01.html) encoded version of the password-encrypted byte array revocation.

<tab-panels selected-index="0">

<nav>
  <button type="button">Core Revocation</button>
  <button type="button">Revocation with Metadata Unencrypted Proof</button>
  <button type="button">Revocation with Metadata Unencrypted Proof (Encrypted)</button>
  <button type="button">Revocation with Metadata Encrypted Proof</button>
  <button type="button">Revocation with Metadata Encrypted Proof (Encrypted)</button>
</nav>

<section>

::: example Core Revocation
```json
{
  "id": "7cMJ7xyvWYkHGSKpNtY2ojP6zeM83KzUhkUx3Czxs88Z",
  "credentialId": "ebf07b21-0891-458a-b642-19e1b16a001d",
  "issuerId": "did:example:RA61wuqiu6jfc4EdAPcGJF",
  "revoked": "2020-09-16T23:13:47Z",
  "reasonCode": 1
}
```
:::

</section>

<section>

::: example Revocation with Metadata Unencrypted Proof
```json
{
  "id": "FXKXbA8MCbMjzvotXRZmAXmKd1Um4DEmize15JWz2Pe9",
  "type": "https://link-to-revocation-spec.com/spec.json",
  "version": "1.0",
  "revokeRevealValue": "+8jt3fqyfuQyPZh8LaiwTIh3+XFgFpfj",
  "revocation": {
    "id": "FXKXbA8MCbMjzvotXRZmAXmKd1Um4DEmize15JWz2Pe9",
    "credentialId": "2ad5c839-5fe9-42e7-98f7-ab731d28a2e1",
    "issuerId": "did:example:SfqBc7sKNw7J5YnwPUmjkg",
    "revoked": "2020-10-13T21:06:56Z"
  },
  "proof": {
    "created": "2020-10-13T21:06:56Z",
    "proofPurpose": "assertionMethod",
    "verificationMethod": "did:example:SfqBc7sKNw7J5YnwPUmjkg#key-1",
    "nonce": "de14ff97-0b2a-46a3-9c9e-39d1f27d8caf",
    "signatureValue": "3eyS68WwX7TtwZe41V5rxfB2ZkCBpQ6vH77ZUJCUn3Nd5L5xymHLkr1iv12xuugMKnYKXMiFQqcNcKyNx475DVks",
    "type": "JcsEd25519Signature2020"
  }
}
```
:::

</section>

<section>

::: example Revocation with Metadata Unencrypted Proof (Encrypted)
```json
{
  "id": "7cMJ7xyvWYkHGSKpNtY2ojP6zeM83KzUhkUx3Czxs88Z",
  "type": "https://link-to-revocation-spec.com/spec.json",
  "version": "1.0",
  "revokeRevealValue": "+8jt3fqyfuQyPZh8LaiwTIh3+XFgFpfj",
  "revocation": "6RJqc5NPPac1PRVJwVE9xJJbKUDbZH3c4rNphvig6SUbHxDZsb9k25H8WezaQKikLfLyjCiekHRFK86WbJpout6v3KJMYq1iXS6cut4Qb4xd8QPUJCHFANibqeZmmH11hnexaUrLX7Z1KfseAUBbj8z8FbpNhQzqB9xMdu1RgUzD523xNaN4mxaWJofz1qdZQ8Vc38faqo5sUaVQHaM3itQ1UGaQVaacobj5Q2DJXKrnwSw2nJsXcTaXwYywFCE7s6UzMa9uyJ3BdZAN17Uaj77dfBGw1hMAN21uEPPRrMuUjijw3h3XKqA27pYvz3Wka1eA7vdh23RrVevoM3bo9Yhpe7soyTfzmRVkGKJDVEvi9Bn5TaaUt3u65Cjq685pMVopCD8NXNUcs2noRiZnfRaSA8cbaMrMnvdNXvc7fuZp9VK55hNqYNugCgCe3SA6HqoQJpuQrceCkBLLR8agwDTgXpmhNwu5pTJwDkQuuyrHyUbtDd8ycAycvZ4zVrmybGR8vTTK7NX2ZW8ySR7FGLEJC1YVSc1CXG2ZmZxV2mVC3Qku7LZ8dzukXZZjHFfyjKr38fdNDLmpDAjprr6StUyqas5AjcFPuJNpAdB8HL1hZ4fR7xRjjjyKHYas2J46w6sFM66Nb66hS77iF2NJkvbkFACuxGnTN4vmkLkiTQ37uGPiBXiLYpWK82w7JVsLsusBN3KcbNhyUph5ccHqn7hCcYxz7CRWowX13HbEvnAoVsudBMdLLu3FVdVpXWZ6WCLAkKA3b3ybabPi7EKMBCZTWcQ3TpsYYcqKHKbNc8xJXA6cSRw88Zw5HoUEEonB3oSdKqHTZvKhzFotZC7vmx3GYBa1fnnhyFDRq4CZzm9ezcGjSdyqrcoxcNjzWph9tFJxvS6Q6Y55JHaJfphuFBUDhDsPVrASD7ChKw2kaPAspdEa9TKk8EzMMZjFfQVKFjEdd3PHjNmG43edS1AmPK4uRdBF8c3pddVKKk7Do1MAApLBTDNCND6aexM4GjcY9ojAC9VEtev2vXDBuF3G7TH5zwp1Nsk22R2UqcDRqTDPQPNpVHuVDf46Gd",
  "proof": {
    "created": "2020-09-16T23:13:47Z",
    "verificationMethod": "did:example:RA61wuqiu6jfc4EdAPcGJF#key-1",
    "nonce": "0365b52e-8ec5-4b0a-9b03-839282d609b8",
    "proofPurpose": "assertionMethod",
    "signatureValue": "4k9P8h2fwcZKwvNzMyiaLCd429iGTiQTAPowHm3QoftscgQfbaZ2mAMVu6QUAQphRQMeWYqmk9v4up9ghjnG5HGz",
    "type": "JcsEd25519Signature2020"
  }
}
```
:::

</section>

<section>

::: example Revocation with Metadata Encrypted Proof
```json
{
  "id": "7cMJ7xyvWYkHGSKpNtY2ojP6zeM83KzUhkUx3Czxs88Z",
  "type": "https://link-to-revocation-spec.com/spec.json",
  "version": "1.0",
  "revokeRevealValue": "+8jt3fqyfuQyPZh8LaiwTIh3+XFgFpfj",
  "revocation": {
    "id": "7cMJ7xyvWYkHGSKpNtY2ojP6zeM83KzUhkUx3Czxs88Z",
    "credentialId": "ebf07b21-0891-458a-b642-19e1b16a001d",
    "issuerId": "did:example:RA61wuqiu6jfc4EdAPcGJF",
    "revoked": "2020-09-16T23:13:47Z",
    "reasonCode": 1,
    "proof": {
      "created": "2020-09-16T23:13:47Z",
      "verificationMethod": "did:example:RA61wuqiu6jfc4EdAPcGJF#key-1",
      "nonce": "0365b52e-8ec5-4b0a-9b03-839282d609b8",
      "proofPurpose": "assertionMethod",
      "signatureValue": "4k9P8h2fwcZKwvNzMyiaLCd429iGTiQTAPowHm3QoftscgQfbaZ2mAMVu6QUAQphRQMeWYqmk9v4up9ghjnG5HGz",
      "type": "JcsEd25519Signature2020"
    }
  }
}
```
:::

</section>

<section>

::: example Revocation with Metadata Encrypted Proof (Encrypted)
```json
{
  "id": "7cMJ7xyvWYkHGSKpNtY2ojP6zeM83KzUhkUx3Czxs88Z",
  "type": "https://link-to-revocation-spec.com/spec.json",
  "version": "1.0",
  "revokeRevealValue": "+8jt3fqyfuQyPZh8LaiwTIh3+XFgFpfj",
  "revocation": "BGYwtqc3ouUrgjQ7nzouZFcUrfXMwMWDwjyduo41Qr4fcoigptJ4xCRJSezMc2yMapY4QygTGHPvptY5cWFMfubYGuSm3wrabYfaN4PdMpsFNsEdB6ccz74JeSczuCewhqYdbRszjxKyXa75ek3Rv8SvDbysRVnUXSjg53VWRdb5ptxaA8Z4DMTQP8EWkD9ohAk7d9SJucxp8KJRssoxxfFjZKVLK652LsBtsH2HU6jEPezd9jbuTXCX1Lz92ZFMKezeGEb4RG482tGyG331DMYw9bYN98tZd3BfhdAMFGhM88LBrZxne6E4rLNnKCQ83hH1fgbs3hhgMsk99TgYLWZftZ4syCBWPKp1jGRKs5NbyxiC417j5fTyFCraMbiEe6ZSLBcJ5Zvb5xZMq2uuXbLtk3dRKUUyZv2vRszAYAueSqvErSN4UCBLaDrF2Qb3AEEWbccJR8B7UG2wdGXTbPyRSbdzLCKUsctyUkpxtrSMVMcdAdgXAV5sywPjmu7DL8qHGKZQcF2euMWSriYuU3ugNDRiohnb7zbU5kk4HF4Y9v3AZEDnQME5VJ643PJs9PdkwhyoGqDJmcNxY5GyCFf38z1nuH9uAx84h52wKVCdNaUX9tXE4Yge7tAtmo1CehGnsQ8FM3CabE2xZCX7qMREbBDuydMVmrhga3xHPbw8h6NwXmU7xivnQNoVYvucr1W8gJDnogFfrqS1RARBpcRW7L1b7QCBgFHhpMLRw16jfLVoCQc9eGx7JvkPd4KA7hN2SZnzDBuFoBCKEMRHhjgWa1HxVPUrHXmKCoLPosYnKnwCawvbf39EXJ4BTcehFGSSMHSzFkLjCBf1VrJguKLRJD1Lp9k9S9xyYFRaDuUZhGwtxM5gdsKdnbheFHEAa4T9z4xxy4yj6gjc6TWWevpKAUDNCCNneV7V5DEBvi8jB5ErFeDcCak7whuPrRkX1njncJgkhqcjfEc4pHdga7WXYfWJALRpZX1nzQ3LWJ8zJFpuuaXNBsZH5gGokL6sYKH5B3KNBBpcxfWSprD6AgodZ2fR5Hwkt976cwrfvizv2o5ctBfK1XX59X"
}
```
:::

</section>

</tab-panels>

### JSON Schema

::: example Revocation JSON Schema
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "oneOf": [
    {
      "properties": {
        "id": { "type": "string" },
        "type": { "type": "string" },
        "version": { "type": "string" },
        "revokeRevealValue": { "type": "string" },
        "proof": {
          "type": "object",
          "properties": {
            "type": { "type": "string" },
            "proofPurpose": { "type": "string" },
            "verificationMethod": { "type": "string" },
            "created": {
              "type": "string",
              "format": "date-time"
            }
          },
          "required": [
            "type",
            "proofPurpose",
            "verificationMethod",
            "created"
          ],
          "additionalProperties": true
        },
        "revocation": {
          "type": "object",
          "properties": {
            "id": { "type": "string" },
            "issuerId": { "type": "string" },
            "credentialId": { "type": "string" },
            "revoked": {
              "type": "string",
              "format": "date-time"
            },
            "reasonCode": { "type": "integer" }
          },
          "required": [
            "id",
            "issuerId",
            "credentialId",
            "revoked"
          ],
          "additionalProperties": false
        }
      },
      "required": [
        "id",
        "type",
        "version",
        "proof"
      ],
      "additionalProperties": false
    },
    {
      "properties": {
        "id": { "type": "string"
        },
        "type": { "type": "string" },
        "version": { "type": "string" },
        "revocation": {
          "type": "object",
          "properties": {
            "id": { "type": "string" },
            "issuerId": { "type": "string" },
            "credentialId": { "type": "string" },
            "revoked": {
              "type": "string",
              "format": "date-time"
            },
            "reasonCode": { "type": "integer" },
            "proof": {
              "type": "object",
              "properties": {
                "type": { "type": "string" },
                "proofPurpose": { "type": "string" },
                "verificationMethod": { "type": "string" },
                "created": {
                  "type": "string",
                  "format": "date-time"
                }
              },
              "required": [
                "type",
                "proofPurpose",
                "verificationMethod",
                "created"
              ],
              "additionalProperties": true
            }
          },
          "required": [
            "id",
            "issuerId",
            "credentialId",
            "revoked",
            "proof"
          ],
          "additionalProperties": false
        }
      },
      "required": [
        "id",
        "type",
        "version"
      ],
      "additionalProperties": false
    }
  ]
}
```
:::

## Storage & Resolvability

According to the [Verifiable Credentials Data Model](https://w3c.github.io/vc-data-model/) it is essential that Revocations use a _globally-distributed service for revocation_ and the _APIs...do not depend on submitting the ID of the credential_.

This specification does not make explicit recommendations for hosting, but instead outlines considerations & concerns when evaluating different hosting models. We recommend that the revocation system be distributed, hosted, and in use by multiple parties.

This requirement is a mitigation against both privacy erosion and censorship. If the revocation service is hosted by the issuer, then they are effectively able to see with whom the Holder shares their data. A neutral 3rd party platform can be contractually obligated to obfuscate this data so as to maintain the privacy of the Holder. With a single host you risk there being business incentives to selectively omit revocations for certain issuers, or leak other transaction data devaluing the trust in the resource. To have the strongest revocation infrastructure ensure that no one party can compromise its integrity.

### Mutability

Storage may be mutable or immutable. If mutable one must be aware of the risks of a party other than an Issuer altering or obstructing a Revocation. At the same time, there is a benefit of being able to simplify un-revoking a Credential which may have been revoked in error.

Immutable storage faces opposite concerns: an audit log is kept and the data is inherently tamper-proof. However, revocation's issued by mistake are not straightforward to recover from and may require credential re-issuance.

### Risks & Rewards

The following table aims to outline the risks & rewards of different storage mechanisms for this Revocation format. Combinations not listed have either not been considered or are bad ideas not worth mentioning.

Storage Type | Writes | Reads | Risks | Rewards | Notes
:----------- | :----- | :---- | :---- | :------ | :-----
Database | Permissioned | Public | Loss of audit history; general mutability risk. | Lower implementation cost. Control over writes allows data integrity risk mitigation. | This scheme requires trusting the host.
Ledger | Public | Public | Identifier namespace can be claimed by a party other than the issuer, effectively censoring an issuer's ability to revoke. This can be mitigated with different Identification schemes. | Complete transparency. | It is recommended that either the `revokeRevealValue` scheme is used or the revocations use an issuer-chosen random identifier. With a random identifier the revocatoins can be written to the ledger, but an external system is needed to provide the mapping between the credential and revocation IDs.
Ledger | Permissioned | Public | Perception of a "closed system." | Control over writes allows data integrity risk mitigation. Benefits of immutability and decentralization. Easier to be controlled by many. | Open sourcing the chain-code can increase the trust of the host(s).

## Appendix

### References
- [DID Core](https://w3c.github.io/did-core/)
- [Verifiable Credentials Data Model](https://w3c.github.io/vc-data-model/)
- [JSON Schema](https://json-schema.org/)
- [Semantic Versioning](https://semver.org/)
- [SHA-256](https://tools.ietf.org/html/rfc4634#section-4.1)
- [UUID-4](https://tools.ietf.org/html/rfc4122)
- [Linked Data Proofs](https://w3c-ccg.github.io/ld-proofs/)
- [PBKDF2](https://www.ietf.org/rfc/rfc2898.txt)