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

### Consumer ###

To search for a list of prescriptions the external system will make an HTTP request which should include, as a minimum, the following parameters:

- NHS Number
- Format (this is a fixed value of ‘trace-summary’. Introduced for forwards compatibility)

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

> The Spine Interaction Id for this operation is `ExternalPrescriptionSearch_1_0`.

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

### Request Response ###

The output is a proprietary JSON format, the content type is 'application/json'and elements will appear in no particular order.

The HTTP status will, under most circumstances be 200. If the query is successful the statusCode will be '0' and the reason will be '' (empty string).

#### Response Headers ####

No special response headers are utilised.

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

*Successful search with multiple prescription found*

```json
{
  "reason": "",
  "version": "1",
  " prescriptionList ": {
    "D29E5B-Z5C475-11E6AR": {
      "prescriptionTreatmentType": {
        "prescriptionTreatmentTypeText": "Repeat Prescribing",
        "prescriptionTreatmentTypeCode": "0002"
      },
      "prescriptionIssueDate": "20150909194211",
      "pendingCancellations": "False",
      "lastEventDate": "20150909194214",
      "currentIssueNumber": "1",
      "issues": {
        "1": {
          "issueDate": "20150909",
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
      "patientNhsNumber": "9990406480"
    },
    "D2A223-Z2C475-11E6AQ": {
      "prescriptionTreatmentType": {
        "prescriptionTreatmentTypeText": "Repeat Prescribing",
        "prescriptionTreatmentTypeCode": "0002"
      },
      "prescriptionIssueDate": "20150909194211",
      "pendingCancellations": "False",
      "lastEventDate": "20150909194214",
      "currentIssueNumber": "1",
      "issues": {
        "1": {
          "issueDate": "20150909",
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
      "patientNhsNumber": "9990406480"
    },
    "D2A238-Z9E475-11E6AE": {
      "prescriptionTreatmentType": {
        "prescriptionTreatmentTypeText": "Repeat Prescribing",
        "prescriptionTreatmentTypeCode": "0002"
      },
      "prescriptionIssueDate": "20150909194211",
      "pendingCancellations": "False",
      "lastEventDate": "20150909194214",
      "currentIssueNumber": "1",
      "issues": {
        "1": {
          "issueDate": "20150909",
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
      "patientNhsNumber": "9990406480"
    },
    "D2A080EE-475C-11E6-A68F-08002732570CN": {
      "prescriptionTreatmentType": {
        "prescriptionTreatmentTypeText": "Repeat Prescribing",
        "prescriptionTreatmentTypeCode": "0002"
      },
      "prescriptionIssueDate": "20150909194211",
      "pendingCancellations": "False",
      "lastEventDate": "20150909194214",
      "currentIssueNumber": "1",
      "issues": {
        "1": {
          "issueDate": "20150909",
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
      "patientNhsNumber": "9990406480"
    },
    "D2A22A-Z70475-11E6AH": {
      "prescriptionTreatmentType": {
        "prescriptionTreatmentTypeText": "Repeat Prescribing",
        "prescriptionTreatmentTypeCode": "0002"
      },
      "prescriptionIssueDate": "20150909194211",
      "pendingCancellations": "False",
      "lastEventDate": "20150909194214",
      "currentIssueNumber": "1",
      "issues": {
        "1": {
          "issueDate": "20150909",
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
      "patientNhsNumber": "9990406480"
    },
    "D2A22C50-475C-11E6-A68F-08002732570CN": {
      "prescriptionTreatmentType": {
        "prescriptionTreatmentTypeText": "Repeat Prescribing",
        "prescriptionTreatmentTypeCode": "0002"
      },
      "prescriptionIssueDate": "20150909194211",
      "pendingCancellations": "False",
      "lastEventDate": "20150909194214",
      "currentIssueNumber": "1",
      "issues": {
        "1": {
          "issueDate": "20150909",
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
      "patientNhsNumber": "9990406480"
    }
  },
  "statusCode": "0"
}
```
