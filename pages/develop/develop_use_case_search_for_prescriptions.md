---
title: Search for basic prescription information
keywords: develop
tags: [develop]
sidebar: overview_sidebar
permalink: develop_use_case_search_for_prescriptions.html
---

## API Use Case ##

This specification describes a single use cases (i.e. Search for a list of prescriptions).

## Security ##

- Consumers and Producers SHALL utilise TLS Mutual Authentication (TLS-MA).

## Prerequisites ##

### Endpoint Authentication ###
The Spine Interaction Id for this operation is `ExternalPrescriptionSearch_1_0` - endpoints must be registered to use this interaction.

### Consumer system ###

To search for a list of prescriptions the external system will make an HTTP request which should include, as a minimum, the following parameters:

- NHS Number
- Format (this is a fixed value of `trace-summary`. Introduced for forwards compatibility)

In addition, the external system may also provide the following optional parameters:

- Prescription date range (earliest and/or latest)<sup>1</sup>
- Prescription status
- Prescription Version
- Message version<sup>2</sup>

1. If `earliestDate` is not supplied it will default to 28 days ago. Note that this means that if no earliest date is supplied and the latest date is more than 28 days ago the earliest date will be after the latest date and no prescriptions will be found.
  - If `latestDate` is supplied it will include all of that date i.e. up until midnight of that date.
  - If `latestDate` is not supplied it will default to the current date/time.
  - If both `earliestDate` and `latestDate` are supplied and `latestDate` is earlier than `earliestDate` no prescriptions will be found.
2. message version indicates the version of the message response and is not used as part of the search.

{% include note.html content="The additional “version” parameter which is not required in this release." %}

## API Usage ##

### Request Operation ###

#### Absolute Request ####

