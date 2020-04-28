# Notice
This document is no longer updated internally. It is being maintained in [IMS Global here](https://github.com/IMSGlobal/openbadges-specification/issues/258).

# DID Based Identity in Open Badges
- Authors: 
  - Gabe Cohen [gabe.cohen@workday.com](mailto:gabe.cohen@workday.com)
  - Rory Martin [rory.martin@workday.com](mailto:rory.martin@workday.com)
  - Bjorn Hamel [bjorn.hamel@workday.com](mailto:bjorn.hamel@workday.com)
- Last updated: 2020-01-14

## Status
- Status: **DRAFT**
- Status Date: 2019-01-13

## Abstract
The [Open Badges v2.0](https://www.imsglobal.org/sites/default/files/Badges/OBv2p0Final/index.html) specification references identities in a number of places. Namely, in regard to _issuers_ and _recipients_ of badges. Issuers are represented by [Profiles](https://www.imsglobal.org/sites/default/files/Badges/OBv2p0Final/index.html#Profile), while badge recipients are identified using an [Identity Object](https://www.imsglobal.org/sites/default/files/Badges/OBv2p0Final/index.html#IdentityObject). The valid _types_ of properties that can serve as identifiers for individuals or organizations in the specification are identified as [Profile Identifier Properties](https://www.imsglobal.org/sites/default/files/Badges/OBv2p0Final/index.html#ProfileIdentifierProperties). This proposal intends to expand the set of properties to include [Decentralized Identifiers (DIDs)](https://w3c.github.io/did-core/).

## Contents
  * [Abstract](#abstract)
  * [Contents](#contents)
  * [Specification](#specification)
    - [Identifiers in Open Badges](#identifiers-in-open-badges)
    - [Primer on DIDs](#primer-on-dids)
    - [DIDs as Open Badge Identifiers](#dids-as-open-badge-identifiers)
    - [Resolvability of DIDs](#resolvability-of-dids)
  * [Drawbacks](#drawbacks)
  * [Prior Art](#prior-art)
  * [Security & Privacy Considerations](#security--privacy-considerations)
  * [Interoperability Considerations](#interoperability-considerations)
  * [Implementation Notes](#implementation-notes)
  * [References](#references)

## Specification

### Identifiers in Open Badges
[Profile Identifier Properties](https://www.imsglobal.org/sites/default/files/Badges/OBv2p0Final/index.html#ProfileIdentifierProperties), as outlined in the specification, are "value[s] of a specific property unique to [an] individual or organization." These properties are of a string data type and are to be used in Profiles and in identifying recipients of an Assertion.

At the time of this draft, the specification identifies _email_, _url_, and _telephone_ as the only "serviceable identifiers."

### Primer on DIDs
[Decentralized Identifiers](https://w3c.github.io/did-core), or DIDs, "are a new type of identifier to provide verifiable, decentralized digital identity." In brief, DIDs are URLs, often backed by a distributed ledger, that can be resolved to a "DID Document" containing cryptographic key material of which the "controller" can prove ownership. The identity provides a mechanism for trustable interactions with the subject of a DID.

DIDs are a key piece of decentralized infrastructure that are called out specifically in the [Verifiable Credentials Data Model](https://www.w3.org/TR/vc-data-model/) and [Identity Hub](https://github.com/decentralized-identity/identity-hub/blob/master/explainer.md) specifications among many others in the Self Sovereign and Digital Identity space.

An example DID document, [taken from the spec](https://w3c.github.io/did-core/#example-2-minimal-self-managed-did-document), is provided below:
```
{
  "@context": "https://www.w3.org/ns/did/v1",
  "id": "did:example:123456789abcdefghi",
  "authentication": [{
    "id": "did:example:123456789abcdefghi#keys-1",
    "type": "RsaVerificationKey2018",
    "controller": "did:example:123456789abcdefghi",
    "publicKeyPem": "-----BEGIN PUBLIC KEY...END PUBLIC KEY-----\r\n"
  }],
  "service": [{
    "id":"did:example:123456789abcdefghi#vcs",
    "type": "VerifiableCredentialService",
    "serviceEndpoint": "https://example.com/vc/"
  }]
}
```

A corresponding example Verifiable Credential, where the subject (who the credential is issued to) is a DID:

```
{
  "@context": [
    "https://www.w3.org/2018/credentials/v1",
    "https://www.w3.org/2018/credentials/examples/v1"
  ],
  "id": "http://example.edu/credentials/3732",
  "type": ["VerifiableCredential", "UniversityDegreeCredential"],
  "credentialSubject": {
    "id": "did:example:123456789abcdefghi",
    "degree": {
      "type": "BachelorDegree",
      "name": "Bachelor of Science and Arts"
    }
  },
  "proof": { ... }
}
```

### DIDs as Open Badge Identifiers
DIDs are URLs, and as such can fit into the Open Badge specification as it stands; however, the authors believe it is of value to differentiate between identifiers and URLs. DIDs are a special class of identifiers that have specific semantic meaning supporting cryptographically backed identity and should be treated differently than traditional webpages or other URLs. Having a separate `did` property provides necessary context on how to deal with the `did` property, and makes it simpler for clients to parse badges.

With the addition of the `did` property to the [Profile Identifier Properties](https://www.imsglobal.org/sites/default/files/Badges/OBv2p0Final/index.html#ProfileIdentifierProperties) list, a sample badge could appear as in the following example:

```
{
  "@context": "https://w3id.org/openbadges/v2",
  "type": "Assertion",
  "id": "https://example.org/beths-robotics-badge.json",
  "recipient": {
    "type": "did",
    "hashed": false,
    "identity": "did:work:H3C2AVvLMv6gmMNam3uVAjZpfkcJCwDwnZn6z3wXmqPV"
  },
  "image": "https://example.org/beths-robot-badge.png",
  "evidence": "https://example.org/beths-robot-work.html",
  "issuedOn": "2016-12-31T23:59:59Z",
  "expires": "2017-06-30T23:59:59Z",
  "badge": "https://example.org/robotics-badge.json",
  "verification": {
    "type": "hosted"
  }
}
```

Including a DID as a possible `did` value in a [Profile](https://www.imsglobal.org/sites/default/files/Badges/OBv2p0Final/index.html#Profile), Extending the [Issuer/Profile Example](https://www.imsglobal.org/sites/default/files/Badges/OBv2p0Final/examples/index.html#Issuer) in the Open Badges specification, we can see DID based issuers and recipients:

```
{
  "@context": "https://w3id.org/openbadges/v2",
  "type": "Assertion",
  "id": "https://example.org/beths-robotics-badge.json",
  "recipient": {
    "type": "did",
    "hashed": false,
    "identity": "did:work:H3C2AVvLMv6gmMNam3uVAjZpfkcJCwDwnZn6z3wXmqPV"
  },
  "issuedOn": "2016-12-31T23:59:59Z",
  "badge": {
    "id": "https://example.org/robotics-badge.json",
    "type": "BadgeClass",
    "name": "Awesome Robotics Badge",
    "description": "For doing awesome things with robots that people think is pretty great.",
    "image": "https://example.org/robotics-badge.png",
    "criteria": "https://example.org/robotics-badge.html",
    "issuer": {
      "type": "Profile",
      "id": "did:work:2UUHQCd4psvkPLZGnWY33L",
      "name": "An Example Badge Issuer",
      "image": "https://example.org/logo.png",
      "url": "https://example.org",
      "email": "steved@example.org",
      "publicKey": "did:work:2UUHQCd4psvkPLZGnWY33L#key-1"
    }
  },
  "verification": {
    "type": "signed",
    "creator": "did:work:2UUHQCd4psvkPLZGnWY33L#key-1"
  }
}
```

When using the `signedBadge` verification type, if the issuer is DID-based, it is recommended that a corresponding signature from key material resolvable in the associated DID Document is used to sign the badge. The issuer's profile may contain the key reference in the `publicKey` property, though it is to be understood that the key is accessible through the DID Document associated with the `id` property in the Profile.

Expanding upon the provided [CryptographicKey Example](https://www.imsglobal.org/sites/default/files/Badges/OBv2p0Final/examples/index.html#CryptographicKey) outlined in the Open Badges specification we can see the ID of a key being a fully-qualified key reference from a DID Document:

```
{
  "@context": "https://w3id.org/openbadges/v2",
  "type": "CryptographicKey",
  "id": "did:work:2UUHQCd4psvkPLZGnWY33L#key-1",
  "owner": "https://example.org/organization.json",
  "publicKeyPem": "-----BEGIN PUBLIC KEY-----MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCqGKukO1De7zhZj6+H0qtjTkVxwTCpvKe4eCZ0FPqri0cb2JZfXJ/DgYSF6vUpwmJG8wVQZKjeGcjDOL5UlsuusFncCzWBQ7RKNUSesmQRMSGkVb1/3j+skZ6UtW+5u09lHNsj6tQ51s1SPrCBkedbNf0Tp0GbMJDyR4e9T04ZZwIDAQAB-----END PUBLIC KEY-----\n"
}
```

Examples of keys and their references are provided in the [DID Core Public Keys section](https://w3c.github.io/did-core/#public-keys).

Using the sample Open Badge above, the following JWT is generated, signed with the key `did:work:H3C2AVvLMv6gmMNam3uVAjZpfkcJCwDwnZn6z3wXmqPV#key-1`:

```
eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJkaWQ6d29yazoyVVVIUUNkNHBzdmtQTFpHbldZMzNMIiwiYmFkZ2UiOiJld29nSUNKQVkyOXVkR1Y0ZENJNklDSm9kSFJ3Y3pvdkwzY3phV1F1YjNKbkwyOXdaVzVpWVdSblpYTXZkaklpTEFvZ0lDSjBlWEJsSWpvZ0lrRnpjMlZ5ZEdsdmJpSXNDaUFnSW1sa0lqb2dJbWgwZEhCek9pOHZaWGhoYlhCc1pTNXZjbWN2WW1WMGFITXRjbTlpYjNScFkzTXRZbUZrWjJVdWFuTnZiaUlzQ2lBZ0luSmxZMmx3YVdWdWRDSTZJSHNLSUNBZ0lDSjBlWEJsSWpvZ0ltUnBaQ0lzQ2lBZ0lDQWlhR0Z6YUdWa0lqb2dabUZzYzJVc0NpQWdJQ0FpYVdSbGJuUnBkSGtpT2lBaVpHbGtPbmR2Y21zNlNETkRNa0ZXZGt4TmRqWm5iVTFPWVcwemRWWkJhbHB3Wm10alNrTjNSSGR1V200MmVqTjNXRzF4VUZZaUNpQWdmU3dLSUNBaWFYTnpkV1ZrVDI0aU9pQWlNakF4TmkweE1pMHpNVlF5TXpvMU9UbzFPVm9pTEFvZ0lDSmlZV1JuWlNJNklIc0tJQ0FnSUNKcFpDSTZJQ0pvZEhSd2N6b3ZMMlY0WVcxd2JHVXViM0puTDNKdlltOTBhV056TFdKaFpHZGxMbXB6YjI0aUxBb2dJQ0FnSW5SNWNHVWlPaUFpUW1Ga1oyVkRiR0Z6Y3lJc0NpQWdJQ0FpYm1GdFpTSTZJQ0pCZDJWemIyMWxJRkp2WW05MGFXTnpJRUpoWkdkbElpd0tJQ0FnSUNKa1pYTmpjbWx3ZEdsdmJpSTZJQ0pHYjNJZ1pHOXBibWNnWVhkbGMyOXRaU0IwYUdsdVozTWdkMmwwYUNCeWIySnZkSE1nZEdoaGRDQndaVzl3YkdVZ2RHaHBibXNnYVhNZ2NISmxkSFI1SUdkeVpXRjBMaUlzQ2lBZ0lDQWlhVzFoWjJVaU9pQWlhSFIwY0hNNkx5OWxlR0Z0Y0d4bExtOXlaeTl5YjJKdmRHbGpjeTFpWVdSblpTNXdibWNpTEFvZ0lDQWdJbU55YVhSbGNtbGhJam9nSW1oMGRIQnpPaTh2WlhoaGJYQnNaUzV2Y21jdmNtOWliM1JwWTNNdFltRmtaMlV1YUhSdGJDSXNDaUFnSUNBaWFYTnpkV1Z5SWpvZ2V3b2dJQ0FnSUNBaWRIbHdaU0k2SUNKUWNtOW1hV3hsSWl3S0lDQWdJQ0FnSW1sa0lqb2dJbVJwWkRwM2IzSnJPakpWVlVoUlEyUTBjSE4yYTFCTVdrZHVWMWt6TTB3aUxBb2dJQ0FnSUNBaWJtRnRaU0k2SUNKQmJpQkZlR0Z0Y0d4bElFSmhaR2RsSUVsemMzVmxjaUlzQ2lBZ0lDQWdJQ0pwYldGblpTSTZJQ0pvZEhSd2N6b3ZMMlY0WVcxd2JHVXViM0puTDJ4dloyOHVjRzVuSWl3S0lDQWdJQ0FnSW5WeWJDSTZJQ0pvZEhSd2N6b3ZMMlY0WVcxd2JHVXViM0puSWl3S0lDQWdJQ0FnSW1WdFlXbHNJam9nSW5OMFpYWmxaRUJsZUdGdGNHeGxMbTl5WnlJc0NpQWdJQ0FnSUNKd2RXSnNhV05MWlhraU9pQWlaR2xrT25kdmNtczZNbFZWU0ZGRFpEUndjM1pyVUV4YVIyNVhXVE16VENOclpYa3RNU0lLSUNBZ0lIMEtJQ0I5TEFvZ0lDSjJaWEpwWm1sallYUnBiMjRpT2lCN0NpQWdJQ0FpZEhsd1pTSTZJQ0p6YVdkdVpXUWlMQW9nSUNBZ0ltTnlaV0YwYjNJaU9pQWlaR2xrT25kdmNtczZNbFZWU0ZGRFpEUndjM1pyVUV4YVIyNVhXVE16VENOclpYa3RNU0lLSUNCOUNuMD0iLCJpYXQiOjE1MTYyMzkwMjJ9.aAnwgqWyuSCeaRQk6vZJC0EnVnBEri8TIVjyE0wdEq8YnCkj0k6xX-PW4nTBnRDRbHWeii7jhLRC36BOJwtbYWpnnjqjNJpsui_TUCxpVZ80ag44fV5gAPlNpyHGv3EmLzaqBxQPW6YH6elAEJ8ViXeM0FfaoR5zA1RuWUeqG_8
```

Decoding this JWT, you will notice the `sub` property is the did of the recipeint, `did:work:H3C2AVvLMv6gmMNam3uVAjZpfkcJCwDwnZn6z3wXmqPV`. The `badge` body has been base64 encoded.

### Resolvability of DIDs
Without widespread web browser support, special tooling is needed to resolve a DID Document. [DID Resolvers](https://w3c.github.io/did-core/#did-resolvers) are called out in the DID Core specification as a method of resolving DIDs according to their [DID Method](https://w3c-ccg.github.io/did-method-registry/).

The [Universal Resolver](https://github.com/decentralized-identity/universal-resolver), actively being developed by the [Decentralized Identity Foundation (DIF)](https://identity.foundation/) is one example of a tool that can be used to resolve DID documents across disparate DID methods and ledgers.

It is recommended that HTTP-backed-DIDs can appear fully qualified using the `did` property. This will allow more clear and simpler resolution for certain `did` methods. An example is provided below:

```
{
  "@context": "https://w3id.org/openbadges/v2",
  "type": "Assertion",
  "id": "https://example.org/beths-robotics-badge.json",
  "recipient": {
    "type": "did",
    "hashed": false,
    "identity": "https://uniresolver.io/1.0/identifiers/did:work:2UUHQCd4psvkPLZGnWY33L"
  },
  "image": "https://example.org/beths-robot-badge.png",
  "evidence": "https://example.org/beths-robot-work.html",
  "issuedOn": "2016-12-31T23:59:59Z",
  "expires": "2017-06-30T23:59:59Z",
  "badge": "https://example.org/robotics-badge.json",
  "verification": {
    "type": "hosted"
  }
}
```

This example shows use of the [DIF Universal Resolver](https://github.com/decentralized-identity/universal-resolver) to resolve the `did`.

## Drawbacks
Though Decentralized Identifiers are an official W3C specification, they have yet to see widespread adoption. Drawbacks are similar to drawbacks for all new, or up-and-coming technologies. We believe this drawback is minimized by the endorsement of the DID specification by the W3C, and a number of prominent technology companies and enthusiasts.

## Prior Art
The document out of Rebooting the Web of Trust VI titled [Open Badges are Verifiable Credentials](https://github.com/WebOfTrustInfo/rwot6-santabarbara/blob/master/final-documents/open-badges-are-verifiable-credentials.md) discusses assertions made by organizations and to individuals identified by DIDs. The RWoT paper advocates using a new `id` property, of which a DID would fall under. This document has been inspired by the existing paper, and chosen to expand on its proposal.

## Security & Privacy Considerations
This proposal aims to allow Decentralized Identifiers to be used for identifying the issuer and recipient of an Open Badge. A DID-based recipient who can prove ownership of a referenced DID Document consequently proves ownership of an Open Badge. Without proper signatures in place a verifying party should be suspicious of the authenticity of Open Badges that include DIDs. 

It is recommended that all Open Badges with DID issuers are _signed badges_. _Hosted badges_ can include DID issuers if the host is able to prove ownership of a DID. This could be accomplished if the host adopts the [Well Known DID Configuration](https://identity.foundation/.well-known/resources/did-configuration/) proposal from the DIF.

Further protocols and standards could be developed allowing badge holders to prove control of their badge with their DID.

## Interoperability Considerations
As thought out and explained in the [Open Badges are Verifiable Credentials](https://github.com/WebOfTrustInfo/rwot6-santabarbara/blob/master/final-documents/open-badges-are-verifiable-credentials.md) paper, adding DID support to the Open Badges specification brings Open Badges closer to Verifiable Credentials. This enables symbiosis between Verifiable Credentials and Badges by creating an ecosystem where both can flourish.

It is the opinion of the authors that both Verifiable Credentials and Open Badges serve distinct purposes. The addition of DID support allows both to exist side by side, with interoperability made simpler.

## Implementation Notes
Central to the idea of identity in Open Badges is the concept of uniqueness. Existing identity properties, such as email, telephone, and URLs have well established mechanisms for interactively proving ownership of the identity, thus preventing fraudulent use of issued Open Badges. Suffice to say that such a corresponding mechanism exists for DID interactions, which allows for a recipient to prove ownership of a DID Document.  It is not recommended to issue Open Badges to, or accept Open Badges from, recipients using DIDs without having access to such a identity verification mechanism.

For more information on proving ownership of a DID, and public keys in a DID Document, please refer to the [DID Core Specification section on Binding of Identity](https://www.w3.org/TR/did-core/#binding-of-identity).

## References
- [Open Badges v2.0](https://www.imsglobal.org/sites/default/files/Badges/OBv2p0Final/index.html)
- [DID Specification](https://w3c.github.io/did-core/)
- [Verifiable Credentials Specification](https://w3c.github.io/vc-data-model/)
- [Identity Hub Specification](https://github.com/decentralized-identity/identity-hub/blob/master/explainer.md)
- [DID Method Registry](https://w3c-ccg.github.io/did-method-registry/)
- [Universal Resolver](https://github.com/decentralized-identity/universal-resolver)
- [Decentralized Identity Foundation](https://identity.foundation/)
- [Well Known DID Configuration](https://identity.foundation/.well-known/resources/did-configuration/)
