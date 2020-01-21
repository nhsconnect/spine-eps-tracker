---
title: Retrieve basic prescription information
keywords: develop
tags: [develop]
sidebar: overview_sidebar
permalink: develop_use_case_retrieve_a_prescription.html
---

## API Use Case ##

This specification describes a single use case (i.e. Retrieve a prescription).

## Security ##

- API clients SHALL utilise TLS Mutual Authentication (TLS-MA).

## Prerequisites ##

### Endpoint Authentication ###
The Spine Interaction Id for this operation is `ExternalPrescriptionQuery_1_0` - endpoints must be registered to use this interaction.

### Consumer system ###

To retrieve a particular prescription the external system will make an HTTP request which should include, as a minimum, the following in the URL:

- PrescriptionId
- format (with a fixed value of `trace`. Introduced for forwards compatibility, specified in the query string)
- issueNumber (optional, specified in the query string)
- version<sup>1</sup> (optional, specified in the query string)

<sup>1</sup>message version indicates the version of the message response and is not used as part of the search

{% include note.html content="The additional “version” parameter which is not required in this release." %}

## API Usage ##

### Request Operation ###

#### Absolute Request ####

```http
GET https://[spine_host]/mm/prescriptions/{prescriptionId}?format=trace&version={version}&issueNumber{issueNumber}
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

#### HTTP Request Parameters ####

The incoming parameters are validated to ensure the correct length of parameters and allowable characters.

The parameter names are as follows, note that these are case sensitive:

| Parameter | Type | Mandatory | Length / Restrictions | Used in search |
|-----------|------|-----------|-----------------------|----------------|
| `PrescriptionId`      |	URL       | Y | 19/20 or 36/37 <br/> A-Z, a-z, 0-9 and - (dash) and + (plus) | Y |
| `issueNumber`         | parameter   | N<sup>1</sup> |	Integer | Y |
| `version`             | parameter   | N | Must be either the previous or current version of the service | N |

<sup>1</sup>If not supplied the current issue (instance) will be used. If specified and the instance doesn't exist the rest of the prescription details will be returned but the instance details will be left blank.



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

### API Response ###

#### Response Headers ####

No special response headers are utilised.

#### Response Body ####

An `200` **OK** HTTP status code will be returned on successful execution of the operation.

A simplified response body is given below:

```json
{
  "reason": "",
  "version": "1",
  "prescription": {
  },
  "statusCode": "0"
}
```

{% include note.html content="This API use case do not provide details of medication, or the dispensing organisation. The [Retrieve detailed prescription information](develop_use_case_view_prescription_with_detail.html) API use case provides this additional information." %}

## API Examples ##

### Scenario 1: Retrieve an repeat prescription ###

Given the following request:

```code
GET https://msg.int.spine2.ncrs.nhs.uk/mm/prescriptions/48A894-C86002-00009E?format=trace
```

The response shows the basic details of repeat prescription ID `48A894-C86002-00009E`. The prescription is with the dispensing organsation `FRN79`. The prescription has two items, one of which has already been dispensed.

#### Reponse Headers ####

```code
HTTP/1.1 200 OK
Content-Type: application/json; charset=UTF-8
Connection: keep-alive
Etag: "240ea7f0845a03cffd33d9ee3826be1027cfc74d"
```

#### Response Body ####

```json
{
	"reason": "",
	"version": "1",
	"prescription": {
		"prescriptionType": {
			"prescriptionTypeText": "PRIMARY CARE PRESCRIBER - PHARMACIST INDEPENDENT/SUPPLEMENTARY PRESCRIBER",
			"prescriptionTypeCode": "0108"
		},
		"prescriptionTreatmentType": {
			"prescriptionTreatmentTypeText": "Repeat Prescribing",
			"prescriptionTreatmentTypeCode": "0002"
		},
		"nominatedDispenserODS": "FX012",
		"daysSupply": "28",
		"pendingCancellations": "False",
		"prescribingOrganisationName": "XXX DO NOT USE XXX TEST GP PRACTICE 14",
		"prescribingOrganisationContact": "",
		"signingDate": "20200108120100",
		"currentIssueNumber": "1",
		"prescribingOrganisationODS": "Y90206",
		"prescriptionId": "48A894-C86002-00009E",
		"lastEventDate": "20200108144916",
		"nominatedDispenserName": "",
		"issue": {
			"dispensingOrganisationODS": "FRN79",
			"appliedCancellations": "False",
			"lastDispenseDate": "20200108",
			"dispensingOrganisationContact": "0113 397 4320",
			"issueNumber": "1",
			"lineItems": {
				"1": {
					"status": {
						"statusText": "Item not dispensed owing",
						"statusCode": "0004"
					},
					"id": "9BA2F6BD-1524-6F8B-E050-D20AE3A254FD"
				},
				"2": {
					"status": {
						"statusText": "Item fully dispensed",
						"statusCode": "0001"
					},
					"id": "9BA2F6BD-1529-6F8B-E050-D20AE3A254FD"
				}
			},
			"prescriptionStatus": {
				"statusText": "With dispenser active",
				"statusCode": "0003"
			},
			"dispensingOrganisationName": "ALLISON RL"
		},
		"patientNhsNumber": "9691003139"
	},
	"statusCode": "0"
}
```

### Scenario 2: Retrieve second issue of a repeat dispensing prescription ###

Given the following request:

```code
GET https://[spine_host]/mm/prescriptions/3AF2CB-C86002-0000BX?format=trace&issueNumber=2
```

The response shows the basic details of issue 2 of repeat dispensing prescription ID `3AF2CB-C86002-0000BX`. The prescription issue has two items. As this is a future instance, the items are not with the dispenser.

#### Reponse Headers ####

```code
HTTP/1.1 200 OK
Date: Wed, 08 Jan 2020 15:38:16 GMT
Content-Type: application/json; charset=UTF-8
Connection: keep-alive
Etag: "ae7f8ec260fb415f98228039c020dbd3662c8550"
```

#### Response Body ####

```json
{
	"reason": "",
	"version": "1",
	"prescription": {
		"prescriptionType": {
			"prescriptionTypeText": "GENERAL PRACTITIONER PRESCRIBING - TRAINEE DOCTOR/GP REGISTRAR",
			"prescriptionTypeCode": "0102"
		},
		"prescriptionTreatmentType": {
			"prescriptionTreatmentTypeText": "Repeat Dispensing",
			"prescriptionTreatmentTypeCode": "0003"
		},
		"nominatedDispenserODS": "FX012",
		"daysSupply": "5",
		"pendingCancellations": "False",
		"prescribingOrganisationName": "VERNON STREET MEDICAL CTR",
		"prescribingOrganisationContact": "0133 2332812",
		"signingDate": "20200108120100",
		"currentIssueNumber": "1",
		"prescribingOrganisationODS": "C81007",
		"prescriptionId": "3AF2CB-C86002-0000BX",
		"lastEventDate": "20200108144916",
		"nominatedDispenserName": "",
		"issue": {
			"dispensingOrganisationODS": "False",
			"appliedCancellations": "False",
			"lastDispenseDate": "False",
			"dispensingOrganisationContact": "",
			"issueNumber": "2",
			"lineItems": {
				"1": {
					"status": {
						"statusText": "To Be Dispensed",
						"statusCode": "0007"
					},
					"id": "9BA2F6BD-152E-6F8B-E050-D20AE3A254FD"
				},
				"2": {
					"status": {
						"statusText": "To Be Dispensed",
						"statusCode": "0007"
					},
					"id": "9BA2F6BD-1533-6F8B-E050-D20AE3A254FD"
				}
			},
			"prescriptionStatus": {
				"statusText": "Repeat dispense future instance",
				"statusCode": "9000"
			},
			"dispensingOrganisationName": ""
		},
		"patientNhsNumber": "9691003147"
	},
	"statusCode": "0"
}


```

