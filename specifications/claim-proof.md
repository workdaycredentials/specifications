# Claim Proofs; W3C Credentials Working Group Proposal 1.0
- Authors:
   - Joe Genereux [joe.genereux@workday.com](mailto:joe.genereux@workday.com)
   - Rory Martin [rory.martin@workday.com](mailto:rory.martin@workday.com)
   - Bjorn Hamel [bjorn.hamel@workday.com](mailto:bjorn.hamel@workday.com)
   - Gabe Cohen [gabe.cohen@workday.com](mailto:gabe.cohen@workday.com)
- Last updated: 2019-10-07

## Status
- Status: **PROPOSAL**
- Status Date: 2019-08-28
- Status Note: Request For Comments (RFC), draft 1
 
## Abstract
The [W3C Verifiable Credentials Data Model](https://w3c.github.io/vc-data-model/#the-principle-of-data-minimization) suggests the principle of **data minimization** to reduce the cost of privacy violations in the exchange of verifiable claims. This document outlines the **claimProof** mechanism proposed by Workday as a solution to achieve data minimization in Linked Data Signatures, and is a natural and backwards compatible extension to the W3C-Spec. The following proposal is intended for migration into the W3C specification as a future addition.
 
## Contents
* [Status](#status)
* [Abstract](#abstract)
* [Contents](#contents)
* [Proposal](#proposal)
  + [1 Introduction](#1-introduction)
      - [1.1 Selective Disclosure](#11-selective-disclosure)
      - [1.2 Zero-Knowledge Proofs & Derived Credentials](#12-zero-knowledge-proofs-and-derived-credentials)
      - [1.3 The drawbacks with anonymization of credentials with ZKP](#13-the-drawbacks-with-anonymization-of-credentials-with-zkp)
  + [2 Attribute Level Signatures](#2-attribute-level-signatures)
      - [2.1 Embedded Proofs](#21-embedded-proofs)
           * [Embedded Proof Example](#embedded-proof-example)
      - [2.2 Claim Proofs](#22-claim-proofs)
           * [Claim Proof Example](#claim-proof-example)
      - [2.3 Claim Proof Requirements](#23-claim-proof-requirements)
           * [Example Claim Proof Block](#example-claim-proof-block)
      - [2.4 Proof Request/Response](#24-proof-requestresponse)
* [Drawbacks/Limitations](#drawbackslimitations)
* [Alternatives](#alternatives)
* [Privacy Considerations](#privacy-considerations)
* [References](#references)
 
## Proposal
 
### 1 Introduction
In [section 7.8](https://w3c.github.io/vc-data-model/#the-principle-of-data-minimization) of the Verifiable Credentials Data Model (here-after **W3C-spec**) we learn that privacy violations occur when `information divulged in one context leaks into another`. It is widely accepted that for individuals and organizations large and small, privacy is becoming a central focus and feature in the exchange of information. Principles such as **data minimization** help reduce the risk of such violations. Hereafter we define data minimization as limiting the information requested, and received, to the absolute minimum necessary.
 
Verifiable credentials help reduce the risks of privacy violations by allowing the holder to share limited information with a verifier. This is in contrast to the traditional credential verification model where a verifier talks directly to an issuer. However, verifiable credentials are also susceptible to privacy violations when the credential contains more information than the verifier requires. To address this susceptibility, the W3C-Spec recommends `for issuers [... to limit] the content of a verifiable credential to the minimum required by potential verifiers for expected use.` And correspondingly, `For verifiers, [... to limit] the scope of the information requested or required for accessing services`.
 
For systems that are using [Linked Data Signatures](https://w3c-dvcg.github.io/ld-signatures) for claims exchange, this specification proposes a mechanism called a **claim proof** that facilitates selective disclosure of individual claims. We recognize that [zero-knowledge proofs](https://w3c.github.io/vc-data-model/#zero-knowledge-proofs) and derived credentials is another technique to achieve data minimization in claims exchange. This document focuses only on systems using Linked Data Signatures.
 
#### 1.1 Selective Claim Disclosure
 
The W3C-spec uses a term called *selective disclosure* to refer to the holder's `ability to make fine-grained decisions about what information to share.`  When using Linked Data Signatures, the granularity is limited to the full credential.  As stated above, this relies on the issuer making a predetermination about `the minimum required by potential verifiers for expected use.`  We submit that this will always lead to some level of privacy violation given that the issuer cannot know a priori the minimum set required by every verifier. We believe that the appropriate level of granularity should be on a per claim basis.
 
We formalize on the concept of **selective claim disclosure**, which we define as the process of only revealing the values and signatures of a subset of claims and withholding all others on the credential. Whether or not that subset of claims satisfies the data requested by the verifying party depends on the credential exchange protocol implementation.. An example is given in the spec: `[...] a driver's license containing a driver's ID number, height, weight, birthday, and home address is a credential containing more information than is necessary to establish that the person is above a certain age.`
 
There are many reasons why a holder of a credential or a verifier may want to hide the non-requested information. The most obvious case that comes to mind relates that the holder simply does not want a verifier to know any extra information than what is needed to satisfy the proof. Conversely, the verifier, concerned about privacy violations, may not want to be liable for requesting and holding any information that isn't required to fulfill the request. Another reason is simply the less information revealed in a presentation results in less sensitive information being transferred from the holder to verifier where someone might be able to intercept the data in the middle.
 
### 2 Claim Proofs
The following section outlines the Claim Proof protocol and its implementation details.
 
#### 2.1 Embedded Proofs
In a typical verifiable credential (as defined by the W3C-spec), we create an **embedded proof** using a linked data signature. This signature is issued over the whole credential in order to detect tampering and verify authorship of a credential or presentation.
 
##### Embedded Proof Example
<pre>
{
  "version": "1.0.0",
  "id": "did:work:<cred_def_did>#uuid",
  "type": ["VerifiableCredential", "UniversityDegreeCredential"],
  "issuer": "did:work:abc123",
  "issuanceDate": "2018-01-01T00:00:00+00:00",
  "targetHolder": "did:work:def345",
  "credentialSchema": {
    "id": "did:work:abcdefghijklmnop;spec:uuid",
    "type": "JsonSchemaValidator2018"
  },
  "credentialSubject": {
    "degree": {
      "type": "BachelorDegree",
      "name": "Bachelor of Science in Mechanical Engineering"
    },
    "institution": "UC Berkeley"
  },
  <b>"proof": {
    "type": "RsaSignature2018",
    "created": "2018-01-01T00:00:00+00:00",
    "verificationMethod": "https://example.com/jdoe/keys/1",
    "signatureValue": "BavEll0/I1zpYw8XNi1bgVg/sCneO4Jugez8RwDg/+
      MCRVpjOboDoe4SxxKjkCOvKiCHGDvc4krqi6Z1n0UfqzxGfmatCuFibcC1wps
      PRdW+gGsutPTLzvueMWmFhwYmfIFpbBu95t501+rSLHIEuujM/+PXr9Cky6Ed
      +W3JT24="
  }</b>
}
</pre>
 
In the example above we have a verifiable credential for a university degree. The embedded proof was created over the entire credential. During verification a holder is able to prove cryptographically she has a credential with the fields *degree* and *institution*; however, because the signature was issued over the entire credential, the holder is not able to selectively reveal only one of those attributes. This can be a very common issue in claims presentations as the holder may need to prove she studied at university but not what degree she attained, or conversely, reveal what she studied but not where.
 
#### 2.2 Claim Proofs
Hypothetically, the issuer could instead choose to issue individual credentials per claim. In theory, this would allow the holder to selectively disclose information at the granularity of a claim. However, the claims are not truly independent of each other, and part of the semantic meaning of the credential is lost when they are separated. Therefore, we seek to provide a mechanism that extends the current use of Linked Data Signatures to also include proofs per claim.
 
In contrast to the example above, we would like to extend the use of a Linked Data Signatures from being over the entire credential to also include a signature per claim. This shift in protocol will allow us to achieve *selective claim disclosure* of individual claims. 
 
We outline the following requirements of such an extension:
* It is backwards compatible with systems that do not support per claim signatures
* It still allows the verifier to request the entire credential
* It allows the holder to select any subset of claims that she chooses to disclose
 
We propose a new property to the W3C-spec, **claimProof**. In this model a signature is generated over *each* attribute in the credentialSubject field followed by a signature over the entire credential using the standard embedded proof model in the section above. Effectively this creates a separate verifiable credential for each attribute, packaged together by the linked data signature over the entire credential. A key difference between using the claimProof and issuing a set of credentials is that the credential metadata is exactly the same and included in the signatures of all of the claims. This binds all of the claims together semantically, but allows for individual disclosure. With this mechanism we are now able to achieve selective claim disclosure of individual properties in a verifiable credential.
 
If the `claimProof` property is present:
- The property **MUST** be a JSON map with keys corresponding to the keys of the credentialSubject field.
- Signature values **MUST** be generated with *all* credential metadata for verification.
 
**claimProof** 
<dd>
The value of the claimProof <b>MUST</b> be a JSON map with keys corresponding to the individual keys of claims in the <i>credentialSubject</i> field. Each claimProof field has an independent proof block for verification purposes that is signed over the metadata. If there are multiple credentialSubjects (an array) then claimProof fields may be structured in a corresponding array. 
</n>
</dd>
 
##### ClaimProof Example
<pre>{
  {
  "modelVersion": "1.0",
  "@context": "https://",
  "id": "did:example:#uuid",
  "type": ["VerifiableCredential", "UniversityDegreeCredential"],
  "issuer": "did:work:abc123",
  "issuanceDate": "2018-01-01T00:00:00+00:00",
  "credentialSchema": {
    "id": "did:work:abcdefghijklmnop;spec:uuid",
    "type": "JsonSchemaValidator2018"
  },
  "credentialSubject": {
    "degree": {
      "type": "BachelorDegree",
      "name": "Bachelor of Science in Mechanical Engineering"
    },
    "institution": "UC Berkeley"
  },
  <b>"claimProofs": {
    "degree": {
      "type": "RsaSignature2018",
      "created": "2018-01-01T00:00:00+00:00",
      "verificationMethod": "https://example.com/jdoe/keys/1",
      "nonce": "123456",
      "signatureValue": "..."
    },
    "institution": {
      "type": "RsaSignature2018",
      "created": "2018-01-01T00:00:00+00:00",
      "verificationMethod": "https://example.com/jdoe/keys/1",
      "nonce": "123456",
      "signatureValue": "..."
    }</b>
  },
  "proof": {
    "type": "RsaSignature2018",
    "created": "2018-01-01T00:00:00+00:00",
    "verificationMethod": "https://example.com/jdoe/keys/1",
    "nonce": "1234",
    "signatureValue": "BavEll0/I1zpYw8XNi1bgVg/sCneO4Jugez8RwDg/+
      MCRVpjOboDoe4SxxKjkCOvKiCHGDvc4krqi6Z1n0UfqzxGfmatCuFibcC1wps
      PRdW+gGsutPTLzvueMWmFhwYmfIFpbBu95t501+rSLHIEuujM/+PXr9Cky6Ed
      +W3JT24="
  }
}
</pre>
 
In the example above we have reissued the same education credential, but this time we want to give the holder the freedom of selective claim disclosure over her individual claims. She now has the option to prove where she studied but not what degree she earned, as well as what she earned but not from where. Because she also still has the embedded proof over the entire document, she still also has the ability to present the entire credential should both fields be required in a presentation or if the verifier *does not support the claimProof property* which we establish as a backwards compatible requirement for the spec.

##### Validation
The following JSONschema can be applied to validate against a credential containing a claimProofs block.

<details>
  <summary>Show/Hide JSON Schema</summary>

```
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "description": "W3C Verifiable Credential using JSON interchange format",
  "type": "object",
  "properties": {
    "modelVersion": {
      "type": "string",
      "pattern": "^\\d+\\.\\d+$"
    },
    "@context": {
      "type": "string"
    },
    "id": {
      "type": "string"
    },
    "type": {
      "type": "array",
      "contains": {
        "const": "VerifiableCredential"
      }
    },
    "issuer": {
      "type": "string"
    },
    "issuanceDate": {
      "type": "string",
      "format": "date-time"
    },
    "credentialSchema": {
      "type": "object",
      "properties": {
        "id": {
          "type": ["null", "string"]
        },
        "type": {
          "type": "string",
          "enum": [
            "JsonSchemaValidator2018"
          ]
        }
      },
      "required": [
        "id",
        "type"
      ],
      "additionalProperties": false
    },
    "credentialSubject": {
      "type": "object",
      "minProperties": 1
    },
    "claimProofs": {
      "type": ["null", "object"],
      "patternProperties": {
        "^.*$": { 
          "type": "object", 
          "properties": {
            "type": {
          "type": "string"
        },
        "created": {
          "type": "string",
          "format": "date-time"
        },
        "verificationMethod": {
          "type": "string"
        },
        "nonce": {
          "type": "string"
        },
        "signatureValue": {
          "type": "string"
        }
          },
          "required": [
            "type",
            "created",
            "verificationMethod",
            "nonce",
            "signatureValue"
          ], 
          "additionalProperties": false
        }
      }
    },
    "proof": {
      "type": "object",
      "properties": {
        "type": {
          "type": "string"
        },
        "created": {
          "type": "string",
          "format": "date-time"
        },
        "verificationMethod": {
          "type": "string"
        },
        "nonce": {
          "type": "string"
        },
        "signatureValue": {
          "type": "string"
        }
      },
      "required": [
        "type",
        "created",
        "verificationMethod",
        "nonce",
        "signatureValue"
      ],
      "additionalProperties": false
    }
  },
  "required": [
    "modelVersion",
    "@context",
    "id",
    "type",
    "issuer",
    "issuanceDate",
    "credentialSchema",
    "credentialSubject",
    "claimProofs",
    "proof"
  ],
  "additionalProperties": false
}
```

</details>
 
 
##### Interoperability
In order to be more compliant with the existing W3C-Spec we recognize that breaking out the per-claim proofs into a new property fits more appropriately in the extensibility model. Effectively the claimProof property could be ignored and the overall proof and credential would still be valid.
 
##### Signatures Note
 
From the W3C-spec: `Because the method used for a mathematical proof varies by representation language and the technology used, the set of name-value pairs that is expected as the value of the proof property will vary accordingly. For example, if digital signatures are used for the proof mechanism, the proof property is expected to have name-value pairs that include a signature, a reference to the signing entity, and a representation of the signing date.` This holds true for claimProofs as well and it is RECOMMENDED that all proofs use the same format in the verifiable credential.
 
##### Example RSA claimProof structure:
* `key of credentialSubject claim`:
  * `type` type of signature
  * `created`: Signing date
  * `verificationMethod`: method of signature verification
  * `claimSignatureValue`: signature of the credential with the single claim present.
 
#### 2.4 Proof Request/Response
A separate proposal to update the W3C-spec to support proof requests/responses for individually signed attributes on a credential will also be submitted.
 
## Drawbacks/Limitations
- The **claimProof** field is optional and relies on the issuer to include it when issuing credentials and the verifier to support a credential exchange protocol that uses it.
 
## Alternatives
The following are a list of known alternatives to achieve selective disclosure over verifiable claims:
- Issuing a separate credential for each field in a schema
- Derived Credentials with Zero-Knowledge-Proofs
- Signatures included as part of the credential subject field. (this was also explored, but that forces direct incompatibility with the W3C-Spec so it is less desirable).
 
The authors of this specification would like to remain very clear that while we are able to achieve selective disclosure without the use of ZKP's in verifiable claims presentations, we are **NOT** dismissing the unique and desirable properties that are provided using ZKP schemes and believe that *both* models each serve unique purposes in claim presentations. We therefore recommend that organizations evaluate both models and make a decision that fits their business use case and needs. We anticipate that future platform implementations will allow for *both* zero-knowledge proof and attribute level signatures as part of their supported feature set. The use of such hybrid platform schemes are also being explored and we recommend that the community further explore how to interoperate at the protocol layer to support both schemes.
 
## Privacy Considerations
- As with any disclosed presentation with verifiable credentials, this proposal for claim proofs over individually signed attributes does not obscure the revealed attributes. This is in contrast to a Zero-Knowledge Proof based system (such as CL-Signatures used in Indy) which has the ability to to attest a claim in zero-knowledge if it is sufficient for the verifier's request (if not done in zero knowledge the properties are still revealed and thus suffer from the same vulnerability). All credential verification and issuance should be treated with care by implementers regardless of the solution.
- It is the opinion of the authors that no matter what techniques are taken to minimize the risk of privacy violations (including zero-knowledge proofs), that privacy violations can *still* occur. We would like to reiterate and emphasize on [this](https://w3c.github.io/vc-data-model/#the-principle-of-data-minimization) point made by the w3c-spec authors about privacy: `While it is possible to practice the principle of [selective] disclosure, it might be impossible to avoid the strong identification of an individual for specific use cases during a single session or over multiple sessions. The authors of this document cannot stress how difficult it is to meet this principle in real-world scenarios.`
 
## References
- W3C Verifiable Credentials Specification: https://www.w3.org/TR/vc-data-model
