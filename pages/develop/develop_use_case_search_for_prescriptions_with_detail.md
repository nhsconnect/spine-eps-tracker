---
title: Search for prescriptions
keywords: develop
tags: [develop]
sidebar: overview_sidebar
permalink: develop_use_case_search_for_prescriptions_with_detail.html
---

## User Story ##

As a clinician delivering urgent care I would like to search for prescriptions for a patient who has presented to me so that I can get an overview of a patient's medication regime and find to correct prescription for me to review.

## Security ##

- Consumers and Producers SHALL utilise TLS Mutual Authentication (TLS-MA).
- Producers SHALL verify that the requestor's entry on Spine Directory Service is registered to use the prescription detail search interaction.

## Prerequisites ##

### Consumer ###

To search for a list of prescriptions the external system will make an HTTP request which must include, as a minimum, the following parameters:

- NHS Number
- Format (this is a fixed value of `trace-summary`. Introduced for forwards compatibility)

In addition, the external system may also provide the following optional parameters:

- Prescription effective date range (earliest and latest date)<sup>1</sup>
- Prescription status
- Prescription Version
- Message version<sup>2</sup>

1. If `earliestDate` is not supplied it will default to 28 days ago. Note that this means that if no earliest date is supplied and the latest date is more than 28 days ago the earliest date will be after the latest date and no prescriptions will be found.
  - If `latestDate` is supplied it will include all of that date i.e. up until midnight of that date.
  - If `latestDate` is not supplied it will default to the current date/time.
  - If both `earliestDate` and `latestDate` are supplied and `latestDate` is earlier than `earliestDate` no prescriptions will be found.
2. message version indicates the version of the message response and is not used as part of the search.

## API Usage ##

### Request Operation ###

#### Absolute Request ####

```http
GET https://[spine_host]/mm/prescriptionsWithDetail?nhsNumber={nhsNumber}&format=trace-summary&earliestDate={earliestDate}&latestDate={latestDate}&prescriptionStatus={prescriptionStatus}&prescriptionVersion={prescriptionVersion}&version={version}
```
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

> The Spine Interaction Id for this operation is `urn:nhs:names:services:mmquery:NHS111_ItemSummary`.

#### Payload Request Parameters ####

The incoming parameters are validated to ensure the correct length of parameters and allowable characters.

The parameter names are as follows, note that these are case sensitive:

| Parameter | Type | Mandatory | Length / Restrictions | Used in search |
|-----------|------|-----------|-----------------------|----------------|
| `nhsNumber`           | parameter | Y | 10 | Y |
| `earliestDate`        | parameter | N | Must have the form yyyymmdd. If both earliest and latest date are supplied the earliest date cannot be later than the latest date | Y |
| `latestDate`          | parameter | N | Must have the form yyyymmdd | Y |
| `prescriptionStatus`  | parameter | N | Must be a valid (four digit) prescription state<sup>2</sup> | Y |
| `prescriptionVersion` | parameter | N | Must be ‘1’, ‘2’, ‘R1’ or ‘R2’ | Y |
| `version`             | parameter | N | Must be either the previous or current version of the service | N |

##### Prescription State #####

```code
    AWAITING_RELEASE_READY = '0000'
    TO_BE_DISPENSED = '0001'
    WITH_DISPENSER = '0002'
    WITH_DISPENSER_ACTIVE = '0003'
    EXPIRED = '0004'
    CANCELLED = '0005'
    DISPENSED = '0006'
    NOT_DISPENSED = '0007'
    CLAIMED = '0008'
    NO_CLAIMED = '0009'
    REPEAT_DISPENSE_FUTURE_INSTANCE = '9000'
    FUTURE_DATED_PRESCRIPTION = '9001'
    PENDING_CANCELLATION = '9005'
```

#### Error Handling ####

If the query is unsuccessful the statusCode is > '0' and the reason will be populated.

For example:

*Failed to run search as no NHS number was supplied*

```json
{
  "reason": "Invalid or missing NHS number",
  "version": "1",
  "prescriptions": {

  },
  "statusCode": "61"
}
```

### Request Response ###

The output is a proprietary JSON format, the content type is 'application/json' and elements will appear in no particular order.

The HTTP status will, under most circumstances be 200. If the query is successful the statusCode will be '0' or '' and the reason will be '' (empty string).

#### Response Headers ####

No special response headers are utilised.

#### Response Body ####

| Data Item | Description | Notes |
| patientNhsNumber | Patient NHS Number | - |
| Prescription ID | Unique Prescription ID | Same for each item on a Prescription
| lineItem ID | Prescription Item ID | UUID |
| repeatInstance |  - |  - |
| - currentIssue | The current issue number | 1 for acute prescriptions |
| - totalAuthorised | The total authorised number of repeats | 1 for acute prescriptions |
| Medication Name | Medication description | Medication name, strength and form as per dm+d concept definition |
| prescriptionStatus | Current status of the Prescription (or Instance) |  - |
| prescriptionIssueDate | Effective date of the prescription, which may be future dated |  - |
| lastEventDate | Date of latest activity on the prescription | - |
| prescriptionTreatmentType | Type of the Prescription | e.g. Acute, Repeat Dispensing, Repeat Prescription | - |
| epsVersion | Version of the electronic prescription | e.g. R1, R2 |
| pendingCancellations | If true, indicates a pending cancellation flag applies to the prescription | Boolean |


#### Payload Response Body ####

Provider systems:

- SHALL return a `200` **OK** HTTP status code on successful execution of the operation.

```json
{
 	"reason": "",
 	 "version": "1.0",
  	"prescriptionList: { .......}
    	 "statusCode": "0"
}
```

*Successful search with a single prescription found*

```json
{
  "version": "1",
  "reason": "",
  "statusCode": "0",
  "prescriptions": {
    "2D35F7-ZA0448-11E88Z": {
      "lastEventDate": "20180422095703",
      "prescriptionIssueDate": "20180117095703",
      "patientNhsNumber": "9912003489",
      "epsVersion": "R2",
      "repeatInstance": {
        "“currentIssue”": "2",
        "“totalAuthorised”": "“6”"
      },
      "pendingCancellations": "False",
      "prescriptionTreatmentType": "Repeat Dispensing",
      "prescriptionStatus": "Dispensed",
      "lineItems": {
        "30b7e9cf-6f42-40a8-84c1-e61ef638eee2": "Perindopril erbumine 2mg tablets",
        "636f1b57-e18c-4f45-acae-2d7db86b6e1e": "Metformin 500mg modified-release tablets"
      }
    },
    "ABC5F7-ZA0448-77E88X": {
      "lastEventDate": "20180319115010",
      "prescriptionIssueDate": "20180319101307",
      "patientNhsNumber": "9912003489",
      "epsVersion": "R2",
      "currentIssueNumber": "1",
      "pendingCancellations": "False",
      "prescriptionTreatmentType": "Acute Prescribing",
      "prescriptionStatus": "Dispensed",
      "lineItems": {
        "636f1b57-e18c-4f45-acae-2d7db86b6e1e": "Hydrocortisone 0.5% cream"
      }
    }
  }
}
```
