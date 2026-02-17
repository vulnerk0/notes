# AD Structure

A basic AD user account with no added privileges can enumerate the majority of objects with in an AD network like the below image:
![[Pasted image 20251112145723.png]]

---
## Forests
the forest is at the top and it contains one or more domains, which themselves can have a child or more (subdomains). A forest is a security boundary within which all objects are under administrative control. 

---
## Domains

 A domain is a structure within which contained objects like users,computers and groups are accessible. It has many built-int Organizational Units (OUs), such as `Domain Contollers, Users, Computers`. Below is a simple Domain structure:
 
```vim
INLANEFREIGHT.LOCAL/
├── ADMIN.INLANEFREIGHT.LOCAL
│   ├── GPOs
│   └── OU
│       └── EMPLOYEES
│           ├── COMPUTERS
│           │   └── FILE01
│           ├── GROUPS
│           │   └── HQ Staff
│           └── USERS
│               └── barbara.jones
├── CORP.INLANEFREIGHT.LOCAL
└── DEV.INLANEFREIGHT.LOCAL
```

In this case the `INLANEFREIGHT.LOCAL` is the Domain and has the `CORP,DEV,ADMIN` children or subdomains, the ADMIN subdomain has the OUs `Computers, GROUPS, USERS`

---
## Trust Relations Between Forests

It is common to see trust relations between two forests, this is because creating a trust relation between two forests is often easier than recreating the users and groups in the other forest. In the below image, we see to forests with two domains `inlanefreight.htb` and `frieghtlogistics.htb`. we see the bidirectional arrow that indicates a trust relation between these domains; so a user in `inlanefreight.htb` can access a resource in `freightlogistics.htb` and vice versa. BUT that doesn't mean that a user in a subdomain from one forest can access a resource from a subdomain from another forest, to do this we would need to set up another trust relation.
![[Pasted image 20251112151824.png]]

---
# AD Terminology
Here are some key terms used frequently in Active Directory:
### Object
An object can be defined as ANY resource that is present within an AD environment such as Organizational Units (OUs), printers, users...etc

### Attributes (الصفات)
Every object has a set of attributes that defines some characteristics of that object. Such as a hostname or a DNS name. All attributes in AD have an associated LDAP name used in LDAP queries such as `displayName` for `Full Name` or `given name` for `First Name`.

### Schema
Also known as the `blueprint` of the enterprise network. It defines what types of objects can exist in the AD database and their associated attributes. It lists definitions corresponding to AD objects and holds information about each object. For example; users in AD belong to the `user` class, and computers belong to the `computer` class and each object has it's own information, each computer on the AD database is called an instance of the `computer` class.

### Domain
A domain is a group of object like computers, OUs, users...etc. Domains can operate independently of one another, or can be connected together via trust relations.

### Forest
A forest is the topmost collection of active directory, it's like a real forest that has trees like `example.com` and each tree has branches called domains.

### Tree
A tree is a collection of AD domains that belong to the same root domain. A forest is a collection of Trees, each domain in the tree shares a boundary with the other domains in the same tree. A parent-child relation is formed between a domain and a subdomain. Two trees in the same forest can't share the same name (namespace). All domains in a tree share a standard Global Catalog which contains all information about objects that belong to the tree.

### Container
Container objects hold other objects and have a defined place in the directory subtree hierarchy.

### Leaf
Leaf objects do not contain other objects and are found at the end of the subtree hierarchy.

### Global Unique Identifier (GUID)
Every single object created by AD is assigned a GUID, not only user and group objects. The GUID is stored in the `ObjectGUID` attribute. The best way to search for an object in AD is by querying its `ObjectGUID` value or you can use other identifiers like the SID or SAM account name. the `ObjectGUID` value never changes and remains as is as long as the object exists in the domain.

