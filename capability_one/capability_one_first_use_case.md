---
title: Capability One First Use Case
keywords: capability, use case
tags: [use_case,fhir]
sidebar: capability_one_sidebar
permalink: capability_one_first_use_case.html
summary: "First use case."
---

## Use Case ##

TODO

## Security ##

- TODO

## Prerequisites ##

### Consumer ###

The Consumer system:

- SHALL TODO.

## API Usage ##

### Request Operation ###

#### FHIR Relative Request ####

```http
POST /TODO
```

#### FHIR Absolute Request ####

```http
POST https://[proxy_server]/https://[provider_server]/[fhir_base]/TODO
```

#### Request Headers ####

Consumers SHALL include the following additional HTTP request headers:

| Header               | Value |
|----------------------|-------|
| `SSP-TraceID`        | Consumer's TraceID (i.e. GUID/UUID) |
| `SSP-From`           | Consumer's ASID |
| `SSP-To`             | Provider's ASID |
| `SSP-InteractionID`  | `urn:nhs:names:services:TODO`|

Example HTTP request headers:

```http
Connection: Keep-Alive
```

#### Payload Request Body ####

The following data-elements are mandatory (i.e. data MUST be present):

- TODO

The following data-elements are optional (i.e. data MAY be supplied):

- TODO

{% include tip.html content="This is a helpful tip about using this Open API." %} 

```xml
<OperationDefinition xmlns="http://hl7.org/fhir">
</OperationDefinition>
```

```json
{
}
```

The Provider system SHALL:

- TODO

#### Error Handling ####

The Provider system SHALL return an error if:

- TODO

### Request Response ###

#### Response Headers ####

```http
HTTP/1.1 200 OK
```

#### Payload Response Body ####

Provider systems:

- SHALL TODO.

```json
{
}
```

## Examples ##

### C# ###

TODO

### Java ###

TODO
