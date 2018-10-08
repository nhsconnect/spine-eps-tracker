---
title: Retrieve a prescription
keywords: develop
tags: [develop]
sidebar: overview_sidebar
permalink: develop_use_case_retrieve_a_prescription.html
summary: Retrieve a prescription
---

## API Use Case ##

This specification describes a single use case (i.e. Retrieve a prescription).

## Security ##

- Consumers and Producers SHALL utilise TLS Mutual Authentication (TLS-MA).

## Prerequisites ##

### Consumer ###

To retrieve a particular prescription the external system will make an HTTP request which should include, as a minimum, the following parameters:

- Prescription Id (this form part of the URL)
- Format (this is a fixed value of ‘trace’. Introduced for forwards compatibility)
- issueNumber (optional)
- Message version<sup>1</sup> (optional)

<sup>1</sup>message version indicates the version of the message response and is not used as part of the search

{% include note.html content="The additional “version” parameter which is not required in this release." %}

## API Usage ##

### Request Operation ###

#### Absolute Request ####

```http
GET https://[eps_tracker_server]/mm/prescriptions/{prescriptionId}?format=trace&version={version}&issueNumber{issueNumber}
```

> [eps_tracker_server] = mm-sync.national.ncrs.nhs.uk

#### Request Headers ####

Consumers SHALL include the following additional HTTP request headers:

| Header               | Value |
|----------------------|-------|
| `Accept`             | `application/json` |
| `Spine-From-Asid`    | Consumer's ASID |
| `Spine-UserId`            | User ID |
| `Spine-RoleProfileId`     | Role Profile ID |

Consumers MAY include the following additional HTTP request headers:

| Header               | Value |
|----------------------|-------|
| `Eps-TraceId`             | Message Trace ID |


The incoming headers are validated to ensure the correct type and length of parameters:

| Parameter | Mandatory | Length | Restrictions | Used in search |
|-----------|-----------|--------|--------------|----------------|
| `Spine-From-Asid` | Y | 12 | 12 digits | N |
| `Spine-UserId`    | Y |	12 | 12 digits | N |
| `Spine-RoleProfileId` | Y | 12 | 12 digits | N |
| `Eps-TraceId` | N | Max 30 | Up to 30 characters (upper or lower case), digits or the – (dash) | N |

#### Payload Request Parameters ####

The incoming parameters are validated to ensure the correct length of parameters and allowable characters.

The parameter names are as follows, note that these are case sensitive:

| Parameter | Type | Mandatory | Length / Restrictions | Used in search |
|-----------|------|-----------|-----------------------|----------------|
| `PrescriptionId`      |	URL       | Y | 19/20 or 36/37 <br/> A-Z, a-z, 0-9 and - (dash) and + (plus) | Y |
| `issueNumber`         | parameter   | N<sup>1</sup> |	Integer | Y |
| `version`             | parameter   | N | Must be either the previous or current version of the service | N |

<sup>1</sup>If not supplied the current issue (instance) will be used. If specified and the instance doesn't exist the rest of the prescription details will be returned but the instance details will be left blank.

> The Spine Interaction Id for this operation is `ExternalPrescriptionQuery_1_0`.

#### Error Handling ####

If the query is unsuccessful the statusCode not be '0' and the reason will be populated.

*Unsuccessful retrieve as prescription not found*

```json
{
  "reason": "Not found",
  "version": "1.0",
  "prescription": {

  },
  "statusCode": "1"
}
```

### Request Response ###

#### Response Headers ####

No special response headers are utilised.

#### Payload Response Body ####

Provider systems:

- SHALL return a `200` **OK** HTTP status code on successful execution of the operation.

```json
{
  "reason": "",
  "version": "1",
  "prescription": {
    "prescriptionType": {
      "prescriptionTypeText": "GENERAL PRACTITIONER PRESCRIBING",
      "prescriptionTypeCode": "0001"
    },
    "prescriptionTreatmentType": {
      "prescriptionTreatmentTypeText": "Acute Prescription",
      "prescriptionTreatmentTypeCode": "0001"
    },
    "nominatedDispenserODS": "FA666",
    "daysSupply": "28",
    "pendingCancellations": "False",
    "prescribingOrganisationName": "VERNON STREET MEDICAL CTR",
    "prescribingOrganisationContact": "01332332812",
    "signingDate": "20120108210751",
    "currentIssueNumber": "1",
    "prescribingOrganisationODS": "C81007",
    "prescriptionId": "000136-ZC2D5C-11E38K",
    "lastEventDate": "20140507100013",
    "nominatedDispenserName": "CROYDON PHARMACY",
    "issue": {
      "dispensingOrganisationODS": "FA666",
      "appliedCancellations": "False",
      "lastDispenseDate": "20140507",
      "dispensingOrganisationContact": "",
      "issueNumber": "1",
      "lineItems": {
        "1": {
          "status": {
            "statusText": "Item fully dispensed",
            "statusCode": "0001"
          },
          "id": "02ED7776-21CD-4E7B-AC9D-D1DBFEE7B8CF"
        },
        "2": {
          "status": {
            "statusText": "Item fully dispensed",
            "statusCode": "0001"
          },
          "id": "45D5FB11-D793-4D51-9ADD-95E0F54D2786"
        }
      },
      "prescriptionStatus": {
        "statusText": "Claimed",
        "statusCode": "0008"
      },
      "dispensingOrganisationName": "CROYDON PHARMACY"
    },
    "patientNhsNumber": "9912003446"
  },
  "statusCode": "0"
}
```

## Examples ##

### C# ###

```csharp
Hello World
```

### Java ###

```java
Hello World
```