### Security Principals
Security Principals are anything that the operating system can authenticate, including users, computer accounts and threads/processes that run in the context of a user or computer account. In AD, security principals are domain objects that can manage access to other resources within the domain. The local user accounts and security groups that want to access resources on the local computer use the [Security Accounts Manager](https://en.wikipedia.org/wiki/Security_Account_Manager) `SAM`.

### Security Identifier (SID)
The `SID` is used with security principals or security groups. Every account, group, or process has its own unique SID, which, in an AD environment, is issued by the domain controller `DC` and stored in a secure database. A SID can only be used once, also, if the security principal is deleted, it can never be used in that environment. When the user logs in, the system creates and access token that contains the user's SID, the rights they have been granted, and the SIDs for any groups that the user is a member of.

### Distinguished Names (DN)
A Distinguished Name describes the full path to an object in AD (such as `cn=bjones, ou=IT, ou=Employees, dc=inlanefreight, dc=local`). In this example, the user `bjones` works in the IT department of the company Inlanefreight, and his account is created in an Organizational Unit (OU) that holds accounts for company employees. The Common Name (CN) `bjones` is just one way the user object could be searched for or accessed within the domain.

### Relative Distinguished Name (RDN)
A Relative Distinguished Name is a single component of the Distinguished Name that identifies the object as unique from other objects at the current level in the naming hierarchy. In our example, `bjones` is the Relative Distinguished Name of the object. AD does not allow two objects with the same name under the same parent container, but there can be two objects with the same RDNs that are still unique in the domain because they have different DNs. For example, the object `cn=bjones,dc=dev,dc=inlanefreight,dc=local` would be recognized as different from `cn=bjones,dc=inlanefreight,dc=local`.
![[Pasted image 20251115145833.png]]
### sAMAccountName
The sAMAccountName is the user's logon name. Here it would just be `bjones`. It must be a unique value and 20 or fewer characters.

### userPrincipalName
The userPrincipalName attribute is another way to identify users in AD. This attribute consists of a prefix (the user account name) and a suffix (the domain name) in the format of `bjones@inlanefreight.local`. This attribute is not mandatory. think of like an ssh login

### FSMO Roles
the Flexible Single Master Operation roles give Domain controllers the ability to continue authenticating and granting permissions to users (authentication and authorization). There are five FSMO roles: `Schema Master` and `Domain Naming Master` (one of each per forest), `Relative ID (RID) Master` (one per domain), `Primary Domain Controller (PDC) Emulator` (one per domain), and `Infrastructure Master` (one per domain). All five roles are given to the first DC in the forest root domain. Each time a new domain is added to the forest, only the RID Master, PDC Emulator and Infrastructure Master roles are assigned to the new domain. Sysadmins can transfer these roles as needed meaning they are not hard-coded.

### Global Catalog (GC)
A Global catalog is a domain controller that stores copies of ALL objects in an AD forest, the GC stores a full copy of all objects in the current domain and a partial copy of objects that belong to other domains in the forest. Standard domain controllers hold a complete replica of objects belonging to its domain only. The GC allows both users and application to find information about any objects in ANY domain IN the forest. GC is a feature that is enabled on a domain controller and it performs these operations:

- Authentication (provided authorization for all groups that a user account belongs to, which is included when an access token is generated)

- Object search (making the directory structure within a forest transparent, allowing a search to be carried out across all domains in a forest by providing just one attribute about an object.)

### Read-Only Domain Controller (RODC)
A Read-Only Domain Controller has a read-only Active Directory database. No AD account passwords are cached on RODC other than the RODC computer account & RODC KRBTGT passwords. it includes a read-only DNS server and it allows for administrator role separation, reduce replication traffic in the environment.

### Replication
Replication happens in AD when AD objects are updated and/or transferred from one DC to another. Replication ensures that changes are synchronized with all other DCs in the forest, that helps in creating a backup in case one DC fails.

### Service Principal Name (SPN)
A Service Principal Name uniquely identifies a service instance. It is used by Kerberos authentication to associate an instance of a service with a logon account. allowing a client application to request the service to authenticate an account without needing to know the account name.

### Group Policy Object (GPO)
Group Policy Objects are virtual collections of policy settings. Each GPO has a unique GUID. A GPO can contain local file system settings or AD settings. These settings can apply to both user and computer objects or defined more granularly at the OU level.

### Access Control List (ACL)
An Access Control List **i**s the ordered collection of Access Control Entries (ACEs) that apply to an object.

### Access Control Entries (ACEs)
Each ACEin an ACL identifies a trustee (user account, group account, or logon session) and lists the access rights that are allowed, denied, or audited for the given trustee.

### Discretionary Access Control List (DACL)
DACLs define which security principals are granted or denied access to an object; it contains a list of ACEs. If an object doesn't have a DACL then the system will grant full access to everyone to that object, but if the DACL exists and doesn't have any ACE entries then the system will deny access to everyone. ACEs in the DACL are checked in sequence until a match is found that allows the requested rights or until access is denied.

### System Access Control Lists (SACL)
Allows for administrators to log access attempts that are made to secured objects. ACEs specify the types of access attempts that cause the system to generate a record in the security event log.

### Fully Qualified Domain Name (FQDN)
An FQDN is the complete name for a computer in the format of [hostname].[domain name].[tld]. This is used to specify an object's location in the tree hierarchy of DNS. You can use the FQDN instead of the machine's IP like when browsing the web. the computer `DC01` under the domain `ABUMALIK.LOCAL` has the FQDN of `DC01.ABUMALIK.LOCAL`.

### Tombstone
A tombstone is a container object in AD that holds deleted AD objects. When an object is deleted from AD it remains for a period of time called the `Tombstone Lifetime` and the `isDeleted` attribute is set to `TRUE`. Once an object exceeds the `Tombstone Lifetime` it will be permanently removed. This happens when an object is deleted from a domain that doesn't have an AD Recycle Bin. An object in the `Tombstone Lifetime` is stripped of most of its attributes and when recovered, these attributes won't recover with it. 

### AD Recycle Bin
The AD Recycle Bin works similarly to the Tombstone in that the deleted objects stay for a period of time set by the sysadmin and the default is 60d. The big difference is that the recycle bin actually preserve the object's attributes, so objects restored from the bin have their full attributes with them.

### SYSVOL
The SYSVOL folder, or share, stores copies of public files in the domain such as system policies, Group Policy settings, logon/logoff scripts. The contents of the SYSVOL folder are replicated to all DCs within the environment using File Replication Services (FRS).

### AdminSDHolder
This object is used to manage ACLs for members of built-in groups in AD marked as privileged. It acts as a container that holds the Security Descriptor applied to members of protected groups. The SDProp process has a schedule of running every hour by default. When this process runs, it checks members of protected groups to ensure that the correct ACL is applied to them.

### dsHeuristics
The dsHeuristics attribute is a string value set on the Directory Service object used to define multiple forest-wide configuration settings. One of these settings is to exclude built-in groups from the Protected Groups list. Groups in this list are protected from modification via the `AdminSDHolder` object. If a group is excluded via the `dsHeuristics` attribute, then any changes that affect it will not be reverted when the SDProp process runs.

### adminCount
The adminCount attribute determines whether or not the SDProp process protects a user. If the value is set to `0` or not specified, the user is not protected. If the attribute value is set to `1`, the user is protected. Attackers will often look for accounts with the `adminCount` attribute set to `1` to target in an internal environment.

### sIDHistory
The sIDHistory attribute holds any SIDs that an object was assigned previously. It is usually used in migrations so a user can maintain the same level of access when migrated from one domain to another. This attribute can potentially be abused if set insecurely, allowing an attacker to gain prior elevated access that an account had before a migration if SID Filtering (or removing SIDs from another domain from a user's access token that could be used for elevated access) is not enabled.

### NTDS.DIT
The NTDS.DIT file can be considered the heart of Active Directory. It is stored on a Domain Controller at `C:\Windows\NTDS\` and is a database that stores AD data such as information about user and group objects, group membership, and, most important to attackers and penetration testers, the password hashes for all users in the domain. If the setting [Store password with reversible encryption](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/store-passwords-using-reversible-encryption) is enabled, then the NTDS.DIT will also store the cleartext passwords for all users created or who changed their password after this policy was set.

---
# AD Objects
An object can be defined as "`Any resource present within an Active Directory environment`".
![[Pasted image 20251115184819.png]]

### Users
These are the users within the organization's AD environment. Users are considered `leaf objects`, which means that they cannot contain any other objects within them. Another example of a leaf object is a mailbox in Microsoft Exchange. A user object is considered a security principal and has a security identifier (SID) and a global unique identifier (GUID). User objects have many possible [attributes](http://www.kouti.com/tables/userattributes.htm), such as their display name, last login time, date of last password change, email address, account description, manager, address, and more. Depending on how a particular Active Directory environment is set up, there can be over 800 possible user attributes when accounting for ALL possible attributes as detailed [here](https://www.easy365manager.com/how-to-get-all-active-directory-user-object-attributes/).

### Contacts
A contact object is usually used to represent an external user and contains informational attributes such as first name, last name, email address, telephone number, etc. They are `leaf objects` and are NOT security principals (securable objects), so they don't have a SID, only a GUID. An example would be a contact card for a third-party vendor or a customer.

### Printers
A printer object points to a printer accessible within the AD network. Like a contact, a printer is a `leaf object` and not a security principal, so it only has a GUID. Printers have attributes such as the printer's name, driver information, port number, etc

### Computers
A computer object is any computer joined to the AD network (workstation or server). Computers are `leaf objects` because they do not contain other objects. However, they are considered security principals and have a SID and a GUID. Like users, they are prime targets for attackers since full administrative access to a computer (as the all-powerful `NT AUTHORITY\SYSTEM` account) grants similar rights to a standard domain user and can be used to perform the majority of the enumeration tasks that a user account can (save for a few exceptions across domain trusts.)

### Shared Folders
A shared folder object points to a shared folder on the specific computer where the folder resides. Shared folders can have stringent access control applied to them and can be either accessible to everyone (even those without a valid AD account), open to only authenticated users (which means anyone with even the lowest privileged user account OR a computer account (`NT AUTHORITY\SYSTEM`) could access it), or be locked down to only allow certain users/groups access. Anyone not explicitly allowed access will be denied from listing or reading its contents. Shared folders are NOT security principals and only have a GUID. A shared folder's attributes can include the name, location on the system, security access rights.

### Groups
A group is considered a `container object` because it can contain other objects, including users, computers, and even other groups. A group IS regarded as a security principal and has a SID and a GUID. In AD, groups are a way to manage user permissions and access to other securable objects (both users and computers). Let's say we want to give 20 help desk users access to the Remote Management Users group on a jump host. Instead of adding the users one by one, we could add the group, and the users would inherit the intended permissions via their membership in the group. In Active Directory, we commonly see what are called "[nested groups](https://docs.microsoft.com/en-us/windows/win32/ad/nesting-a-group-in-another-group)" (a group added as a member of another group), which can lead to a user(s) obtaining unintended rights. Groups in AD can have many [attributes](http://www.selfadsi.org/group-attributes.htm), the most common being the name, description, membership, and other groups that the group belongs to.

### Organizational Units (OUs)
An organizational unit, or OU from here on out, is a container that systems administrators can use to store similar objects for ease of administration. OUs are often used for administrative delegation of tasks without granting a user account full administrative rights. For example, we may have a top-level OU called Employees and then child OUs under it for the various departments such as Marketing, HR, Finance, Help Desk, etc. If an account were given the right to reset passwords over the top-level OU, this user would have the right to reset passwords for all users in the company. However, if the OU structure were such that specific departments were child OUs of the Help Desk OU, then any user placed in the Help Desk OU would have this right delegated to them if granted.

### Domain
A domain is the structure of an AD network. Domains contain objects such as users and computers, which are organized into container objects: groups and OUs. Every domain has its own separate database and sets of policies that can be applied to any and all objects within the domain. Some policies are set by default (and can be tweaked), such as the domain password policy.

### Domain Controllers
Domain Controllers are essentially the brains of an AD network. They handle authentication requests, verify users on the network, and control who can access the various resources in the domain. All access requests are validated via the domain controller and privileged access requests are based on predetermined roles assigned to users. It also enforces security policies and stores information about every other object in the domain.

### Sites
A site in AD is a set of computers across one or more subnets connected using high-speed links. They are used to make replication across domain controllers run efficiently.

### Built-in
This is a container that holds the [default groups](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups) in an AD domain.

### Foreign Security Principals
A foreign security principal (FSP) is an object created in AD to represent a security principal that belongs to a trusted external forest. They are created when an object such as a user, group, or computer from an external (outside of the current) forest is added to a group in the current domain. They are created automatically after adding a security principal to a group. Every foreign security principal is a placeholder object that holds the SID of the foreign object (an object that belongs to another forest.) Windows uses this SID to resolve the object's name via the trust relationship. FSPs are created in a specific container named ForeignSecurityPrincipals with a distinguished name

# AD Functionality
As mentioned before, there are five Flexible Single Master Operation (FSMO) roles. These roles can be defined as follows:
![[Pasted image 20251116130235.png]]
### Trusts
A trust is used to establish `forest-forest` or `domain-domain` authentication, allowing users to access resources in (or administer) another domain outside of the domain their account resides in. A trust creates a link between the authentication systems of two domains.
![[Pasted image 20251116162839.png]]
![[Pasted image 20251116162918.png]]
Trusts can be transitive or non-transitive.

- A transitive trust means that trust is extended to objects that the child domain trusts.
    
- In a non-transitive trust, only the child domain itself is trusted.
    

Trusts can be set up to be one-way or two-way (bidirectional).

- In bidirectional trusts, users from both trusting domains can access resources.
 
- In a one-way trust, only users in a trusted domain can access resources in a trusting domain, not vice-versa. The direction of trust is opposite to the direction of access.

