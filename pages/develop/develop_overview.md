---
title: Development Overview
keywords: develop
tags: [develop]
sidebar: overview_sidebar
permalink: develop_overview.html
summary: An overview of the high-level approach for the EPS tracker APIs.
---

## Security ##

Security of the service follows established mechanisms for synchronous queries on the Spine. Systems are required to be registered endpoints and will need to register the appropriate message set for their accredited system. They will have a Spine2 issued certificate and communications are secured over TLS mutual authentication. Thus only authorised external systems will have access to this service.

This will be enforced by checking that:

- the ASID of the sender matches that of the certificate being used to connect.
- the ASID of the sender is included in the list of allowed ASIDs for this interaction.

## Requests ##

All requests are HTTP GET requests.

## Response ##

The json response will follow a simple four part structure. Note that these sections are not ordered so may appear in any order.

Search for Basic prescriptions

![ePS Tracker Response](images/develop/eps-tracker-response.png)

The "Prescription Information" section differs by API use case as follows:

| API use case | Prescription Infomration json element |
| ---- | ------ |
| [Search for basic prescription information](develop_use_case_search_for_prescriptions.html) | PrescriptionList |
| [Retrieve basic prescription information](develop_use_case_retrieve_a_prescription.html) | prescription |
| [Search for detailed prescription information](develop_use_case_search_for_prescriptions_with_detail.html) | prescriptions |
| [Retrieve detailed prescription information](develop_use_case_view_prescription_with_detail.html) | {prescription ID} |


## Error Handling ##

If there is a problem, e.g. an invalid request parameter, or the prescription record can't be found, the HTTP response will still be 200 but the statusCode within the response will be a value other than '0' and the reason will indicate the nature of the problem. The request is rejected before any attempt is made to retrieve the prescription.

| Status Code | Reason Text | Meaning |
| 1 | Not found | Requested prescription was not found. |
| 2 | No unique prescription | found sibling. |
| 3 | Issue not found | The requested issue (instance) didn’t exist, the prescription detail will still be returned without the issue details. |
| 4 | Failed to parse prescription | The prescription record was malformed in some way. |
| 5 | Unexpected exception | Any problem other than those listed above. |
| 50 | Query parameters have not been provided | No parameters were supplied for this service. |
| 51 | Invalid prescription id | `prescriptionId` parameter was not supplied or does not adhere to validation rules. |
| 52 | Invalid or missing asid  | Asid parameter was not supplied or does not adhere to validation rules. |
| 53 | Invalid version  | `version` does not adhere to validation rules. |
| 54 | Invalid traceId | `traceId` does not adhere to validation rules. |
| 55 | Invalid userId  | `userId` does not adhere to validation rules. |
| 56 | Invalid roleProfileId  | `roleProfileId` does not adhere to validation rules. |
| 57 | Invalid search date  | The `earliestDate` or the `latestDate` parameter failed validation. |
| 59 | Invalid search prescription state  | The `prescriptionStatus` parameter failed validation. |
| 60 | Invalid search prescription version  | The `prescriptionVersion` parameter failed validation. |
| 61 | Invalid or missing NHS number  | The `nhsNumber` parameter was not supplied or failed validation. |
