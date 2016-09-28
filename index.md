---
title: Introduction
keywords: homepage
tags: [getting_started]
sidebar: overview_sidebar
permalink: index.html
toc: true
summary: A brief introduction to getting started.
---

## Introduction ##

This document provides users of the EPS Prescription Tracker with the information required to utilise the service. It is provided in lieu of an update to the External Interface Speciation. Spine2 is planning to provide a new EIS for all future messaging although at this stage such an artefact is not warranted for the Prescription Tracker.

## Message patterns and message structure ##

The Prescription Tracker service provides a read only interface to obtain information about a patient’s prescriptions. It is written as a proprietary interface as defined in the following document. The tracker utilises a synchronous request / response pattern. It is synchronous with respect to HTTP connections which means that only a single HTTP connection is required to perform a complete request. Future interfaces on Spine2 are will tend towards this simple pattern wherever possible with the intention of eventually removing the need for asynchronous ebXML communications.
The prescription tracker provides a REST interface with two queries – prescription search and prescription retrieve - allowing querying to take place utilising a simple syntax. Both queries utilise a GET request which is made to the service and which returns a json result. 
