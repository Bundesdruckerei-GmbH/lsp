# WE BUILD - Conformance Specification: Authentic Source–QTSP Attribute Data Interface

Version 0.1 (Proposal)

## Table of Contents

- [1. Introduction](#1-introduction)
- [2. Scope](#2-scope)
- [3. Normative Language](#3-normative-language)
- [4. Roles and Components](#4-roles-and-components)
- [5. Protocol Overview](#5-protocol-overview)
- [6. High-level Flows](#6-high-level-flows)
	- [6.1 Source onboarding and trust establishment](#61-source-onboarding-and-trust-establishment)
	- [6.2 Attribute retrieval](#62-attribute-retrieval)
	- [6.3 Attribute verification](#63-attribute-verification)
	- [6.4 Failure and recovery](#64-failure-and-recovery)
- [7. Normative Requirements](#7-normative-requirements)
	- [7.1 Common requirements](#71-common-requirements)
	- [7.2 QTSP requirements](#72-qtsp-requirements)
		- [7.2.1 EAA provider proofing obligations](#721-eaa-provider-proofing-obligations)
	- [7.3 Authentic Source requirements](#73-authentic-source-requirements)
	- [7.4 Data protection and security](#74-data-protection-and-security)
- [8. Interface Definitions](#8-interface-definitions)
	- [8.1 Retrieve](#81-retrieve)
	- [8.2 Verify](#82-verify)
	- [8.3 Transport and security binding](#83-transport-and-security-binding)
- [9. Conformance](#9-conformance)
- [References](#references)

# 1. Introduction

Digital attestations are only useful when their underlying attributes can be obtained and assessed reliably. In WE BUILD, those attributes are provided by different public- and private-sector authentic sources to QTSPs across organisational and national boundaries. A shared approach to this exchange between authentic source and QTSP can improve interoperability without requiring sources to expose the same internal systems or use identical data models.

This specification establishes common expectations for how an Authentic Source Interface Provider and a QTSP exchange attribute data to have their attestation issued into a wallet. It aims to make that interaction predictable and testable across WE BUILD use cases. The BU2 EUCC flow is an initial use case; the specification is intended to support other attestation scenarios as well.
The applicable attestation rulebook and credential data schema define credential-specific attributes and data structure. The relevant WE BUILD credential-issuance specification sets issuance interoperability requirements. During onboarding, the QTSP and ASIP agree the supported data sets, subject-identification schemes, source, endpoint, and access and operational conditions; the QTSP establishes the applicable legal basis and purpose for each request. The BU2 EUCC flow is an initial use case; the specification is intended to support other attestation scenarios as well.

This document should be read together with the applicable attestation rulebook, its credential data schema, and the relevant WE BUILD credential-issuance specification.

# 2. Scope

This specification covers the logical interface by which a QTSP obtains current attribute data from an authentic source and, where supported, asks the source to verify attributes supplied by another party. It covers source-side provision and verification used in issuing electronic attestations of attributes (EAAs), including Qualified EAAs (QEAAs), without defining the QTSP's complete issuance process.

The WE BUILD common baseline is:

- the Authentic Source Interface Provider (ASIP) **MUST** support QTSP-initiated `Retrieve` for its declared data sets;
- the Authentic Source Interface Provider (ASIP) **MAY** support `Verify` as an additional operation; and
- a QTSP **MUST** support `Retrieve` for each source and data set it claims to interoperate with.

This baseline strengthens TS 119 478 by requiring I3 `Retrieve`, which TS 119 478 makes optional. It does not make I2 `Verify` optional for an implementation claiming TS 119 478 conformance. An Authentic Source that implements only the WE BUILD baseline **MUST NOT** claim TS 119 478 conformance.

TS 119 478 addresses verification under eIDAS Article 45e for specified attributes relying on public-sector authentic sources, and also defines optional retrieval of additional attributes at the user's request. WE BUILD may reuse its interfaces for broader public- or private-sector attestation scenarios. For each use, the QTSP **MUST** establish the applicable legal basis; reuse of the interface for a broader scenario does not imply that every TS 119 478 legal obligation applies unchanged.

This specification is credential-format neutral. It does not define the content schema for every EAA, the legal basis for a particular disclosure, or the QTSP's complete issuance process. The applicable attestation rulebook and data schema define the credential's attributes and structure; the QTSP's issuance policy and applicable law determine the legal basis and purpose, while the source-service agreement identifies the authoritative source and supported subject-identification scheme. Any stricter assurance or security requirements **MUST** be stated in this specification, another applicable WE BUILD conformance specification, or the source-service agreement.

**In scope**

- the roles, trust relationship, and responsibilities of the QTSP and Authentic Source;
- the semantics and minimum information requirements for `Retrieve` and the optional API operation `Verify`;
- security, data-protection, error-handling, and evidence requirements for this exchange; and
- conformance expectations for each role.

**Out of scope**

- the detailed methods and full lifecycle of QTSP identity proofing and EAA issuance, except for the subject, subscriber, representation, source-identity, and authorization checks that directly affect an attribute-data request; these remain governed by the applicable EAA policy and ETSI TS 119 471;
- the wallet-to-QTSP credential issuance protocol and the credential format;
- attestation-specific attribute definitions and validation rules, which belong in the relevant rulebook and data schema;
- source registry internals, national identifiers, and source-specific API implementation details; and
- universal source discovery. This specification does not assume that a consortium-wide source directory exists.

# 3. Normative Language

The keywords **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** are to be interpreted as described in [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119).

These keywords indicate the strength of requirements for conforming implementations. Requirements apply only to the role(s) named in the requirement.

# 4. Roles and Components

This specification defines the following roles:

- **Authentic Source:** a repository or system under the responsibility of a public-sector body or private entity that contains and provides attributes and is considered a primary source or recognized as authentic under Union or national law, including administrative practice. This definition follows eIDAS and ETSI TS 119 478.
- **Authentic Source Interface Provider (ASIP):** the provider of the source interface. It may be the Authentic Source itself or an intermediary acting on its behalf; it is responsible for the interface responses it provides.
- **QTSP:** a trust service provider that requests attributes or verification from an Authentic Source and uses the response as input to its own proofing and issuance decisions. The QTSP remains responsible for satisfying requirements applicable to its attestation service, including ETSI TS 119 471 where applicable; it does not transfer that responsibility to the Authentic Source.
- **EAA Service Provider (EAASP) / Qualified EAASP (QEAASP):** the provider of one or more EAA services, qualified or non-qualified. A QTSP performing the issuance role in this specification is subject to the applicable EAASP or QEAASP requirements in ETSI TS 119 471. That standard's service-level requirements are not themselves requirements of the Authentic Source API.
- **Attestation Subject:** the natural person, legal person, or other entity to which the requested attributes relate. The subject may be represented by a holder or an authorised representative.

The QTSP and Authentic Source may be separate legal entities. A component may fulfil more than one role only where the separation of responsibilities and applicable legal requirements remain clear.

# 5. Protocol Overview

The QTSP first establishes that it is communicating with the intended Authentic Source and is authorised to request the relevant data. For each issuance transaction, it submits a purpose-bound request identifying the subject and the attributes required by the applicable attestation rulebook and data schema. The source returns the requested data and its response status. The QTSP evaluates that response against its issuance policy and the applicable attestation rulebook before issuing an attestation.

ETSI TS 119 478 V1.1.1 (2026-01), *Specification of interfaces related to Authentic Sources*, specifies I2 (`Verify`) and I3 (`Retrieve`), as well as I1 (`Discover`) and, for its HTTP/OAuth binding, I4 (`Authorize`). Under TS 119 478, I2 `Verify` is required and I3 `Retrieve` is optional. It defines HTTP/OAuth and ISO 15000 interface bindings. ETSI TS 119 471 is relevant to the QTSP's policy and security obligations when issuing QEAAs; it does not define this source API.

For WE BUILD conformance, `Retrieve` is mandatory: the QTSP asks the source to return attributes held by, or obtainable from, that source. `Verify` is optional: the QTSP submits candidate attribute values and asks the source to compare them with authoritative records. For TS 119 478 conformance, the requirements are different: I2 `Verify` is mandatory, while I3 `Retrieve` is optional. This specification therefore adds mandatory retrieval without implying that the ETSI-mandatory verification interface can be omitted by an ETSI-conforming implementation.

The interface is not a substitute for provider proofing. A successful source response is evidence supplied to the QTSP; the QTSP determines whether the source, data, subject binding, and assurance are sufficient for the intended attestation. Optional support for the API `Verify` operation does not make attribute verification optional: the provider must meet the proofing obligations in its applicable EAA policy and service requirements. For QEAAs, the QTSP must also meet the applicable legal and ETSI requirements for its service.

The BU2 scenario cites TS 119 478 for the source interface. A deployment claiming conformance to TS 119 478 **MUST** select the HTTP/OAuth binding, the ISO 15000 binding, or both, and identify the applicable edition and endpoint metadata. It **MUST** apply the selected binding's request/response schemas and security and authorization procedures. The WE BUILD requirement to implement `Retrieve` is additional to the selected binding; it does not waive any mandatory ETSI operation or requirement.

# 6. High-level Flows

## 6.1 Source onboarding and trust establishment

1. The QTSP and Authentic Source agree the service endpoint, supported data sets, identifier schemes, purposes, access conditions, security mechanisms, and operational contacts.
2. The QTSP establishes that the source is the intended entity and is authoritative or authorised for the relevant data sets, using the trust mechanism documented in the service agreement. This specification does not prescribe a universal discovery service.
3. The source authorises the QTSP client to invoke the agreed operations. The parties configure key and certificate lifecycle, status checking, and incident handling.
4. The parties test successful and unsuccessful exchanges before using the interface for issuance.

## 6.2 Attribute retrieval

1. The QTSP determines that an issuance transaction requires attributes defined by the applicable attestation rulebook and data schema, and that it has a lawful basis and authority to request them.
2. The user authorizes the request and provides or confirms the subject-identification data required by the selected binding. For HTTP/OAuth, the user-authorization flow is performed through I4 `Authorize`; an ISO 15000 deployment obtains and records the required user authorization before submitting the source request, as specified by TS 119 478.
3. The QTSP sends a `Retrieve` request to the source over the selected binding. The request identifies the subject and requested attributes or data set as required by the binding and source-service agreement.
4. The source authenticates and authorises the QTSP, validates the request and its authorization context, resolves the subject, and retrieves the requested data from its authoritative records.
5. The source returns the requested attributes and the response status and source identification required by the binding. It returns the binding-defined error or non-success result when data cannot be supplied.
6. The QTSP validates the response, including source identity, subject binding, integrity, status, and any freshness information provided by the binding or source-service agreement. It records relevant evidence and decides whether the response is sufficient for issuance.
7. The QTSP proceeds, requests additional information, or stops issuance according to its policy. A missing, stale, invalid, or unverifiable response is not treated as a successful retrieval.

## 6.3 Attribute verification

1. The QTSP has candidate attribute values, the issuance policy calls for source-side verification, and the source-service agreement supports the `Verify` operation.
2. The user authorizes the request and provides or confirms the subject-identification data required by the selected binding.
3. The QTSP sends a `Verify` request containing the subject and candidate attribute values in the format required by the binding.
4. The Authentic Source returns the binding-defined result for each attribute. Under the HTTP/OAuth binding, TS 119 478 defines `Match`, `NoMatch`, `MatchWithVariation`, and `Unknown`; transport-level errors are distinct from these attribute-level results.
5. The QTSP evaluates the result under its issuance policy. A positive verification result does not, by itself, establish the QTSP's identity-proofing or overall issuance requirements.

## 6.4 Failure and recovery

1. If authentication, authorisation, request validation, subject resolution, or source processing fails, the source returns a protocol-defined error without disclosing unrelated subject data.
2. The QTSP distinguishes a definitive negative result from a temporary failure or indeterminate result. It does not convert a timeout, unavailable source, or malformed response into a positive result.
3. The QTSP retries only where permitted by the protocol and source-service agreement, using the same transaction context where appropriate and avoiding duplicate or ambiguous processing.
4. If the failure cannot be resolved within the issuance policy, the QTSP stops or defers issuance and communicates an appropriate outcome through the separate issuance flow.

# 7. Normative Requirements

## 7.1 Common requirements

An implementation claiming conformance **MUST**:

1. implement the applicable protocol binding and the requirements of this specification for the role it declares;
2. bind each request and response to the intended subject, requested data set, purpose, and issuance transaction, using the mechanisms defined by that binding;
3. distinguish successful, partially successful, negative, indeterminate, and temporary-failure outcomes wherever the binding supports those distinctions;
4. reject malformed, unauthenticated, unauthorised, or context-mismatched messages; and
5. preserve sufficient transaction evidence to demonstrate what was requested, which source responded, when the exchange occurred, and what result was returned, subject to applicable data-protection and retention requirements.

## 7.2 Authentic Source requirements

An Authentic Source Interface Provider conforming to the WE BUILD common baseline **MUST**:

1. provide a `Retrieve` capability for each data set it declares as supported;
2. return only data it is authorised to disclose and that is within the scope of the request;
3. identify the interface provider and, where it acts for another entity, the Authentic Source, as required by the selected binding;
4. ensure that returned data and status represent the source's authoritative records at the time indicated in the response;
5. protect the confidentiality and integrity of requests and responses and authenticate the requesting QTSP according to the selected binding and service agreement;
6. use the selected binding's defined success, failure, not-found, and deferred-response behaviour; and
7. publish or agree the data sets, identifier schemes, supported operations, and operational conditions that it supports.

An Authentic Source Interface Provider **MAY** provide `Verify` under the WE BUILD common baseline. If it does, it **MUST** return the result semantics required by the selected binding. An implementation claiming TS 119 478 conformance **MUST** provide I2 `Verify` as required by that specification.

## 7.3 QTSP requirements and differing EAASP requirements

### 7.3.1 QTSP requirements
A QTSP conforming to this specification **MUST**:

1. implement the client side of `Retrieve` for each source and data set it claims to support;
2. identify and authenticate the Authentic Source and establish that it is authoritative or authorised for the requested data before relying on its response;
3. request only attributes necessary for the declared purpose and permitted by the applicable attestation rulebook and legal basis;
4. ensure that the user's authorization and subject-identification data required by the selected source-interface binding are obtained before making a request;
5. validate response integrity, source identity, subject binding, result status, and any freshness information provided by the binding before relying on returned data;
6. assess the response under its own issuance policy and applicable attestation rulebook; a source response does not transfer the QTSP's proofing or issuance responsibility to the source;
7. not issue an attestation on the basis of a failed, indeterminate, stale, or invalid response unless its applicable policy explicitly permits an alternative evidence path and that path is independently satisfied; and
8. apply the applicable EAASP or QEAASP requirements of ETSI TS 119 471 for the EAA service it operates.

### 7.3.2 EAA provider proofing obligations

When acting as an EAASP, the QTSP **MUST**, in accordance with its EAA service policy and EAA policy:

1. verify the identity of the EAA Subject and/or EAA Subscriber and ensure that the attributes and attribute subject correspond to the EAA Subject;
2. where the EAA Subject and EAA Subscriber differ, verify the Subscriber and obtain and verify evidence of the Subscriber's authority to act for the Subject;
3. where attributes are validated against an Authentic Source, verify the identity of that source or of the recognised intermediary acting for it;
4. request only the minimum data necessary for issuance, verify the attributes required by the EAA policy against the source or sources specified by that policy, and avoid verifying identity attributes that are not necessary for issuance; and
5. define the attributes to be verified through the EAA policy for the attestation.

When acting as a QEAASP, the QTSP **MUST** verify the authenticity of the requested attributes for the identified EAA Subject against the relevant national Authentic Source or an intermediary recognised at national level, as required by the applicable EAA policy. This verification **SHOULD** be electronic where possible. These requirements apply regardless of whether the source interaction uses `Retrieve`, `Verify`, or another permitted evidence path; they do not make the API `Verify` operation mandatory under the WE BUILD common baseline.

These requirements reflect ETSI TS 119 471 clauses 4.2.1.1, 4.2.2.1, and 4.2.2.2. The remaining service-wide policy, operational, cryptographic, and management requirements of TS 119 471 apply to the EAASP/QEAASP as applicable, but are outside the scope of this interface specification.

A QTSP **MUST** implement the `Verify` API operation when it claims conformance to TS 119 478 or to another applicable WE BUILD conformance specification that requires `Verify`. Otherwise, under the WE BUILD common baseline, it **MAY** use `Retrieve` as its source interaction. This optionality concerns the API operation only; it does not waive any attribute-verification obligation imposed on the EAASP/QEAASP by its EAA policy, applicable law, or ETSI TS 119 471.




# 8. Interface Definitions

The logical operations below are binding requirements. Wire-level endpoint paths, encodings, and field names **MUST** follow the applicable ETSI binding or another binding explicitly defined by an applicable WE BUILD conformance specification; they are not redefined here. Each logical item below **MUST** be mapped to the selected binding.

## 8.1 Verify

*Direction:* QTSP → Authentic Source  
*Operation:* `Verify`  
*ETSI operation:* I2  
*Capability:* OPTIONAL under the WE BUILD common baseline; REQUIRED for ETSI TS 119 478 conformance
*Capability:* OPTIONAL API operation under the WE BUILD common baseline; REQUIRED for ETSI TS 119 478 conformance

When implemented, `Verify` **MUST** follow the selected binding. Where a deployment uses TS 119 478, the request **MUST** include the subject, candidate attribute data, and user-authorization context required by that binding. Any other binding and its request/response mapping and result semantics **MUST** be defined by an applicable WE BUILD conformance specification.

- **HTTP/OAuth:** I2 uses `POST /verify`. The HTTP status code indicates the operation-level result, while the response body carries the attribute-level results. The implementation **MUST** preserve TS 119 478's distinction between `Match`, `NoMatch`, `MatchWithVariation`, and `Unknown`. Full-attribute verification is required for TS conformance; fragment verification is optional.
- **ISO 15000:** I2 uses the TS 119 478 Verification query and response model, including its per-attribute results and defined success, failure, and deferred-response forms.

An implementation **MUST NOT** redefine or collapse the result semantics of the selected binding. A `Verify` result applies only to the attributes and subject represented in that request.

## 8.2 Retrieve

*Direction:* QTSP → Authentic Source  
*Operation:* `Retrieve`  
*ETSI operation:* I3  
*Capability:* REQUIRED for WE BUILD conformance; OPTIONAL under ETSI TS 119 478

Where a deployment uses TS 119 478, the request and response **MUST** follow the selected ETSI binding. Any other binding **MUST** be defined by an applicable WE BUILD conformance specification; conformance to that binding alone **MUST NOT** be claimed as TS 119 478 conformance. Implementations **MUST NOT** assume that fields in one binding map directly to fields in the other.

- **HTTP/OAuth:** I3 uses `POST /retrieve`. The request body contains the requested attribute identifiers; subject identification and user authorization are obtained through I4 and conveyed as specified by the TS 119 478 OAuth flow. A successful response contains the retrieved attributes and provider identification, and identifies the Authentic Source separately when the ASIP is an intermediary. The HTTP status codes and response object **MUST** follow TS 119 478 clause 6.1.2, including its `200`, `400`, `401`, `404`, and `501` semantics.
- **ISO 15000:** I3 uses the TS 119 478 Retrieve query and response model. The request carries the subject and attribute details and the required request context; the response uses the specified per-attribute `RetrieveResult` and the defined success, failure, or deferred-response form.

An applicable WE BUILD conformance specification or source-service agreement **MAY** define additional transaction-correlation or freshness requirements. Such requirements **MUST NOT** require fields to be added to an ETSI message in a way that breaks the selected binding; extensions **MUST** use a mechanism permitted by that binding.

## 8.3 Transport and security binding

TS 119 478 defines two alternatives for the Authentic Source interface. An implementation claiming conformance to that standard **MUST** support at least one of the following; it **MAY** support both:

- HTTP with OAuth 2.0, including I2 `Verify`, I3 `Retrieve` if implemented, and I4 `Authorize` as specified by TS 119 478 clauses 6.1.1–6.1.3; or
- ISO 15000 / ebXML messaging, including I2 `Verify` and I3 `Retrieve` if implemented, as specified by TS 119 478 clauses 6.2.1–6.2.4.

The WE BUILD common baseline in this specification **MUST** require I3 `Retrieve` even though TS 119 478 makes it optional. The deployment documentation **MUST** identify:

- the selected binding and exact TS 119 478 edition;
- supported data sets, attribute identifiers, schemas, and subject identification data;
- endpoint information and, if used, the I1 Discover mechanism or another source-discovery method;
- any stricter security, operational, timeout, retry, and service-level requirements.

I1 `Discover` is not required by this specification's common baseline. Where a deployment uses I1, it **MUST** apply the TS 119 478 discovery model for attribute metadata and data-service endpoints. The BU2 EUCC deployment **SHOULD** use the TS 119 478 I3 `Retrieve` operation. A source that claims TS 119 478 conformance **MUST** also implement I2 `Verify`, even if the BU2 issuance flow uses retrieval for a particular transaction.

# 9. Conformance

An implementation conforms to this specification only for the role(s) and data sets it declares, and only where it meets the applicable requirements in Section 7 and supports the relevant flows in Section 6.

An **Authentic Source Interface Provider** conforms to the WE BUILD common baseline if it provides `Retrieve` for each declared data set and meets the requirements in Section 7.3. `Verify` support is declared separately and is optional unless another applicable WE BUILD conformance specification requires it.

A provider claiming **ETSI TS 119 478 conformance** must additionally implement I2 `Verify`, select at least one of the two ETSI bindings, and satisfy all applicable requirements of the selected binding, including its user authorization and access-control requirements. WE BUILD conformance alone does not imply ETSI conformance.

A **QTSP** conforms to this specification for the source and data sets it supports if it implements the `Retrieve` client, meets the QTSP requirements in Section 7.2, and satisfies applicable ETSI TS 119 471 requirements for the attestation service it operates. A claim of TS 119 478 conformance additionally requires compliance with the TSP requirements in the selected TS 119 478 binding.

Conformance claims **MUST** identify:

- the role and supported data sets;
- the supported data sets, attribute identifiers, and subject identifier schemes;
- whether `Verify` is supported; and
- the applicable protocol binding and its version.

Other applicable WE BUILD conformance specifications **MAY** define stricter requirements for specific use cases. They **MUST NOT** weaken the mandatory requirements in this specification.

# 10.  Data protection and security

The QTSP and Authentic Source Interface Provider **MUST**:

1. use the authenticated transport and message-protection mechanisms required by the selected binding and this specification;
2. apply access control so that a QTSP can invoke only the operations and data sets for which it is authorised;
3. protect credentials, keys, and tokens against disclosure, misuse, and unauthorised reuse, and maintain a process for rotation and revocation;
4. minimise personal and business data in requests, responses, and logs;
5. retain and disclose transaction evidence only as required for the service, applicable law, and the agreed retention policy; and
6. define operational procedures for security incidents, suspected compromise, planned maintenance, and service unavailability.

Where the deployment uses the TS 119 478 HTTP/OAuth binding, the implementation **MUST** meet its requirements for OAuth 2.0 authorization-code flow, user identification and authorization, PKCE, client authentication, and sender-constrained access tokens. Where it uses the ISO 15000 binding, authorization and evidence that the request is user-authorized **MUST** be handled as required by TS 119 478. These binding-specific requirements **MUST NOT** be replaced with a generic mutual-TLS requirement. For HTTP/OAuth client registration, the QTSP **SHOULD** consider the TS 119 478 recommendation to use Wallet-Relying Party Access Certificates (WRPAC) under ETSI TS 119 411-8 and to validate QTSP entitlement through the applicable national register. Authentication of the interface provider alone does not establish that it is authoritative for a particular data set; that authority must be established separately.

# References

[1] ETSI, *ETSI TS 119 471 V1.1.1 (2025-05), Electronic Signatures and Trust Infrastructures (ESI); Policy and Security requirements for Providers of Electronic Attestation of Attributes Services*. Clauses 4.2.1.1, 4.2.2.1, and 4.2.2.2 are particularly relevant to this specification.

[2] ETSI, *ETSI TS 119 478 V1.1.1 (2026-01), Electronic Signatures and Trust Infrastructures (ESI); Specification of interfaces related to Authentic Sources*. Clauses 4–6 and Annexes A–C.

[3] ETSI, *TS 119 411-8, Electronic Signatures and Trust Infrastructures (ESI); Policy and security requirements for Trust Service Providers issuing certificates; Part 8: Access Certificate Policy for EUDI Wallet Relying Parties*. Applicable edition to be identified by the deployment.

[4] WE BUILD Consortium, *Specification Scenario – Issuance of EU Company Certificate, EUCC*, BU2, QTSP-powered issuance flow, branch `Specifications_issuanceEUCC_QTSP-powered-flow_update` (2026).
