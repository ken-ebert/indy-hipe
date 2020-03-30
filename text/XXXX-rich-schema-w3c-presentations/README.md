# Indy HIPE XXXX: W3C-Compatible Verifiable Presentations
- Authors: Alexander Shcherbakov <alexander.shcherbakov@evernym.com>, Brent Zundel <brent.zundel@evernym.com>, Ken Ebert <ken@sovrin.org> 
- Start Date: 2020-03-27

## Status
- Status: [PROPOSED](/README.md#hipe-lifecycle)
- Status Date: 2019-03-27
- Status Note: just proposed



## Summary
[summary]: #summary

A new JSON-LD  format for anonymous (zero-knowledge proof based) verifiable
presentations in Indy, compatible with the W3C Verifiable Credentials Data Model
1.0 web standard. This is part of the rich schema work. 

Despite being compatible with W3C, Indy presentations have a number of explicit
assumptions about content and cardinality for some of W3C verifiable
presentation properties. 


## Motivation
[motivation]: #motivation

### Standards Compliance
The W3C has published the Verifiable Credentials Data Model 1.0 as an official
recommendation. This proposal brings the format of Indy's anonymous credential
presentations into compliance with that standard.

### Interoperability
Compliance with the Verifiable Credentials Data Model 1.0 introduces the
possibility of interoperability with other credential presentations that also
comply with the standard that use similar proof mechanisms. The verifiable
credential data model specification is limited to defining the data structure
of verifiable credentials and presentations. This includes defining extension
points, such as "proof."


## Tutorial
[tutorial]: #tutorial

W3C Verifiable Presentations are used for the aggregation of claim information
from multiple verifiable credentials. They may also be used to present a set of
credentials which were derived from source credentials in such a way as to limit
information sharing. Indy anonymous credential presentations will make use of
derived credentials. 

The proposed presentation format follows the W3C specification requirements
for presentations which use zero-knowledge proofs:
- [Basic properties for presentations](https://www.w3.org/TR/vc-data-model/#presentations-0)
- [Presentations using derived credentials](https://www.w3.org/TR/vc-data-model/#presentations-using-derived-credentials)
- [Zero-knowledge proof example](https://w3c.github.io/vc-data-model/#zero-knowledge-proofs)

Although in general Indy anonymous credential presentations described here
follow and are compatible with the Verifiable Credential Data Model 1.0
specification, there are a number of explicit requirements for Indy anonymous
credential presentations (see the next two sections).

### Properties
Any Indy presentation compatible with W3C standard MUST have the following
properties:

#### @context 
JSON-LD Context. The value MUST be an ordered set where 
 - the first item MUST be a URI with the value `https://www.w3.org/2018/credentials/v1`.
 - subsequent other items SHOULD be the `@id` of contexts used by the rich
schemas that correspond to each included anonymous credential. 
 (see [Rich Schema Context](https://github.com/hyperledger/indy-hipe/tree/master/text/0149-rich-schema-schema#context)).


#### type
A type of the verifiable presentation.
It MUST be an unordered set consisting of the following two values:
- `VerifiablePresentation` which is a type common for all W3C verifiable
presentations (see `https://www.w3.org/2018/credentials/v1` context).
- `@id` of the presentation definition for which this is a valid presentation.

#### verifiableCredential
The value contains an unordered list of derived credentials, each of which
consists of (nested) claims about each subject of the credential and the
corresponding values. Each derived credential MUST follow the guidelines
for Indy anonymous credentials defined in
[Indy HIPE 0160: W3C Compatible Verifiable Credentials.](https://github.com/hyperledger/indy-hipe/tree/master/text/0160-rich-schema-w3c-credentials)

The set of claims included in the derived credentials MUST match the ones
requested by the presentation definition object. This may be a subset of the
claims present in the original anonymous credential.

The value of the claims are 'raw' values according to the claim's type, as
defined by the corresponding rich schema.

The integer representation of the claim values, as required by the CL
(Camenisch-Lysyanskaya) ZKP signature scheme, are included as attributes in the
`proof` property of each derived credential. The transformation from claim value
to integer is defined by the encoding objects from the corresponding mapping. 

##### verifiableCredential.proof
Each derived credential has a `proof` property can be used to verify the
included claims in zero-knowledge and to prove knowledge of the issuer's
signature on the source credential.   

The `proof` MUST contain a `credentialSchema` property that MUST have the
same value as the `credentialSchema` property of the source credential. This
provides the verifier with a link to the public key material necessary for
verifying the provided proofs.
 
Other properties in the `proof` are type-specific and used by the crypto layer
only. End users should not have any assumptions about the fields and should not
try to parse them.   

#### proof
The verifiable presentation `proof` property provides information necessary to
verify that each derived credential is linked to the same holder. 

The `proof` MUST contain a `type` property which indicates the ZKP signature
scheme used. Currently the `type` MUST be equal to either `CL` or `CL2` for
Camenisch-Lysyanskaya (Anoncreds 1.0) or Camenisch-Lysyanskaya 2.0
(Anoncreds 2.0), correspondingly.  

The value of this `type` property MUST have the same value as the
`signatureType` property of the credential definition specified by the
`credentialSchema` of the `proof` property of each derived credential included
in the presentation.
 
### Rules and Assumptions
A summary of explicit assumptions for W3C-compatible verifiable presentations in
Indy: 

- Indy currently supports only zero-knowledge-based credentials (anoncreds). The only
zero-knowledge signature type currently supported is the Camenisch-Lysyanskaya
signature (either version 1.0 or 2.0). Therefore all credentials included in
a verifiable presentation are derived credentials

- The `id` of each derived credential's rich schema context must be specified in
the `@context` property in addition to the common 
`https://www.w3.org/2018/credentials/v1`.

- The `id` of the presentation definition must be included in the `type`
property in addition to the common `VerifiablePresentation`.

- Indy presentations use DIDs for identification and referencing. In particular,
the following values are expected to be DIDs:
  - Rich schema's contexts (part of `@context` property)
  - Presentation definition's `id` (part of `type`)
  - Credential definition's `id` (`credentialSchema`'s `id` property)

- If the `id` of a rich schema object is defined as a DID, then the
corresponding DID's id-string should be the base58 representation of the
SHA2-256 hash of the canonical form  of the rich schema `content`'s JSON.
 The canonicalization scheme we recommend is the IETF draft 
 [JSON Canonicalization Scheme (JCS)](https://tools.ietf.org/id/draft-rundgren-json-canonicalization-scheme-16.html).

- The set of claims in each derived credential's `credentialSubject` property
must match the ones requested by the presentation definition.
  
- Currently Indy supports only the following values for some of the `type`
properties:
  - The `proof`'s `type` must be equal to either `CL` or `CL2` for
  Camenisch-Lysyanskaya (Anoncreds 1.0) and Camenisch-Lysyanskaya 2.0
  (Anoncreds 2.0) signature schemes, respectively.
  - The `credentialSchema`'s `type` must be equal to `cdf`.

The following diagram summarizes the relationship between W3C-compatible
verifiable presentations in Indy and other rich schema objects: 

![](relationship-diagram.png)



### Example
Let's consider a **Presentation Definition** object with the following
`content`: 
```
TBD
```

Let's consider a **W3C Credential** object with the following `content`:
```
{
  "@context": [
    "https://www.w3.org/2018/credentials/v1",
    "did:sov:2f9F8ZmxuvDqRiqqY29x6dx9oU4qwFTkPbDpWtwGbdUsrCD"    
  ]
  "type": ["VerifiableCredential", "did:sov:4e9F8ZmxuvDqRiqqY29x6dx9oU4qwFTkPbDpWtwGbdUsrCD"],
  "credentialSchema": {
    "id": "id:sov:9F9F8ZmxuvDqRiqqY29x6dx9oU4qwFTkPbDpWtwGbdUsrCD",
    "type": "cdf"
  },
  "issuer": "did:sov:Wz4eUg7SetGfaUVCn8U9d62oDYrUJLuUtcy619",
  "issuanceDate": "2020-03-01T19:23:24Z",
  "credentialSubject": {
    "driver": {
      "firstName": "Jane",
      "lastName": "Doe",
      "dateOfBirth": "1969-02-14"
    },
    "dateOfIssue": "2020-03-01",
    "issuingAuthority": "State of Utah",
    "licenseNumber": "ABC1234",
    "categoriesOfVehicles": {
      "vehicleType": "passenger",
      "dateOfIssue": "2018-06-03",
    },
    "administrativeNumber": "1234"
  },
  "proof": {
    "type": "CL",
    "signature": "8eGWSiTiWtEA8WnBwX4T259STpxpRKuk...kpFnikqqSP3GMW7mVxC4chxFhVs",
    "signatureCorrectnessProof": "SNQbW3u1QV5q89qhxA1xyVqFa6jCrKwv...dsRypyuGGK3RhhBUvH1tPEL8orH",
    "revocationData": "...."
  }
}
```
Finally, let's consider the corresponding **Credential Definition** object with
`id=did:sov:9F9F8ZmxuvDqRiqqY29x6dx9oU4qwFTkPbDpWtwGbdUsrCD` and the following
`content`:
```
"signatureType": "CL",
"mapping": "did:sov:5e9F8ZmxuvDqRiqqY29x6dx9oU4qwFTkPbDpWtwGbdUsrCD",
"schema": "did:sov:4e9F8ZmxuvDqRiqqY29x6dx9oU4qwFTkPbDpWtwGbdUsrCD",
"publicKey": {
    "primary": "...",
    "revocation": "..."
}
```

Then the corresponding **W3C Presentation** for the CL signature scheme may
look as follows: 
```
{
  "@context": [
    "https://www.w3.org/2018/credentials/v1",
    "did:sov:2f9F8ZmxuvDqRiqqY29x6dx9oU4qwFTkPbDpWtwGbdUsrCD"
  ]
  "type": ["VerifiablePresentation", "did:sov:4e9F8ZmxuvDqRiqqY29x6dx9oU4qwFTkPbDpWtwGbdUsrCD"],
  "verifiableCredential": [
    {
      "credentialSubject": {
        "over21": "true"    
      },
      "proof":{
        "credentialSchema": {
          "id": "id:sov:9F9F8ZmxuvDqRiqqY29x6dx9oU4qwFTkPbDpWtwGbdUsrCD",
          "type": "cdf"
        },
        "attributes": "...",
        "pokSignature": "...",
        "predicates": "...",
        "nonRevocation": "..."
      }
    }    
  ],
  "proof": {
    "type": "CL",
    "linking": "..."
  }
}
```

Let's consider every field in detail:
- `@context` points to two contexts: one common for all W3C credentials,
 and one used by the rich schema of the source credential.
 - `type` is an array of two values: the first is common for all W3C
 presentations, `VerifiableCPresentation` (see 
 `https://www.w3.org/2018/credentials/v1` context), the second is the `@id` of
 the presentation definition the presentation is created against.
 - `verifiableCredential` contains a list of derived credentials with (nested)
 claims and the corresponding values. The set of claims matches the ones
 requested by the presentation definition.
   - `proof` contains a `credentialSchema` which identifies the credential
   definition of the source credential. The other content is specific to the
   signature type and should not be parsed by applications.
- `proof` contains a `type` (`CL` in this example). The other content is
specific to the signature type and should not be parsed by applications. 

### Indy Node API
As Indy presentations are never stored on the Ledger, no changes in the
`indy-node` repository are required beyond general rich schema support
(see [Rich Schema Objects Common](https://github.com/hyperledger/indy-hipe/tree/master/text/0120-rich-schemas-common)).

### Indy VDR API
As Indy presentations are never stored on the Ledger, no changes in the
`indy-vdr` repository are required beyond general rich schema support
(see [Rich Schema Objects Common](https://github.com/hyperledger/indy-hipe/tree/master/text/0120-rich-schemas-common)).

### Indy Credx API

The W3C and Rich Schema compatible protocols will be implemented in `indy-credx`
under the following assumptions:
- `indy-credx` already has a set of API calls for credential issuance and proof
presentation protocols. These calls work with the "old" (non-W3C-compatible)
 presentation format and the "old" (non-rich) schema objects.
- W3C and Rich Schema compatible protocols will be implemented as a set of new
API calls.
- The new W3C-compatible API calls can be almost the same as for the current
("old") approach since issuance and presentation protocols are not changed.
However, there is the following difference:
  - The expected format of credentials and presentations is W3C-compatible.
  - All calls expect rich schema objects instead of "old" Schema objects.
  - The encoding of claim values to integers is done according to the encoding
  objects specified by the corresponding mapping object.
- The new W3C-compatible API calls have `w3c` prefix to be distinguished from
non-W3C ("old") ones. 
      
#### Compatiblity with non-W3C credentials
The compatibility between "old" format of credentials and schemas and a "new"
W3C one is **not** assumed on libindy layer. It means that:
  - W3C-compatible versions of issuance and presentation protocols will work
  only with rich schema objects.
  - W3C-compatible presentation definitions expect W3C presentations, so only
  W3C-compatible credentials can be used to generate a presentation.
  - We assume that on the libindy level non-W3C-compatible proof requests
  ("old" proof requests) will only work with non-W3C-compatible credentials.
  Applications using libindy can, in theory, use W3C-compatible credentials with
  "old" proof requests, or use "old" credentials with W3C-compatible
  presentations, but this is out-of-scope for the current HIPE.

|               | W3C-compatible credentials | Non-W3C-compatible credentials |
| ------------- | ------------- | --- |
| **W3C-compatible proof request**  | Yes  | No  |
| **Non-W3C-compatible proof request**  | No  | Yes |
  

  

#### Relationship with Rich Schema
libindy API will have the following assumptions:
- W3C-compatible credentials will work with rich schema objects only.
- Non-W3C-compatible ("old") credentials will work with the "old" Schema only.

Applications using libindy can, in theory, use rich schema for
non-W3C-compatible credentials, but this is out-of-scope for the current HIPE.  

|               | W3C-compatible credentials | Non-W3C-compatible credentials |
| ------------- | ------------- | --- |
| **Rich Schema**  | Yes  | No  |
| **Old Schema**  | No  | Yes |
  
#### Relationship with Anoncreds version   
Both W3C-compatible and non-W3C-compatible issuance and presentation protocols
can work with either Anoncreds 1.0 (`CL`) or Anoncreds 2.0 (`CL2`); the
Anoncreds version is orthogonal to the Schema and credentials format.
 - Anoncreds 1.0 is already working with the Old Schema approach.
 - Both Anoncreds 1.0 and 2.0 must work the with rich schema approach.
 - It's questionable whether Anoncreds 2.0 should work with the old Schema
 approach.

|               | W3C-compatible credentials  and Rich Schema | Non-W3C-compatible credentials and Old Schema |
| ------------- | ------------- | --- |
| **Anoncreds 1.0**  | In Progress  | Already supported  |
| **Anoncreds 2.0**  | TBD  |  Questionable  |

## Reference
[reference]: #reference

- [W3C Verifiable Credentials Specification](https://w3c.github.io/vc-data-model)
- [0119: Rich Schema Objects](https://github.com/hyperledger/indy-hipe/tree/master/text/0119-rich-schemas)
- [0120: Rich Schema Objects Common](https://github.com/hyperledger/indy-hipe/tree/master/text/0120-rich-schemas-common)
- [0160: W3C Compatible Verifiable Credentials.](https://github.com/hyperledger/indy-hipe/tree/master/text/0160-rich-schema-w3c-credentials)

## Drawbacks
[drawbacks]: #drawbacks

- The presentation object formats introduced here will not be backwards
compatible with the current set of credential objects.
- Rich schemas introduce greater complexity.
- The new formats rely largely on JSON-LD serialization and may be
dependent on full or limited JSON-LD processing.


## Unresolved questions and future work
[unresolved]: #unresolved-questions

- We may consider using old credentials for W3C-compatible presentations as well
as using W3C compatible credentials for old proof requests.
- It may make sense to extend DID specification to include using DID as a
credential ID.
- It may make sense to extend DID specification to include using DID for
referencing rich schema objects.
- The proposed canonicalization form of a content to be used for DID's id-string
generation is in a Draft version, so we may find a better way to do it.