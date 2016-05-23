## Publishing Registry Entries

To publish a registry entry for a specific business;
* Business user chooses (via their financial software system) to participate in e-invoicing.
* The financial software system checks with the business already has an entry in the register. If so then the software must update the existing entry rather then create a new one.
* The financial software system redirects the user to an approved OIDC identity provider (we show the Australian government VANGuard IDP in this case) where the the business user will verify their identity and authorise the registry entry publishing action.
* The financial software creates a keypair and signing certificate for the business.  Using the JWT token from the OIDC provider, the financial software creates / updates the business registry.  

![Service Metadata Publishing](ServiceMetadataPublish.png)
