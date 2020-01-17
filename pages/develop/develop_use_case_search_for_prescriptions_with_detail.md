---
title: Search for detailed prescription information
keywords: develop
tags: [develop]
sidebar: overview_sidebar
permalink: develop_use_case_search_for_prescriptions_with_detail.html
---

## User Story ##

As a clinician delivering urgent care I would like to search for prescriptions for a patient who has presented to me so that I can get an overview of a patient's medication regime and find the correct prescription for me to review.

## Security ##

- API clients SHALL utilise TLS Mutual Authentication (TLS-MA).

## Prerequisites ##

### Endpoint Authentication ###
The Spine Interaction Id for this operation is `urn:nhs:names:services:mmquery:NHS111_ItemSummary` - endpoints must be registered to use this interaction.

### User Authentication ###
Users must be authenticated using the [Spine Security Broker](https://nhsconnect.github.io/FHIR-SpineCore/smartcards.html), and provide user details in the HTTP request header.

### Consumer system ###

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

#### HTTP Query String Parameters ####

The incoming parameters, found in the URL query string, are validated to ensure the correct length of parameters and allowable characters.

The parameter names are as follows, note that these are case sensitive:

| Parameter | Type | Mandatory | Length / Restrictions | Used in search |
|-----------|------|-----------|-----------------------|----------------|
| `nhsNumber`           | parameter | Y | 10 | Y |
| `earliestDate`        | parameter | N | Must have the form yyyymmdd. If both earliest and latest date are supplied the earliest date cannot be later than the latest date | Y |
| `latestDate`          | parameter | N | Must have the form yyyymmdd | Y |
| `prescriptionStatus`  | parameter | N | Must be a valid (four digit) prescription state<sup>2</sup> | Y |
| `prescriptionVersion` | parameter | N | Must be ‘1’, ‘2’, ‘R1’ or ‘R2’ | Y |
| `version`             | parameter | N | API major version number, fixed value of `1` | N |

*Prescription State*

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

See [Error Handling](develop_overview.html#error-handling) for details of possible error codes.

### API Response ###

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

{% include note.html content="This API use case provides details of medications in the `lineItems` element" %}


## API usage examples ##

### Scenario 1: Two prescriptions found - one acute, one repeat prescribing

Given the following request:

```code
GET https://[spine_host]/mm/nhs111itemsummary?nhsNumber=9467157349&format=trace-summary
```

The response shows, for NHS Number `9467157349`, two EPS Release 1 prescriptions. 
1. Prescription ID `9C18AE6F-510D-F7A3-E050-D20AE3A231C8K` is a repeat prescription with two line items.
2. Prescription ID `9C18AE6F-510C-F7A3-E050-D20AE3A231C8C`, an acute prescription

#### Response Headers ####

```code
HTTP/1.1 200 OK
Date: Tue, 14 Jan 2020 11:32:41 GMT
Content-Type: application/json
Connection: keep-alive
Strict-Transport-Security: max-age=15724800; includeSubDomains
Etag: "3b1f8a1ec407006396d38ae5c9f7525dfa6bfa29"
Pragma: no-cache
Cache-Control: no-store
```
#### Response Body ####

```json
{
	"reason": "",
	"version": "1",
	"prescriptions": {
		"9C18AE6F-510D-F7A3-E050-D20AE3A231C8K": {
			"prescriptionTreatmentType": "Repeat Prescribing",
			"prescriptionIssueDate": "20200114120100",
			"pendingCancellations": "False",
			"epsVersion": "R1",
			"lastEventDate": "20200114111540",
			"repeatInstance": {
				"dispenseHistory": {
					"1": {
						"dispenseDate": "False",
						"dispensingOrgName": "ALLISON RL",
						"dispensingOrgCode": "FRN79"
					}
				},
				"totalAuthorised": "1",
				"currentIssue": "1"
			},
			"lineItems": {
				"9C18AE6F-5129-F7A3-E050-D20AE3A231C8": "Levothyroxine sodium 100microgram tablets",
				"9C18AE6F-512E-F7A3-E050-D20AE3A231C8": "Simvastatin 40mg tablets"
			},
			"prescriptionStatus": "With Dispenser",
			"patientNhsNumber": "9467157349"
		},
		"9C18AE6F-510C-F7A3-E050-D20AE3A231C8C": {
			"prescriptionTreatmentType": "Acute Prescription",
			"prescriptionIssueDate": "20200114120100",
			"pendingCancellations": "False",
			"epsVersion": "R1",
			"lastEventDate": "20200114111540",
			"repeatInstance": {
				"dispenseHistory": {
					"1": {
						"dispenseDate": "False",
						"dispensingOrgName": "ALLISON RL",
						"dispensingOrgCode": "FRN79"
					}
				},
				"totalAuthorised": "1",
				"currentIssue": "1"
			},
			"lineItems": {
				"9C18AE6F-511F-F7A3-E050-D20AE3A231C8": "Beclometasone 100micrograms/dose inhaler CFC free",
				"9C18AE6F-5124-F7A3-E050-D20AE3A231C8": "Atorvastatin 10mg tablets"
			},
			"prescriptionStatus": "With Dispenser",
			"patientNhsNumber": "9467157349"
		}
	},
	"statusCode": "0"
}
```


### Scenario 2: Search returns two results -two active presriptions with dispenser

Given the following request:

```code
GET https://[spine_host]/mm/nhs111itemsummary?nhsNumber=9467157969&format=trace-summary
```

The response shows, for NHS Number `9467157969`, two prescriptions in the previous 28 days


#### Response Headers ####

```code
HTTP/1.1 200 OK
Date: Tue, 14 Jan 2020 11:33:57 GMT
Content-Type: application/json
Connection: keep-alive
Strict-Transport-Security: max-age=15724800; includeSubDomains
Etag: "acdc1bd3cb3d76a6b2267642b1bd3d7adb2f443a"
Pragma: no-cache
Cache-Control: no-store
```
#### Response Body ####

```json
{
	"reason": "",
	"version": "1",
	"prescriptions": {
		"0DF0C0-N82668-000039": {
			"prescriptionTreatmentType": "Repeat Prescribing",
			"prescriptionIssueDate": "20200114120100",
			"pendingCancellations": "False",
			"epsVersion": "R2",
			"lastEventDate": "20200114111556",
			"repeatInstance": {
				"dispenseHistory": {
					"1": {
						"dispenseDate": "20200114",
						"dispensingOrgName": "ALLISON RL",
						"dispensingOrgCode": "FRN79"
					}
				},
				"totalAuthorised": "1",
				"currentIssue": "1"
			},
			"lineItems": {
				"9C18AE6F-5138-F7A3-E050-D20AE3A231C8": "Metformin 500mg tablets",
				"9C18AE6F-5133-F7A3-E050-D20AE3A231C8": "Simvastatin 40mg tablets"
			},
			"prescriptionStatus": "With Dispenser - Active",
			"patientNhsNumber": "9467157969"
		},
		"296A99-N82668-00001R": {
			"prescriptionTreatmentType": "Acute Prescription",
			"prescriptionIssueDate": "20200114120100",
			"pendingCancellations": "False",
			"epsVersion": "R2",
			"lastEventDate": "20200114111555",
			"repeatInstance": {
				"dispenseHistory": {
					"1": {
						"dispenseDate": "20200114",
						"dispensingOrgName": "ALLISON RL",
						"dispensingOrgCode": "FRN79"
					}
				},
				"totalAuthorised": "1",
				"currentIssue": "1"
			},
			"lineItems": {
				"9C18AE6F-514C-F7A3-E050-D20AE3A231C8": "Isosorbide mononitrate 60mg modified-release tablets",
				"9C18AE6F-5147-F7A3-E050-D20AE3A231C8": "Ipratropium bromide 20micrograms/dose inhaler CFC free"
			},
			"prescriptionStatus": "With Dispenser - Active",
			"patientNhsNumber": "9467157969"
		}
	},
	"statusCode": "0"
}
```


### Scenario 3: A single result - a repeat dispensing prescription

Given the following request:

```code
GET https://[spine_host]/mm/nhs111itemsummary?nhsNumber=9467157977&format=trace-summary
```

The response shows, for NHS Number `9467157977`, shows a single EPS Release 2 presciption in the last 28 days. This is a repeat dispensing presciption with a single issue. 

#### Response Headers ####

```code
HTTP/1.1 200 OK
Date: Tue, 14 Jan 2020 11:35:19 GMT
Content-Type: application/json
Connection: keep-alive
Strict-Transport-Security: max-age=15724800; includeSubDomains
Etag: "a40c8303ff63acc6fa76be96219edd15009e249c"
Pragma: no-cache
Cache-Control: no-store
```
#### Response Body ####

```json
{
	"reason": "",
	"version": "1",
	"prescriptions": {
		"74A4DF-N82668-00005V": {
			"prescriptionTreatmentType": "Repeat Dispensing",
			"prescriptionIssueDate": "20200114120100",
			"pendingCancellations": "False",
			"epsVersion": "R2",
			"lastEventDate": "20200114111555",
			"repeatInstance": {
				"dispenseHistory": {
					"1": {
						"dispenseDate": "20200114",
						"dispensingOrgName": "ALLISON RL",
						"dispensingOrgCode": "FRN79"
					},
					"3": {
						"dispenseDate": "False",
						"dispensingOrgName": "Not Found",
						"dispensingOrgCode": "False"
					},
					"2": {
						"dispenseDate": "False",
						"dispensingOrgName": "Not Found",
						"dispensingOrgCode": "False"
					},
					"5": {
						"dispenseDate": "False",
						"dispensingOrgName": "Not Found",
						"dispensingOrgCode": "False"
					},
					"4": {
						"dispenseDate": "False",
						"dispensingOrgName": "Not Found",
						"dispensingOrgCode": "False"
					},
					"6": {
						"dispenseDate": "False",
						"dispensingOrgName": "Not Found",
						"dispensingOrgCode": "False"
					}
				},
				"totalAuthorised": "6",
				"currentIssue": "1"
			},
			"lineItems": {
				"9C18AE6F-513D-F7A3-E050-D20AE3A231C8": "Certolizumab pegol 200mg/1ml solution for injection pre-filled syringes",
				"9C18AE6F-5142-F7A3-E050-D20AE3A231C8": "Lidocaine 400mg/20ml (2%) / Adrenaline (base) 100micrograms/20ml (1 in 200,000) solution for injection vials"
			},
			"prescriptionStatus": "With Dispenser - Active",
			"patientNhsNumber": "9467157977"
		}
	},
	"statusCode": "0"
}
```
