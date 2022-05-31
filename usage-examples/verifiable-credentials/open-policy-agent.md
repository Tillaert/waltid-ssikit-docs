---
description: Credential validation powered by the Open Policy Agent
---

# Open Policy Agent

The Open Policy Agent ([https://www.openpolicyagent.org](https://www.openpolicyagent.org)) is an open source, general-purpose policy engine that unifies policy enforcement. OPA provides a high-level declarative language called [Rego](https://www.openpolicyagent.org/docs/latest/#rego) that lets you specify policy as code in order to offload policy decision-making from your business logic.&#x20;

The SSI Kit offers an integration with OPA and therefore allows the flexible validation of W3C Verifiable Credentials by the execution of Rego policies.

## Integration of the Open Policy Agent with the SSI Kit

The following graphic illustrates the technical architecture how a custom application can verify credentials by utilizing the Open Policy Agent.

![SSI Kit and the Open Policy Agent](<../../.gitbook/assets/opa (1).png>)

In order to verify W3C Verifiable Credentials and Presentations, the SSI Kit offers the [Auditor API](https://auditor.ssikit.walt.id/). This API serves as integration point for a Verifier application, but also can be used for testing by the built-in CLI tool. In either way a Verifiable Credential (VC) is forwarded to the SSI Kit in order to have it verified.

The SSI Kit loads a Rego Policy either from a file-system, database or a trusted registry that most likely is implemented using Distributed Ledger Technology.

Further on the SSI Kit generates the verification request which is processed by the OPA engine. This request consists of the policy, the input-data to be verified and the action. The input-data is just the relevant data-points of the credential - typically the nested Json object "credentialSubject" or part of it. The "action" is the request that should be granted by the policy.&#x20;

The Open Policy Agent processes the verification request and returns the result to the SSI Kit. The SSI Kit evaluates the result and composes an aggregated credential validation response (as aso other elements of the credential are verified) for the calling party.&#x20;

## Example request

The following command shows how to validate a credential based on a Rego/OPA Policy.

```
./ssikit.sh vc verify rego-vc.json -p RegoPolicy='{"dataPath" : "$.credentialSubject.holder", "input" : "{\"user\": \"did:ebsi:ze2dC9GezTtVSzjHVMQzpkE\", \"action\": \"apply_to_masters\", \"location\": \"Slovenia\" }", "rego" : "src/test/resources/rego/test-policy.rego", "resultPath" : "$.result[0].expressions[0].value.allow"}'
```

Detailed explanation of parameters:

The standard command for validating VCs is: **./ssikit.sh vc verify \<vc-file>**. In the case of above the following credential is placed in file **rego-vc.json**.&#x20;

```
{
  "@context" : [ "https://www.w3.org/2018/credentials/v1" ],
  "credentialSchema" : {
    "id" : "https://api.preprod.ebsi.eu/trusted-schemas-registry/v1/schemas/0xb77f8516a965631b4f197ad54c65a9e2f9936ebfb76bae4906d33744dbcc60ba",
    "type" : "FullJsonSchemaValidator2021"
  },
  "credentialSubject" : {
    "holder" : {
      "constraints" : {
        "location" : "Slovenia"
      },
      "grant" : "apply_to_masters",
      "id" : "did:ebsi:ze2dC9GezTtVSzjHVMQzpkE",
      "role" : "family"
    },
    "id" : "did:ebsi:zvXXbgxmw6xzeQqQdvD3N2w",
    "policySchemaURI" : "https://raw.githubusercontent.com/walt-id/waltid-ssikit/master/src/test/resources/verifiable-mandates/test-policy.rego"
  },
  "evidence" : [ {
    "evidenceValue" : "",
    "id" : "https://essif.europa.eu/tsr-va/evidence/f2aeec97-fc0d-42bf-8ca7-0548192d5678",
    "type" : [ "VerifiableMandatePresentation" ]
  } ],
  "id" : "urn:uuid:942f6892-3b50-4f9c-9c5d-3651da11db00",
  "issued" : "2022-05-31T06:11:15.641814310Z",
  "issuer" : "did:ebsi:zvXXbgxmw6xzeQqQdvD3N2w",
  "validFrom" : "2022-05-31T06:11:15.641816751Z",
  "issuanceDate" : "2022-05-31T06:11:15.641816751Z",
  "type" : [ "VerifiableCredential", "VerifiableMandate" ],
  "proof" : {
    "type" : "EcdsaSecp256k1Signature2019",
    "creator" : "did:ebsi:zvXXbgxmw6xzeQqQdvD3N2w",
    "created" : "2022-05-31T06:11:21Z",
    "domain" : "https://api.preprod.ebsi.eu",
    "nonce" : "2fb595d1-f53e-4f6b-8da3-b5a3795a09e0",
    "proofPurpose" : "assertion",
    "jws" : "eyJiNjQiOmZhbHNlLCJjcml0IjpbImI2NCJdLCJhbGciOiJFUzI1NksifQ..UP1pRh3bX6QNc22GA8y-Zi3YkQdC-4k4e241xQtR3Rw-jzx15aQdXrR8RcvtEZxMCb-xidOfH8K9SVSvxBer9w"
  }
}
```

The argument "-p" is used for specifiying a built-in VerificationPolicy of the SSI Kit. To review the existing policies feel free to access the[ policy API of the Auditor](https://auditor.ssikit.walt.id/v1/swagger#/Verification%20Policies/listPolicies).

The **RegoPolicy** indicates a validation process by utilizing the Open Policy Agent. As shown in the example the RegoPolicy can be parameterized in order to flexibly configure the validation request.&#x20;

In this example the RegoPolicy takes the following input:

```
{
  "dataPath": "$.credentialSubject.holder",
  "input": "{\"user\": \"did:ebsi:ze2dC9GezTtVSzjHVMQzpkE\", \"action\": \"apply_to_masters\", \"location\": \"Slovenia\" }",
  "rego": "src/test/resources/rego/test-policy.rego",
  "resultPath": "$.result[0].expressions[0].value.allow"
}
```

**dataPath**: This optional attribute is the Json-path to point-out which nested element of the Verifiable Credential that should be used as input data for the OPA engine.

**input**: The validation request, which is the permission that is asked for.

**rego**: File path to the rego policy.

**resultPath**: As the output of the OPA engine is a Json object, this optional parameter allows to specify which part of the object should be used to determine if the result is either **true** or **false**.