```http
GET https://[spine_host]/mm/prescriptions?nhsNumber={nhsNumber}&format=trace-summary&earliestDate={earliestDate}&latestDate={latestDate}&prescriptionStatus={prescriptionStatus}&prescriptionVersion={prescriptionVersion}&version={version}
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

### API Response ###

The response uses a proprietary JSON format, the content type is `application/json` and elements will appear in no particular order.

The HTTP status will, under most circumstances be `200`. If the query is successful the statusCode will be `0` and the reason will be an empty string.

#### Response Headers ####

No special response headers are utilised.

#### Response Body ####

A `200` **OK** HTTP status code will be returned on successful execution of the API call.

A simplified response body is provided below:

```json
{
   "reason": "",
   "version": "1.0",
   "prescriptionList: {.......},
   "statusCode": "0"
}
```

{% include note.html content="This API use case do not provide details of medication, or the dispensing organisation. The [Search for detailed prescription information](develop_use_case_search_for_prescriptions_with_detail.html) API use case provides this additional information." %}

## API examples ##

### Scenario 1: Successful search finding two prescriptions ###

Given the following request:

```code
GET https://[spine_host]/mm/prescriptions?nhsNumber=9691003120&format=trace-summary
```

The response shows, for NHS Number `9691003120`, that in the previous 28 days, this person had the following presriptions:

1. Prescription with ID `9BA2F6BD-14FD-6F8B-E050-D20AE3A254FDA`, with two items, which was produced in an acute setting, and is currently with the dispenser.
2. A repeat prescription with ID `9BA2F6BD-14FE-6F8B-E050-D20AE3A254FDI`, with two items, also currently with dispenser. 


#### Reponse Headers ####

```code
HTTP/1.1 200 OK
Date: Wed, 08 Jan 2020 16:10:47 GMT
Content-Type: application/json
Connection: keep-alive
Etag: "bafcc747f93f0e3a668de87e61d1a2a477b97ce8"
```

#### Response Body ####

```json
{
	"prescriptionList": {
		"9BA2F6BD-14FD-6F8B-E050-D20AE3A254FDA": {
			"prescriptionTreatmentType": {
				"prescriptionTreatmentTypeText": "Acute Prescription",
				"prescriptionTreatmentTypeCode": "0001"
			},
			"prescriptionIssueDate": "20200108120100",
			"pendingCancellations": "False",
			"lastEventDate": "20200108144900",
			"currentIssueNumber": "1",
			"issues": {
				"1": {
					"issueDate": "20200108",
					"lineItems": {
						"1": {
							"status": {
								"statusText": "Item with dispenser",
								"statusCode": "0008"
							}
						},
						"2": {
							"status": {
								"statusText": "Item with dispenser",
								"statusCode": "0008"
							}
						}
					},
					"prescriptionStatus": {
						"statusText": "With Dispenser",
						"statusCode": "0002"
					}
				}
			},
			"patientNhsNumber": "9691003120"
		},
		"9BA2F6BD-14FE-6F8B-E050-D20AE3A254FDI": {
			"prescriptionTreatmentType": {
				"prescriptionTreatmentTypeText": "Repeat Prescribing",
				"prescriptionTreatmentTypeCode": "0002"
			},
			"prescriptionIssueDate": "20200108120100",
			"pendingCancellations": "False",
			"lastEventDate": "20200108144900",
			"currentIssueNumber": "1",
			"issues": {
				"1": {
					"issueDate": "20200108",
					"lineItems": {
						"1": {
							"status": {
								"statusText": "Item with dispenser",
								"statusCode": "0008"
							}
						},
						"2": {
							"status": {
								"statusText": "Item with dispenser",
								"statusCode": "0008"
							}
						}
					},
					"prescriptionStatus": {
						"statusText": "With Dispenser",
						"statusCode": "0002"
					}
				}
			},
			"patientNhsNumber": "9691003120"
		}
	},
	"reason": "",
	"version": "1",
	"statusCode": "0"
}
```

### Scenario 2: Successful search finding one repeat dispensing prescription ###

Given the following request:

```code
GET https://[spine_host]/mm/prescriptions?nhsNumber=9691003147&format=trace-summary
```

The response shows, for NHS Number `9691003147`, that in the previous 28 days, this person had a single prescription:

Prescription ID `3AF2CB-C86002-0000BX` is a repeat dispensing prescription, and the first of the batch of 6 issues of the prescription currently being dispensed. This is has two items, one of which thus far has been dispensed.


#### Reponse Headers ####

```code
HTTP/1.1 200 OK
Date: Wed, 08 Jan 2020 16:14:36 GMT
Content-Type: application/json
Connection: keep-alive
Etag: "8aa3b443d22146867cb8e9d36afb14dc9363bbc2"
```

#### Response Body ####

```json
{
	"prescriptionList": {
		"3AF2CB-C86002-0000BX": {
			"prescriptionTreatmentType": {
				"prescriptionTreatmentTypeText": "Repeat Dispensing",
				"prescriptionTreatmentTypeCode": "0003"
			},
			"prescriptionIssueDate": "20200108120100",
			"pendingCancellations": "False",
			"lastEventDate": "20200108144916",
			"currentIssueNumber": "1",
			"issues": {
				"1": {
					"issueDate": "20200108",
					"lineItems": {
						"1": {
							"status": {
								"statusText": "Item fully dispensed",
								"statusCode": "0001"
							}
						},
						"2": {
							"status": {
								"statusText": "Item not dispensed owing",
								"statusCode": "0004"
							}
						}
					},
					"prescriptionStatus": {
						"statusText": "With Dispenser - Active",
						"statusCode": "0003"
					}
				},
				"3": {
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
						"statusText": "Repeat Dispense future instance",
						"statusCode": "9000"
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
						"statusText": "Repeat Dispense future instance",
						"statusCode": "9000"
					}
				},
				"5": {
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
						"statusText": "Repeat Dispense future instance",
						"statusCode": "9000"
					}
				},
				"4": {
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
						"statusText": "Repeat Dispense future instance",
						"statusCode": "9000"
					}
				},
				"6": {
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
						"statusText": "Repeat Dispense future instance",
						"statusCode": "9000"
					}
				}
			},
			"patientNhsNumber": "9691003147"
		}
	},
	"reason": "",
	"version": "1",
	"statusCode": "0"
}

```


