# Service Metadata Publishing and Lookup

The API specification for registry updates and queries is published SwaggerHub as [SMP Specification](https://swaggerhub.com/api/ausdigital/smp/0.1)

## Service Metadata Publishing

The metadata registries are the heart of the system and users must have confidence in the integrity of published content. 

This framework operates on the basis that the issuer of a business identifier (the ABR in the case of the ABN) is best positioned to verify the identity of anyone attempting to authorise an update to the metadata registry.  Also that the best and most ubiquitous protocol to verify identity and authorisation is OIDC (Open ID Connect).  OIDC is the protocol behind the more familiar "sign-in with facebook / google".   But in this case it is "sign in to government with the credential that proves you are an authorised officer of the specified ABN".  

The key OIDC roles are
* The Resource Owner (RO).  This is the business identified by an ABN that is authorising a registry update or sending a signed message. We will use an imaginary business "ACME Pty Ltd" identified by ABN "11223344556" for our example.
* The Relying Party (RP).  These are the the service providers (eg access points and metadata publishers) that need to verify the business identity. We will use an imaginary financial software system "BizAccounts" (bizaccounts.com.au) as the access point and an imaginary entity  "BizRegister" (bizregister.net) as the registry service provider.
* The IDP (Identity provider).  This is the identity authority. In this case the [VANguard](http://vanguard.business.gov.au) trust broker service run by the Department of Industry.

The Prerequisites are
* That ACME is a registered business with the ABR identified as ABN 11223344556.  This is just the normal australian business registration process.  ACME has also either an AUSkey or (if a sole trader) has linked their business to their my.gov.au account.
* That BizAccounts is enrolled with VANguard as an RP and has been issued with a VANguard signed certificate that will identify BizAccounts to VANguard as a registered RP.
* That BizRegister will hold the service metadata for ACME - either becuase they are BizAccount's preferred provider or becuase ACME already has a previous service entry on BizRegister.  Note that, in many cases, the access point and the metadata registry are the same. But since the model does not require this, we will use tow different parties to demonstrate that the security model still works.

### How it works

![Service Metadata Publishing](ServiceMetadataPublish.png)

The use case for updating BizRegister with an entry BizAccounts as the authorised e-invoicing end point for ACME is described below.  Steps tagged "user" are seen by the user whilst those tagged "system" are background OIDC interactions.
* user: ACME is logged in to their service provider at www.bizaccounts.com.au and clicks on an "enable e-invoicing" button.
* system: BizAccount makes an OIDC callback request to VANguard.
* user: ACME is redirected to VANguard.gov.au where they authenticate using their my.gov.au (sole traders) or AUSkey (other businesses). VANguard pops-up an authorisation dialog saying "BizAccounts would like to act as you e-invoicing service provider" with "Accept" or "Reject" actions.  ACME clicks "Accept".
* system: VANguard returns a signed JWT (JSON Web Token) to BizAccount that contains two "claims".  One is the verified ABN "1122334456" and the other is the explicitly authorised service "e-invoicing".
* system: BizAccounts calls the REST POST bizregister/1122334456/service end point to update ACME's service metadata and provides the signed JWT token as evidence of ACME's authority.  BizAccounts also provides their VANGuard RP certificate which will be published as the "certificate" element against the e-invoicjng end point for ACME.
* system: BizRegister verifies the JWT token against the VANguard service to confirm that ACME has indeed authorised BizAccounts to act as their e-invoice end point.  If OK, then BizRegister updates the entry for ACME.

it's important to note that, although this might seem complex, 
* The user experience is simple - they just clicked the "enable e-invoicing" button in BizAccounts and proved their identity to VANguard.
* The technology implementation for BizAccounts and BizRegister is simple - it's just the same OIDC protocol that is natively supported by their chosen technology stack becuase it's used every day for the more common "sign in with facebook" case.

## Service Metadata Lookup

There are two steps to the lookup.  
* Digital Capability Locator (DCL) : First a DNS lookup to find the correct service metadata entry, 
* Digital Capbility Publisher (DCP) : then follow the URL returned by the DCL to get the full metadata record from the DCP.

### Simplified Digital Capability Lookup (DCL)

The lookup assumes that each registrar will also host a DCL and that the result is a direct link to the perticipant's service metadata collection.
* NAPTR record lookup on 3601120601.abr.gov.au returns http://smp.com/{GUID}
* Do a GET on that URL and get back service metadata collection for ABN 3601120601

The URL structure of the NAPTR response is not specified - just so long as it's a URL that points directly the corresponding business service metadata entry.  Using a GUID as the key would allow a metadata publisher to use the same entry point for multiple business identifier schemes (ie whether i do a DNS NAPTR record lookup using an ABN or a DUNS or the DNS domain of the business, I would get back the same URL to the detailed service metadata).

### Simplified Digital Service Publisher Lookup (DCP)

The DCP entryproint from the DCL directly returns a collection of one or more ServiceMetaData entries. 

So GET http://smp.com/{GUID} will just directly return the structure shown below.  
So will GET http://smp.com?abn=23601120601

### Sample Service Metadata Response

The sample below shows a service metadata record example for a business with ABN 23601120601 that supports both standard invoice and RCTI processes.

```
    {"ServiceMetadata":[{
      "ParticipantIdentifier": {"scheme": "abn", "value": "23601120601"},
      "DocumentIdentifier": {"scheme": "dbc-docid", "value": "invoice-1"},
      "ProcessList": [{
        "ProcessIdentifier": {"scheme": "dbc-procid","value": "invoice-1"},
        "ServiceEndpointList": [
          {
          "transportProfile": "REST-POST",
          "EndpointURI": "https://api.myob.com/au/essentials/businesses/23601120601/purchase/invoice-1",
          "RequireBusinessLevelSignature": "true",
          "MinimumAuthenticationLevel": "2",
          "ServiceActivationDate": "2015-05-01",
          "ServiceExpirationDate": "2018-05-01",
          "Certificate": "TlRMTVNTUAABAAAAt7IY4gk....",
          "ServiceDescription": "invoice service",
          "TechnicalInformationUrl": "http://developer.myob.com/api/essentials-accounting/endpoints/"}
          {
        "ProcessIdentifier": {"scheme": "dbc-procid","value": "rcti-1"},
        "ServiceEndpointList": [
          {
          "transportProfile": "REST-POST",
          "EndpointURI": "https://api.myob.com/au/essentials/businesses/23601120601/sales/invoice-1",
          "RequireBusinessLevelSignature": "true",
          "MinimumAuthenticationLevel": "2",
          "ServiceActivationDate": "2015-05-01",
          "ServiceExpirationDate": "2018-05-01",
          "Certificate": "TlRMTVNTUAABAAAAt7IY4gk....",
          "ServiceDescription": "invoice service",
          "TechnicalInformationUrl": "http://developer.myob.com/api/essentials-accounting/endpoints/"}]
        }]
      },
    "Signature": "AD56YGNN8876TTFJ123…"
    },
    {... another signed service metadata structure ...
    }]
    
```

