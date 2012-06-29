#SAML
Security Assertion Markup Language (SAML) is an XML-based open standard writen by [OASIS](http://www.oasis-open.org/) that enables the interchange of authn and authz data between secure domains. Enables web-based authentication and authorization scenarios including single sign-on. Following a short description of SAML 2.0 from [SAML technical overview](http://www.oasis-open.org/committees/download.php/27819/sstc-saml-tech-overview-2.0-cd-02.pdf). 
The three main goals of this technology are:

1. Single sign-on. SAML solves the supported multi-domain SSO (MDSSO) problem by providing a standard vendor-independent grammar and protocol for transferring information about a user from one web server to another independent of the server DNS domains
2. Federated identity management. The user is said to have a federated identity when partners have established an agreement on how to refer to the user.
3. Web services and other industry standards. SAML allows for its security assertion format to be used outside of a native SAML-based protocol context.

SAML defines four types of xml-based elements: assertions, profiles, bindings and protocols.

__Assertions__ contain a packet of security information. It may contain three types of statements: Authentication statements, Attribute statements and Authorization statements.

__Protocols__ describe HOW SAML elements are packaged within SAML request and response elements.

__Bindings__ are mappings of a SAML protocol message onto standard messaging formats and/or communication protocols, like SOAP or HTTP.

__Profiles__ describe in detail how SAML assertions,protocols and bindings combine to support a defined use case.
