# eInvoice REST API
This page documents a national e-Invoicing REST API standard.  The work is governed by the [Australian Digital Business Council](http://digitalbusinesscouncil.com.au/).

The basic idea is that accounting software product vendors will build support for the standard so that any business can send an invoice to any other business so long as they know each other's business identifier (usually an ABN) and, optionally, have established a trading agreement to exchange electronic documents.  

## Overview
It works like a city phone book.  If party A wants to send an invoice to party B then party A will lookup the invoice service information for party B in the registry and then send the invoice to the specified location using mutually supported technical standards.  
* Each party publishes information about their invoice service (supported documents & transports, certificates, URL end points) to a registry.  In most cases, the accounting software package will do this for their customers.  The registry entry is only updated whenever there is a change to any party's technical profile.
* Optionally, a party may specify (via their service information registry entry) that a trading agreement is required before they will accept electronic documents.  In such cases, either party can initiate the trading request and, if agreed by the other party, the whitelist is updated with the relevant identifiers and the parties can exchange documents.
* Bidirectional document excahnge happens simply by looking up the other party's service data and certificate, encrypting the invoice data and sending it to the other oarty.
* Note that all interfaces are implemented as RESTFul services in accoridance with the [Australian Government API design guide](https://www.dto.gov.au/standard/design-guides/api/)
* .
![Overview](eInvoiceOverview.png)

## The API specifications

Are published to [swaggerhub](https://swaggerhub.com/api/ausdigital/invoice/0.1).

## The Registry



