When interacting with Active Directory you'll come across a set of protocols like [Lightweight Directory Access Protocol (LDAP)](https://en.wikipedia.org/wiki/Lightweight_Directory_Access_Protocol), Microsoft's version of [Kerberos](https://en.wikipedia.org/wiki/Kerberos_\(protocol\)), [DNS](https://en.wikipedia.org/wiki/Domain_Name_System) for authentication and communication, and [MSRPC](https://ldapwiki.com/wiki/MSRPC) which is the Microsoft implementation of [Remote Procedure Call (RPC)](https://en.wikipedia.org/wiki/Remote_procedure_call). This is an overview of said protocols.

### Kerberos
#### Authentication Process
![[Pasted image 20251117194030.png]]
![[Pasted image 20251117194044.png]]
>[!note]
>The Kerberos protocol uses port 88 (both TCP and UDP). When enumerating an Active Directory environment, we can often locate Domain Controllers by performing port scans looking for open port 88 using a tool such as Nmap.

---
### LDAP
Ports: LDAP TCP/389 , LDAPS TCP/636

The Lightweight Directory Access Protocol is the protocol used by applications to communicate with directory servers like AD. You can think of it like Apache and HTTP, applications use HTTP to talk to Apache. An LDAP session begins by first connecting to an LDAP server, also known as a Directory System Agent. The Domain Controller in AD actively listens for LDAP requests, such as security authentication requests.
![[Pasted image 20251117195901.png]]
#### AD LDAP Authentication
LDAP is set up to authenticate using a `BIND` operation to set the authentication state for an LDAP session. There are two types of LDAP authentication:

- `Simple Authentication`: This includes anonymous authentication, unauthenticated authentication, and username/password authentication. Simple authentication means that a `username` and `password` create a BIND request to authenticate to the LDAP server.

- `SASL Authentication`: The Simple Authentication and Security Layer (SASL) framework uses other authentication services, such as Kerberos, to bind to the LDAP server and then uses this authentication service (Kerberos in this example) to authenticate to LDAP. The LDAP server uses the LDAP protocol to send an LDAP message to the authorization service, which initiates a series of challenge/response messages resulting in either successful or unsuccessful authentication. SASL can provide additional security due to the separation of authentication methods from application protocols.

>[!note]
>LDAP messages are sent in clear text, so anyone can sniff out the LDAP messages on a network. UNLESS an encryption is setup.
