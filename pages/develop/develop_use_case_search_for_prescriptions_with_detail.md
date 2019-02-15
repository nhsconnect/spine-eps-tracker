---
title: Search for prescriptions
keywords: develop
tags: [develop]
sidebar: overview_sidebar
permalink: develop_use_case_search_for_prescriptions_with_detail.html
---

## User Story ##

As a clinician delivering urgent care I would like to search for prescriptions for a patient who has presented to me so that I can get an overview of a patient's medication regime and find the correct prescription for me to review.

## Security ##

- Consumers and Producers SHALL utilise TLS Mutual Authentication (TLS-MA).
- Producers SHALL verify that the requestor's entry on Spine Directory Service is registered to use the prescription detail search interaction.

## Prerequisites ##

### Consumer ###

To search for a list of prescriptions the external system will make an HTTP request which must include the following parameter:

- NHS Number

This must have been traced and verified, so is likely to have been obtained from a PDS trace.

## API Usage ##

### Request ###

#### Example Request ####

```http
GET https://[spine_host]/mm/nhs111itemsummary?nhsNumber={nhsNumber}&format=trace-summary&earliestDate={earliestDate}&latestDate={latestDate}&prescriptionStatus={prescriptionStatus}&prescriptionVersion={prescriptionVersion}&version={version}
```
> Host details for all Spine environments can be retrieved from [Spine Assurance Portal](http://www.assurancesupport.digital.nhs.uk) [NHS network access only]

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
| `Eps-TraceId` | N | Max 30 | Up to 30 characters (upper or lower case), digits or `-` | N |

> The Spine Interaction Id for this operation is `urn:nhs:names:services:mmquery:NHS111_ItemSummary`.

#### Request Parameters ####

The incoming parameters are validated to ensure the correct length of parameters and allowable characters.

The parameter names are as follows, note that these are case sensitive:

| Parameter | Type              | Mandatory | Length / Restrictions | Used in search |
|-----------|-------------------|-----------|-----------------------|----------------|
| `nhsNumber` | query parameter | Y         | 10                    | Y              |

#### Error Handling ####

In most cases if the query is unsuccessful the statusCode is > 0 and the reason will be populated. If the query was correctly formed but the search returned no results the HTTP status code will be 200 as it will in most most cases where the query was invalid or not allowed. In some cases where the HTTP request is invalid or not supported by the server an HTTP status > 200 may be returned without a json response.

For example:

*Failed to run search as no NHS number was supplied*

```json
{
  "reason": "Invalid or missing NHS number",
  "version": "1",
  "prescriptions": {},
  "statusCode": "61"
}
```

### Response ###

The output is a proprietary JSON format, the content type is 'application/json' and elements will appear in no particular order.

The HTTP status will, under most circumstances be 200. If the query is successful the statusCode will be '0' or '' and the reason will be '' (empty string).

#### Response Headers ####

No special response headers are utilised.

#### Response Body ####

| Data Item           | Description                       | Notes |
| ------------------- | --------------------------------- | ----- |
| Prescription ID     | Unique Prescription ID            | Same for each item on a Prescription |
| patientNhsNumber    | Patient NHS Number                | 10 digits |
| prescriptionStatus  | Current status of the prescription (or issue) | Description - not coded |
| prescriptionTreatmentType | Type of prescription - Acute / Repeat Dispensing / Repeat Prescribing | Description - not coded |
| prescriptionIssueDate | Effective date of the prescription, which may be future dated |  - |
| repeatInstance      |  -                                | - |
| - currentIssue      | The current issue number          | 1 for acute prescriptions |
| - totalAuthorised   | The total authorised number of repeats | 1 for acute prescriptions |
| lastEventDate       | Date of latest activity on the prescription | - |
| epsVersion          | Version of the electronic prescription | e.g. R1, R2 |
| pendingCancellations | If true, indicates a pending cancellation flag applies to the prescription | String *not* boolean |
| Line Item ID         | Prescription Item ID              | UUID |
| Medication Name     | Medication description            | Medication name, strength and form as per dm+d concept definition |

#### Example Responses ####

*Prescription not found*
```json

```
*Prescription detail successfully found*

```json
{
   "statusCode" : "0",
   "reason" : "",
   "prescriptions" : {
      "611843-C81007-00001F" : {
         "prescriptionIssueDate" : "20181031121000",
         "prescriptionTreatmentType" : "Repeat Prescribing",
         "prescriptionStatus" : "Claimed",
         "pendingCancellations" : "False",
         "repeatInstance" : {
            "totalAuthorised" : "1",
            "currentIssue" : "1"
         },
         "lastEventDate" : "20190121163722",
         "patientNhsNumber" : "9651614498",
         "lineItems" : {
            "7983C385-B882-E99C-E050-D20AE3A22DF1" : "Sodium bicarbonate 5% ear drops",
            "7983C385-B87D-E99C-E050-D20AE3A22DF1" : "Oilatum Emollient (GlaxoSmithKline Consumer Healthcare)",
            "7983C385-B878-E99C-E050-D20AE3A22DF1" : "Bydureon 2mg powder and solvent for prolonged-release suspension for injection pre-filled pen (AstraZeneca UK Ltd)",
            "7983C385-B873-E99C-E050-D20AE3A22DF1" : "Micronor 350microgram tablets (Janssen-Cilag Ltd)"
         },
         "epsVersion" : "R2"
      },
      "54A9B7-C81007-000014" : {
         "prescriptionTreatmentType" : "Repeat Prescribing",
         "prescriptionIssueDate" : "20180907120900",
         "epsVersion" : "R2",
         "lineItems" : {
            "754B10D1-04E9-3828-E050-D20AE3A22BBD" : "Sodium bicarbonate 5% ear drops",
            "754B10D1-04DA-3828-E050-D20AE3A22BBD" : "Micronor 350microgram tablets (Janssen-Cilag Ltd)",
            "754B10D1-04E4-3828-E050-D20AE3A22BBD" : "Oilatum Emollient (GlaxoSmithKline Consumer Healthcare)",
            "754B10D1-04DF-3828-E050-D20AE3A22BBD" : "Bydureon 2mg powder and solvent for prolonged-release suspension for injection pre-filled pen (AstraZeneca UK Ltd)"
         },
         "patientNhsNumber" : "9651614498",
         "repeatInstance" : {
            "currentIssue" : "1",
            "totalAuthorised" : "1"
         },
         "lastEventDate" : "20190215105008",
         "prescriptionStatus" : "Claimed",
         "pendingCancellations" : "False"
      }
   },
   "version" : "1"
}
```
