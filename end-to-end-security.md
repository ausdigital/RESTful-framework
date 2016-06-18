# End to End Security Model

The e-invoicing REST specification provides a very high integrity end-to-end security model.

## Requreiments

The main concerns that the security model needs to address are:
* Registry identity integrity.  That users of the registry can trust that service information about a business genuinely belongs to that business.
* Business owner authorisation.  That updates to services are only made with the explicit authority of the business that owns the services.
* Invoice integrity.  That the recipient of an invoice can trust that the invoice really came from the identified business and hasnt been tampered with.
* Confidentiality.  That the commercially sensitive contents of invoices are not visible to any party other than the seller and the buyer.
* Auditability.  That there is an evidence trail for updates to the registry and invoice message routing.
* Availability.  That the network is avaialable, even when specific buyer or seller sysems are unavailable.

The REST e-invoicing specification meets all these requirements with very high integrity and efficiency using a completely paperless automated process.  A key feature is that the framework does not depend on intermediaries (eg access points or registries) to achieve end-to-end trust.  This both reduces the barriers to entry for competition in the service provider marketplace and increases overall security.

## Identity Integrity and OIDC

At the heart of the security model are a small number of identity providers  

- Write up Vanguard OIDC here - 

## Message integrity and PKI

All messages are both encrypted and signed using a PKI framework.

- write up message integrity using Vanguard RP certifiates and Registry based chain of trust. - 


