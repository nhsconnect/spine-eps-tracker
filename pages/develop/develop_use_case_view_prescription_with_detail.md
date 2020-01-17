---
title: Retrieve detailed prescription information
keywords: develop
tags: [develop]
sidebar: overview_sidebar
permalink: develop_use_case_view_prescription_with_detail.html
---

## API Use Case ##

As a clinician I would like to view a prescription for a patient who has presented to me
so that I can base referral and prescribing information on the specific state of a prescription.

## Security ##

- API clients SHALL utilise TLS Mutual Authentication (TLS-MA).

## Prerequisites ##

### Endpoint Authentication ###
The Spine Interaction Id for this operation is `urn:nhs:names:services:mmquery:NHS111_ItemDetail` - endpoints must be registered to use this interaction.

### User Authentication ###
Users must be authenticated using Spine Security Broker, and provide user details in the HTTP request header.

### Prescription Information ###
To view the detail of a prescription the requesting system will make an HTTP request which must include, as a minimum, the following parameters:

- Prescription ID (in the HTTP querystring)
- Requesting User Details (as HTTP custom headers `Spine-UserId` and `Spine-RoleProfileId`)
- Requesting System details (as the HTTP custom header `Spine-From-Asid`)

Optionally Requests can include:
- Line Item ID
- Prescription Issue Number

This information is likely to have been obtained from the results of a previous prescription search.

## API Usage ##

### Request ###

#### Absolute Request ####

To search for prescription ID 5B507C-Y90206-061F9E:

