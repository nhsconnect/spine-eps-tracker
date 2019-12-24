---
title: Introduction to Spine EPS Tracker
keywords: homepage
tags: [getting_started]
sidebar: overview_sidebar
permalink: index.html
toc: true
summary: A brief introduction to getting started.
---

## Introduction ##

An introduction to provide implementers of the EPS Prescription Tracker API with the information required to develop against the service.

## Purpose ##

The Prescription Tracker service provides a read only interface to obtain information about a patient’s prescriptions. It is written as a proprietary interface, though this is expected to be available as a set of FHIR resources in a future iteration. The tracker utilises a synchronous request / response pattern. It is synchronous with respect to HTTP connections which means that only a single HTTP connection is required to perform a complete request.

The prescription tracker provides a RESTful-style interface. Two resource types are available - a 'bare' prescription resource containing only non-sensitive information intended to show status and location information, and a fully detailed prescription resource including the clinical content of the prescription. All queries use a GET request returning a json response. All requests are idempotent in that they do not affect the status of prescriptions.
