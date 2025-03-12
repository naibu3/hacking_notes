![[Pasted image 20241119102714.png]]

# Components

- **Object** - An object can be defined as ANY resource present within an Active Directory environment such as OUs, printers, users, domain controllers, etc.

	- **Attributes** - Every object in Active Directory has an associated set of [attributes](https://docs.microsoft.com/en-us/windows/win32/adschema/attributes-all) used to define characteristics of the given object. A computer object contains attributes such as the hostname and DNS name. All attributes in AD have an associated LDAP name that can be used when performing LDAP queries, such as `displayName` for `Full Name` and `given name` for `First Name`.

		- **Global Unique Identifier (GUID)** - A GUID is a unique 128-bit identifier assigned to every AD object. It ensures object uniqueness across the domain and is stored in the `ObjectGUID` attribute.
		- **Security Identifier (SID)** - A SID uniquely identifies a security principal or group in AD. Each SID is unique and never reused, even if the associated object is deleted.
		- **Distinguished Name (DN)** - A DN describes the full path to an object in AD, identifying its location in the directory hierarchy.
		- **Relative Distinguished Name (RDN)** - An RDN is a unique part of the DN that identifies an object at its level in the directory.

- **Schema** - The Active Directory [schema](https://docs.microsoft.com/en-us/windows/win32/ad/schema) is essentially the blueprint of any enterprise environment. It defines what types of objects can exist in the AD database and their associated attributes. It lists definitions corresponding to AD objects and holds information about each object. For example, users in AD belong to the class "user," and computer objects to "computer," and so on. Each object has its own information (some required to be set and others optional) that are stored in Attributes. When an object is created from a class, this is called instantiation, and an object created from a specific class is called an instance of that class. For example, if we take the computer RDS01. This computer object is an instance of the "computer" class in Active Directory.

	- **NTDS.DIT** - The AD database storing all domain objects and password hashes, located at `C:\Windows\NTDS`.

- **Domain** - A domain is a logical group of objects such as computers, users, OUs, groups, etc. We can think of each domain as a different city within a state or country. Domains can operate entirely independently of one another or be connected via trust relationships.

	- **Fully Qualified Domain Name (FQDN)** - An FQDN specifies an object's full name in DNS, such as `host.domain.tld`.
	- **Global Catalog (GC)** - The GC is a domain controller containing full domain object copies and partial objects from other domains in the forest. It supports authentication and cross-domain searches.
	- **Read-Only Domain Controller (RODC)** - An RODC contains a read-only copy of the AD database and DNS. It reduces replication traffic and prevents local changes from affecting the domain.

- **Forest** - A forest is a collection of Active Directory domains. It is the topmost container and contains all of the AD objects introduced below, including but not limited to domains, users, groups, computers, and Group Policy objects. A forest can contain one or multiple domains and be thought of as a state in the US or a country within the EU. Each forest operates independently but may have various trust relationships with other forests.

	- **dsHeuristics** - This attribute configures forest-wide settings, including excluding certain groups from protection.
-  **Tree** - A tree is a collection of Active Directory domains that begins at a single root domain. A forest is a collection of AD trees. Each domain in a tree shares a boundary with the other domains. A parent-child trust relationship is formed when a domain is added under another domain in a tree. Two trees in the same forest cannot share a name (namespace). Let's say we have two trees in an AD forest: `inlanefreight.local` and `ilfreight.local`. A child domain of the first would be `corp.inlanefreight.local` while a child domain of the second could be `corp.ilfreight.local`. All domains in a tree share a standard Global Catalog which contains all information about objects that belong to the tree.

- **Container** - Container objects hold other objects and have a defined place in the directory subtree hierarchy.
- **Leaf** - Leaf objects do not contain other objects and are found at the end of the subtree hierarchy.

- **Security Principals** - Security principals include any entity that the OS can authenticate (e.g., users, computer accounts, processes). They control access to resources in a domain.

	- **sAMAccountName** - A unique logon name for a user, limited to 20 characters.
	- **userPrincipalName** - An alternate user identifier in the format `username@domain`. It is not mandatory.
	- **FSMO Roles** - AD uses Flexible Single Master Operation (FSMO) roles to manage changes and replication. There are five roles: Schema Master, Domain Naming Master, RID Master, PDC Emulator, and Infrastructure Master.
	- **Service Principal Name (SPN)** - SPNs uniquely identify service instances, enabling Kerberos authentication for services.

- **Group Policy Object (GPO)** - GPOs define policy settings applied to user and computer objects. They can be scoped to domains or specific OUs.
- **Access Control List (ACL)** - An ACL is a collection of Access Control Entries (ACEs) that define access rights to an object.

	- **Access Control Entries (ACEs)** - Each ACE specifies access permissions for a trustee (user, group, or session).
	- **Discretionary Access Control List (DACL)** - A DACL specifies who can or cannot access an object. If missing, full access is granted; if empty, access is denied.
	- **System Access Control List (SACL)** - A SACL logs access attempts on objects, recording events in the security log.
	- **AdminSDHolder** - A container managing ACLs for privileged AD groups. The SDProp process ensures correct ACLs for protected groups.
	- **adminCount** - The `adminCount` attribute identifies if a user is protected by the SDProp process. Attackers may target accounts with `adminCount=1`.

- **Tombstone** - A tombstone is a placeholder for deleted objects, stored for a configurable lifetime (default: 60-180 days).
- **AD Recycle Bin** - Introduced in Windows Server 2008 R2, the AD Recycle Bin allows restoration of deleted objects while retaining most attributes.

- **SYSVOL** - The SYSVOL folder stores shared files, including policies and scripts, replicated across domain controllers.

# Structure

**Forests**, **domains** and **organization units** (*OUs*) are the basic building blocks of any active directory structure.

![[Pasted image 20241119103022.png]]
> A forest - which is a security boundary - may contain multiple domains and each domain may contain multiple OUs.

# Trusts

A trust is used to establish `forest-forest` or `domain-domain` authentication, allowing users to access resources in (or administer) another domain outside of the domain their account resides in. A trust creates a link between the authentication systems of two domains.

There are several trust types.

| **Trust Type** | **Description**                                                                                                                                                        |
| -------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Parent-child` | Domains within the same forest. The child domain has a two-way transitive trust with the parent domain.                                                                |
| `Cross-link`   | A trust between child domains to speed up authentication.                                                                                                              |
| `External`     | A non-transitive trust between two separate domains in separate forests which are not already joined by a forest trust. This type of trust utilizes SID filtering.     |
| `Tree-root`    | a two-way transitive trust between a forest root domain and a new tree root domain. They are created by design when you set up a new tree root domain within a forest. |
| `Forest`       | a transitive trust between two forest root domains.                                                                                                                    |
![[trust.png]]

Trusts can be **transitive** or **non-transitive**.

- A transitive trust means that trust is extended to objects that the child domain trusts.
- In a non-transitive trust, only the child domain itself is trusted.

Trusts can be set up to be **one-way** or **two-way** (bidirectional).

- In bidirectional trusts, users from both trusting domains can access resources.
- In a one-way trust, only users in a trusted domain can access resources in a trusting domain, not vice-versa. The direction of trust is opposite to the direction of access.