```bash
curl --cacert certs/cacerts --cert certs/int-eps.crt.pem --key keys/int-eps.key.pem --header 'Spine-From-Asid: 200000000946' --header 'Spine-UserId: 313175813564' --header 'Spine-RoleProfileId: 562926913100' --header 'Accept: application/json' https://[spine_host]/mm/nhs111itemdetails?prescriptionId=5B507C-Y90206-061F9E
```
> Host details and for all Spine environments can be retrieved from [Spine Assurance Portal](http://www.assurancesupport.digital.nhs.uk). (HSCN connectivity required). Visit the this site also to request the necessary TLS certificates.

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

#### Request Querystring Parameters ####

The incoming parameters are validated to ensure the correct format.

The parameter names are as follows, note that these are case sensitive:

| Parameter         | Type            | Mandatory | Length / Restrictions |
|-------------------|-----------------|-----------|-----------------------|
| `prescriptionId`  | query parameter | Y         | Must be a valid EPS R1 or R2 prescription ID, including checkdigit |
| `lineitemid`      | query parameter | N         | Uppercase UUID |
| `issueNumber`     | query parameter | N         | Integer - 1 based |


### API Response ###

The output is a proprietary JSON format, the content type is "application/json" and elements will appear in no particular order.

The HTTP status will, under most circumstances be `200`. If the query is successful the statusCode element in the request body will be `0` and the reason will be an empty string.

{% include note.html content="This API call response provides full details of medication in the lineItems element, include dosage, form, quantity, and SNOMED CT code" %}

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

See [Error Handling](develop_overview.html#error-handling) for details of possible error codes.

#### Response Headers ####

No custom response headers are utilised.

#### Response Body ####

An `200` **OK** HTTP status code will be returned on successful execution of the operation.

A simplified example of this is given below:

```json
{
 	"reason": "",
 	 "version": "1",
   "{Prescription ID}": { ... },
	 "statusCode": "0"
}
```


## API usage examples ##

### Scenario 1: An acute EPS Release 1 prescription

Given the following request:

```code
GET https://[spine_host]/mm/nhs111itemdetails?prescriptionId=9C18AE6F-510C-F7A3-E050-D20AE3A231C8C
```

The response details a prescription with two line items which is with the dispenser. 

#### Response Headers ####

```code
HTTP/1.1 200 OK
Date: Tue, 14 Jan 2020 11:52:04 GMT
Content-Type: application/json
Connection: keep-alive
Strict-Transport-Security: max-age=15724800; includeSubDomains
Etag: "7d8a7e431a3c71916c82631de7cfe990773a7baa"
Pragma: no-cache
Cache-Control: no-store
```
#### Response Body ####

```json
{
	"reason": "",
	"version": "1",
	"9C18AE6F-510C-F7A3-E050-D20AE3A231C8C": {
		"prescriptionDispensedDate": "False",
		"prescriptionTreatmentType": "Acute Prescription",
		"dispensingPharmacy": {
			"phone": "",
			"ods": "",
			"name": "",
			"address": ""
		},
		"prescriptionIssueDate": "20200114120100",
		"prescriptionDownloadDate": "20200114111540",
		"prescriptionLastIssueDispensedDate": "False",
		"pendingCancellations": "False",
		"epsVersion": "R1",
		"prescriptionClaimedDate": "",
		"nominatedPharmacy": {
			"phone": "",
			"ods": "",
			"name": "",
			"address": ""
		},
		"prescriber": {
			"phone": "01482711112",
			"ods": "B81001",
			"name": "DR ME AHMED'S PRACTICE",
			"address": "700 HOLDERNESS ROAD, HULL, NORTH HUMBERSIDE, HU9 3JA"
		},
		"lastEventDate": "20200114111540",
		"repeatInstance": {
			"totalAuthorised": "1",
			"currentIssue": "1"
		},
		"lineItems": {
			"9C18AE6F-511F-F7A3-E050-D20AE3A231C8": {
				"code": "408063002",
				"description": "Beclometasone 100micrograms/dose inhaler CFC free",
				"itemStatus": "With Dispenser",
				"dosage": "As Directed",
				"uom": "dose",
				"quantity": "200"
			},
			"9C18AE6F-5124-F7A3-E050-D20AE3A231C8": {
				"code": "320029006",
				"description": "Atorvastatin 10mg tablets",
				"itemStatus": "With Dispenser",
				"dosage": "As Directed",
				"uom": "tablet",
				"quantity": "28"
			}
		},
		"prescriptionStatus": "With Dispenser",
		"patientNhsNumber": "9467157349"
	},
	"statusCode": "0"
}
```


### Scenario 2: Issue 1 of a repeat dispensing EPS Release 2 prescription.

Given the following request:

```code
GET https://[spine_host]/mm/nhs111itemdetails?prescriptionId=74A4DF-N82668-00005V&issueNumber=1
```

The response shows issue 1 of prescription ID `74A4DF-N82668-00005V` 


#### Response Headers ####

```code
TTP/1.1 200 OK
Date: Tue, 14 Jan 2020 11:58:16 GMT
Content-Type: application/json
Connection: keep-alive
Strict-Transport-Security: max-age=15724800; includeSubDomains
Etag: "006979ed43406cffb9e28f7f262e20fa29b10b2c"
Pragma: no-cache
Cache-Control: no-store
```
#### Response Body ####

```json
{
	"reason": "",
	"74A4DF-N82668-00005V": {
		"prescriptionDispensedDate": "20200114",
		"prescriptionTreatmentType": "Repeat Dispensing",
		"dispensingPharmacy": {
			"phone": "",
			"ods": "FRN79",
			"name": "NORTHAMPTON PCT",
			"address": "HIGHFIELD, CLIFTONVILLE ROAD, NORTHAMPTON, NN1 5DN"
		},
		"prescriptionIssueDate": "20200114120100",
		"prescriptionDownloadDate": "20200114111531",
		"prescriptionLastIssueDispensedDate": "False",
		"pendingCancellations": "False",
		"epsVersion": "R2",
		"prescriptionClaimedDate": "",
		"nominatedPharmacy": {
			"phone": "",
			"ods": "FX012",
			"name": "",
			"address": ""
		},
		"prescriber": {
			"phone": "01669620339",
			"ods": "A84002",
			"name": "THE ROTHBURY PRACTICE",
			"address": "3 MARKET PLACE, ROTHBURY, MORPETH, NE65 7UW"
		},
		"lastEventDate": "20200114111555",
		"repeatInstance": {
			"totalAuthorised": "6",
			"currentIssue": "1"
		},
		"lineItems": {
			"9C18AE6F-513D-F7A3-E050-D20AE3A231C8": {
				"code": "17315811000001100",
				"description": "Certolizumab pegol 200mg/1ml solution for injection pre-filled syringes",
				"itemStatus": "Dispensed",
				"dosage": "As Directed",
				"uom": "syringe",
				"quantity": "2"
			},
			"9C18AE6F-5142-F7A3-E050-D20AE3A231C8": {
				"code": "3602511000001108",
				"description": "Lidocaine 400mg/20ml (2%) / Adrenaline (base) 100micrograms/20ml (1 in 200,000) solution for injection vials",
				"itemStatus": "Not Dispensed - Owing",
				"dosage": "As Directed",
				"uom": "vial",
				"quantity": "1"
			}
		},
		"prescriptionStatus": "With Dispenser - Active",
		"patientNhsNumber": "9467157977"
	},
	"version": "1",
	"statusCode": "0"
}
```


### Scenario 3: A repeat prescribing prescription at EPS Release 1

Given the following request:

```code
GET https://[spine_host]/mm/nhs111itemdetails?prescriptionId=9C18AE6F-510D-F7A3-E050-D20AE3A231C8K
```

#### Response Headers ####

```code
HTTP/1.1 200 OK
Date: Tue, 14 Jan 2020 11:55:07 GMT
Content-Type: application/json
Connection: keep-alive
Strict-Transport-Security: max-age=15724800; includeSubDomains
Etag: "632baa1f2756b5f7c9859171d19d1be970de35ae"
Pragma: no-cache
Cache-Control: no-store
```
#### Response Body ####

```json
{
	"reason": "",
	"version": "1",
	"9C18AE6F-510D-F7A3-E050-D20AE3A231C8K": {
		"prescriptionDispensedDate": "False",
		"prescriptionTreatmentType": "Repeat Prescribing",
		"dispensingPharmacy": {
			"phone": "",
			"ods": "",
			"name": "",
			"address": ""
		},
		"prescriptionIssueDate": "20200114120100",
		"prescriptionDownloadDate": "20200114111540",
		"prescriptionLastIssueDispensedDate": "False",
		"pendingCancellations": "False",
		"epsVersion": "R1",
		"prescriptionClaimedDate": "",
		"nominatedPharmacy": {
			"phone": "",
			"ods": "",
			"name": "",
			"address": ""
		},
		"prescriber": {
			"phone": "01945700223",
			"ods": "D81015",
			"name": "PARSON DROVE SURGERY",
			"address": "240 MAIN ROAD, PARSON DROVE, WISBECH, PE13 4JA"
		},
		"lastEventDate": "20200114111540",
		"repeatInstance": {
			"totalAuthorised": "1",
			"currentIssue": "1"
		},
		"lineItems": {
			"9C18AE6F-5129-F7A3-E050-D20AE3A231C8": {
				"code": "374296008",
				"description": "Levothyroxine sodium 100microgram tablets",
				"itemStatus": "With Dispenser",
				"dosage": "As Directed",
				"uom": "tablet",
				"quantity": "28"
			},
			"9C18AE6F-512E-F7A3-E050-D20AE3A231C8": {
				"code": "320000009",
				"description": "Simvastatin 40mg tablets",
				"itemStatus": "With Dispenser",
				"dosage": "As Directed",
				"uom": "tablet",
				"quantity": "28"
			}
		},
		"prescriptionStatus": "With Dispenser",
		"patientNhsNumber": "9467157349"
	},
	"statusCode": "0"
}

```
