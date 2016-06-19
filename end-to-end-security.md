# End to End Security Model

The e-invoicing REST specification provides a very high integrity end-to-end security model.

## Requreiments

The main concerns that the security model needs to address are:
* Registry identity integrity.  That users of the registry can trust that service information about a business genuinely belongs to that business.
* Business owner authorisation.  That updates to services are only made with the explicit authority of the business that owns the services.
* Invoice integrity.  That the recipient of an invoice can trust that the invoice really came from the identified business and hasn't been tampered with.
* Confidentiality.  That the commercially sensitive contents of invoices are not visible to any party other than the seller and the buyer.
* Auditability.  That there is an evidence trail for updates to the registry and invoice message routing.
* Availability.  That the network is avaialable, even when specific buyer or seller sysems are unavailable.

The REST e-invoicing specification meets all these requirements with very high integrity and efficiency using a completely paperless and automated process.  A key feature is that the framework does not depend on trust of intermediaries (eg access points or registries) to achieve end-to-end trust.  This both reduces the barriers to entry for competition in the service provider marketplace and increases overall security.

The business identity integrity and invoice message integrity discussions below apply to the majority use case of lookup and routing based on the ABN as a buisness identifier.  Marginal cases using other identifier schemes including domain names are discussed later on this page.

## Business Identity Integrity

The metadata registries are the heart of the system and user must have confidence in the integrity published content.  That the entry for a specific ABN really represents that ABN and has been authorised by that ABN.

This framework operates on the basis that the issuer of a business identifier (the ABR in the case of the ABN) is best positioned to verify the identity of anyone attempting to authorise an update to the metadata registry.  Also that the best and most ubiquitous protocol to verify identity and authorisation is OIDC (Open ID Connect).  OIDC is the protocol behind the more familiar "sign-in with facebook / google".   But in this case it is "sign in to government with the credential that proves you are an authorised officer of the specified ABN".  

The key OIDC roles are
* The Resource Owner (RO).  This is the business identified by an ABN that is authorising a registry update or sending a signed message. We will use an imaginary business "ACME Pty Ltd" identified by ABN "11223344556" for our example.
* The Relying Party (RP).  These are the the service providers (eg access points and metadata publishers) that need to verify the business identity. We will use an imaginary financial software system "BizAccounts" (bizaccounts.com.au) as the access point and an imaginary entity  "BizRegister" (bizregister.net) as the registry service provider.
* The IDP (Identity provider).  This is the identity authority. In this case the [VANguard](http://vanguard.business.gov.au) trust broker service run by the Department of Industry.

The Prerequisites are
* That ACME is a registered business with the ABR identified as ABN 11223344556.  This is just the normal australian business registration process.  ACME has also either an AUSkey or (if a sole trader) has linked their business to their my.gov.au account.
* That BizAccounts is enrolled with VANguard as an RP and has been issued with a VANguard signed certificate that will identify BizAccounts to VANguard as a registered RP.
* That BizRegister will hold the service metadata for ACME - either becuase they are BizAccount's preferred provider or becuase ACME already has a previous service entry on BizRegister.  Note that, in many cases, the access point and the metadata registry are the same. But since the model does not require this, we will use tow different parties to demonstrate that the security model still works.

The use case for updating BizRegister with an entry BizAccounts as the authorised e-invoicing end point for ACME is described below.  Steps tagged "user" are seen by the user whilst those tagged "system" are backgrounf OIDC interactions.
1. user: ACME is logged in to their service provider at www.bizaccounts.com.au and clicks on an "enable e-invoicing" button.
2. system: BizAccount makes an OIDC callback request to VANguard.
3. user: ACME is redirected to VANguard.gov.au where they authenticate using their my.gov.au (sole traders) or AUSkey (other businesses). VANguard pops-up an authorisation dialog saying "BizAccounts would like to act as you e-invoicing service provider" with "Accept" or "Reject" actions.  ACME clicks "Accept".
4. system: VANguard returns a signed JWT (JSON Web Token) to BizAccount that contains two "claims".  One is the verified ABN "1122334456" and the other is the explicitly authorised service "e-invoicing".
5. system: BizAccounts calls the REST POST bizregister/1122334456/service end point to update ACME's service metadata and provides the signed JWT token as evidence of ACME's authority.  BizAccounts also provides their VANGuard RP certificate which will be published as the "certificate" element against the e-invoicjng end point for ACME.
6. system: BizRegister verifies the JWT token against the VANguard service to confirm that ACME has indeed authorised BizAccounts to act as their e-invoice end point.  If OK, then BizRegister updates the entry for ACME.

it's important to note that, although this might seem complex, 
* The user experience is simple - they just clicked the "enable e-invoicing" button in BizAccounts and proved their identity to VANguard.
* The technology implementation for BizAccounts and BizRegister is simple - it's just the same OIDC protocol that is natively supported by their chosen technology stack becuase it's used every day for the more common "sign in with facebook" case.

## Invoice Message integrity

The recipient of an e-invoice musy have confidence that it was issued by the sender ABN and not some third party masquerading as that ABN.  Also, both parties should have confidence that the potentially sensitve commercial information cannot be seen by malicious third parties.

Unlike updating the metadata registry entry, which is an occasional process, sending and receiving invoices is a much higher volume activity and so it would be impractical to expect users to (for example) authenticate to VANguard for every invoice.  Therefore a different approach is used to ensure trust of identity and message integrity.  The framework uses PKI and the digital certificates published as part of the registry update process.

We will introduce two new parties.
* Widget Pty Ltd is a seller organisation that wants to send an e-invoice to ACME Pty Ltd.
* ProFinancials is the e-invoicing solution and access point provider for Widget Pty Ltd.

The use case for the secure exchange of the e-invoice and the business response documents is:
1. user: Widget Pty Ltd accounts receivable staff create an invoice to ACME Pty Ltd in the normal way using their ProFinancials system.
2. system: ProFinancials looks up the service metadata for ACME pty ltd via the DCL and DCP and confirms that there is a capability match (document standard and message transport).
3. system: Profinancials signs the invoice using their private key (the corresponding public key is published with the certifiate in Widget Pty Ltd service metadata) and performs an SSL REST POST to the end point specified in the ACME service metadata with the signed invoice.json as the body.
4. system: BizAccounts receives the invoice, retreives the seller party ABN (Widget's ABN) from the invoice document, does a DCL/DCP lookup using that ABN to get Widget's service metadata, and verifies the signature using the public key from Widget's service metadata.  This critical step verifies the chain of trust - that the sender of the invoice is the same party that (with VANguard authentication) created the registry entry.
5. system: BizAccounts posts the invoice into ACME Pty Ltd Ledger in accordance with ACME standard invoice processing rules.  Typically, if the invoice is a demand for future payment then it will be matched against an order or contract.  If the invoice is a tax invoice (ie already paid with zero balance) then the invoice will be auto-reconciled against bank statement.
6. user: If a demand for future payment, then ACME Pty Ltd accounts payable staff review / reconcile the invoice and follow their usual internal approval processes.
7. system: Once processed, BizAccounts creates an response document that is returned to the seller party via the same process in reverse (ie BizAccounts will lookup, sign & send - then ProFinancials will verify signature and reconcile against the invoice).






