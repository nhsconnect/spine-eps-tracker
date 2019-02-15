---
title: View Prescription With Detail
keywords: develop
tags: [develop]
sidebar: overview_sidebar
permalink: develop_use_case_view_prescription_with_detail.html
---

## API Use Case ##

As a clinician I would like to view a prescription for a patient who has presented to me
so that I can base referral and prescribing information on the specific state of a prescription.

## Security ##

- Consumers and Producers SHALL utilise TLS Mutual Authentication (TLS-MA).
- Producers SHALL verify that the requesting system's entry on Spine Directory Service is registered to use the interaction.

## Prerequisites ##

### Endpoint Authentication ###
The Spine Interaction Id for this operation is `urn:nhs:names:services:mmquery:NHS111_ItemDetail` - endpoints must be registered to use this interaction.

### User Authentication ###
Users must be authenticated using Spine Security Broker, and provide user details in the request header.

### Prescription Information ###
To view the detail of a prescription the requesting system will make an HTTP request which must include, as a minimum, the following parameters:

- Prescription ID
- Requesting User Details
- Requesting System details

Optionally Requests can include:
- Line Item ID
- Prescription Issue Number

This information is likely to have been obtained from the results of a previous prescription search.

## API Usage ##

### Request ###

#### Absolute Request ####

To search for prescription ID 5B507C-Y90206-061F9E:

```bash
curl --cacert certs/cacerts --cert certs/int-eps.crt.pem --key keys/int-eps.key.pem --header 'Spine-From-Asid: 200000000946' --header 'Spine-UserId: 313175813564' --header 'Spine-RoleProfileId: 562926913100' --header 'Accept: application/json' https://msg.int.spine2.ncrs.nhs.uk/mm/nhs111itemdetails?prescriptionId=5B507C-Y90206-061F9E
```
> Host details for all Spine environments can be retrieved from [Spine Assurance Portal](http://www.assurancesupport.digital.nhs.uk)

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

#### Request Parameters ####

The incoming parameters are validated to ensure the correct format.

The parameter names are as follows, note that these are case sensitive:

| Parameter         | Type            | Mandatory | Length / Restrictions |
|-------------------|-----------------|-----------|-----------------------|
| `prescriptionId`  | query parameter | Y         | Must be a valid EPS R1 or R2 prescription ID, including checkdigit |
| `lineitemid`      | query parameter | N         | Uppercase UUID |
| `issueNumber`     | query parameter | N         | Integer - 1 based |


### Responses ###
#### Error Handling ####

If the query is unsuccessful the statusCode not be '0' and the reason will be populated.

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

The output is a proprietary JSON format, the content type is "application/json" and elements will appear in no particular order.

The HTTP status will, under most circumstances be 200. If the query is successful the statusCode will be "0" and the reason will be "" (empty string).

#### Response Headers ####

No special response headers are utilised.

#### Payload Response Body ####

Provider systems:

- SHALL return a `200` **OK** HTTP status code on successful execution of the operation.

```json
{
 	"reason": "",
 	 "version": "1",
   "prescriptionList: { .......},
	 "statusCode": "0"
}
```

*Successful search with a single prescription found*

```json
{
  "reason": "",
  "version": "1",
  " prescriptionList": {
    "A3B4D9-Z42475-11E6B+": {
      "prescriptionTreatmentType": {
        "prescriptionTreatmentTypeText": "Repeat Dispensing",
        "prescriptionTreatmentTypeCode": "0003"
      },
      "prescriptionIssueDate": "20160711110721",
      "pendingCancellations": "False",
      "lastEventDate": "20160711110722",
      "currentIssueNumber": "1",
      "issues": {
        "1": {
          "issueDate": "False",
          "lineItems": {
            "1": {
              "status": {
                "statusText": "To Be Dispensed",
                "statusCode": "0007"
              }
            },
            "2": {
              "status": {
                "statusText": "To Be Dispensed",
                "statusCode": "0007"
              }
            }
          },
          "prescriptionStatus": {
            "statusText": "To Be Dispensed",
            "statusCode": "0001"
          }
        },
        "2": {
          "issueDate": "False",
          "lineItems": {
            "1": {
              "status": {
                "statusText": "To Be Dispensed",
                "statusCode": "0007"
              }
            },
            "2": {
              "status": {
                "statusText": "To Be Dispensed",
                "statusCode": "0007"
              }
            }
          },
          "prescriptionStatus": {
            "statusText": "Prescription future instance",
            "statusCode": "9000"
          }
        }
      },
      "patientNhsNumber": "9912003489"
    }
  },
  "statusCode": "0"
}
```

*Successful search with multiple prescriptions found*

```json
{
   "statusCode" : "0",
   "reason" : "",
   "version" : "1",
   "54A9B7-C81007-000014" : {
      "pendingCancellations" : "False",
      "prescriptionLastIssueDispensedDate" : "False",
      "prescriptionDownloadDate" : "20190213150209",
      "repeatInstance" : {
         "currentIssue" : "1",
         "totalAuthorised" : "1"
      },
      "prescriptionDispensedDate" : "20190215",
      "prescriptionTreatmentType" : "Repeat Prescribing",
      "lastEventDate" : "20190215105008",
      "prescriptionClaimedDate" : "20190215105008",
      "lineItems" : {
         "754B10D1-04DA-3828-E050-D20AE3A22BBD" : {
            "code" : "193611000001107",
            "quantity" : "84",
            "itemStatus" : "Not Dispensed",
            "dosage" : "As Directed",
            "description" : "Micronor 350microgram tablets (Janssen-Cilag Ltd)",
            "uom" : "tablet"
         },
         "754B10D1-04E9-3828-E050-D20AE3A22BBD" : {
            "code" : "330987003",
            "dosage" : "As Directed",
            "quantity" : "10",
            "itemStatus" : "Dispensed",
            "description" : "Sodium bicarbonate 5% ear drops",
            "uom" : "ml"
         },
         "754B10D1-04E4-3828-E050-D20AE3A22BBD" : {
            "quantity" : "250",
            "itemStatus" : "Dispensed",
            "dosage" : "As Directed",
            "code" : "159411000001102",
            "uom" : "ml",
            "description" : "Oilatum Emollient (GlaxoSmithKline Consumer Healthcare)"
         },
         "754B10D1-04DF-3828-E050-D20AE3A22BBD" : {
            "dosage" : "Inject subcutaneously as directed",
            "quantity" : "4",
            "itemStatus" : "Dispensed",
            "code" : "28426011000001108",
            "uom" : "pre-filled disposable injection",
            "description" : "Bydureon 2mg powder and solvent for prolonged-release suspension for injecti                                                               on pre-filled pen (AstraZeneca UK Ltd)"
         }
      },
      "nominatedPharmacy" : {
         "ods" : "",
         "name" : "",
         "address" : "",
         "phone" : ""
      },
      "patientNhsNumber" : "9651614498",
      "prescriber" : {
         "ods" : "C81007",
         "name" : "VERNON STREET MEDICAL CTR",
         "address" : "13 VERNON STREET, DERBY, DERBYSHIRE, DE1 1FW",
         "phone" : "01512631737"
      },
      "epsVersion" : "R2",
      "prescriptionIssueDate" : "20180907120900",
      "prescriptionStatus" : "Claimed",
      "dispensingPharmacy" : {
         "address" : "The Pharmacy, The Street, SW8 3QJ",
         "name" : "Test Pharmacy",
         "ods" : "FA00D",
         "phone" : "07775123456"
      }
   }
}
```
