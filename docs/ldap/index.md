# __:fontawesome-solid-users-gear: LDAP__

LDAP is a protocol for accessing and maintaining distributed directory information services over an Internet Protocol (IP) network. It is used to look up data in a directory and is often used for user and group management, system configuration management, address book functionality, and more. LDAP is typically used in enterprise settings for accessing and maintaining a centralized directory of users and resources.

## Directory Server
It’s a type of network database that stores information represented as trees of entries. 

## Entry
A record in directory server. Consists of three primary components: a distinguished name, a collection of attributes, and a collection of object classes.

### [Directory Information Tree (DIT)](https://ldap.com/dit-and-the-ldap-root-dse)

Key points:

1. Directory Information Tree (DIT): The DIT is a hierarchical organization of entries in an LDAP directory. Each entry is identified by a Distinguished Name (DN), which includes the entry's Relative Distinguished Name (RDN) and the names of its ancestor entries. The DIT starts from the root (or top) entry, also known as the root DSE.
2. Root DSE: The root DSE is a special entry in an LDAP directory that provides information about the directory server. It's not part of the DIT, but it's accessible in every LDAP server. The root DSE has no DN, and it's accessed by binding to the server without specifying a DN.
3. Attributes of the Root DSE: The root DSE contains several operational attributes that provide information about the server, including the server's capabilities, the naming contexts (the DNs of the root entries of the DITs) it maintains, the controls it supports, etc.
4. Accessing the Root DSE: To access the root DSE, you can perform a base object search without specifying a DN. The search should request the specific attributes you're interested in, or it can request all operational attributes.
5. Uses of the Root DSE: The root DSE is often used in LDAP clients to discover information about the server's capabilities, to find the naming contexts it maintains, and to determine the appropriate base DN for searches.


### DNs and RDNs

DN -> distinguished name

RDN -> relative distinguished name

Entry’s DN, uniquely identifies that entry and its position in the directory information tree (DIT) hierarchy. The DN of an LDAP entry is much like the path to a file on a filesystem.

DN is comprised of zero or more RDNs. Each RDN is comprised of one or more (usually just one) attribute-value pairs.
!!! example
    ```{.bash}
    # RDN with single attribute-value pair
    "uid=john.doe" -> comprised of an attribute named "uid" with a value of "john.doe"
    ```

    ```{.bash}
    # RDN with multiple attribute-value pairs
    “givenName=John+sn=Doe”
    “cn=John Doe+mail=jdoe@example.com”
    ```

RDNs with multiple name-value pairs are called multivalued RDNs, and they are primarily used for cases in which it is not possible to guarantee that an RDN with a single component could be unique among entries at a given level of the hierarchy

The order of the RDNs specifies the position of the associated entry in the DIT. RDNs are separated by commas

!!! example
    the DN 
    ```{.bash}
    "uid=john.doe,ou=People,dc=example,dc=com" has four RDNs, with the parent DN being "ou=People,dc=example,dc=com"
    ```


### Base DN
The base DN is the point from where a server will search for entries. It's the starting point for any LDAP search operation. For example, if the base DN is ”ou=Marketing,dc=example,dc=com”, the search will include entries in the "Marketing" organizational unit and its subordinates.


### Common acronyms

1. CN (Common Name): This is an attribute used to hold a name that represents the object. For a person, it could be their full name, like "John Doe". For a printer, it could be the model or location.
2. OU (Organizational Unit): This is used to group objects together. For example, all users in the same department could be grouped under an OU representing that department.
3. DC (Domain Component): This is used to represent the different parts of a domain name. For example, the domain "example.com" would be represented as "dc=example,dc=com".
4. DN (Distinguished Name): This is a sequence of CN, OU, DC, and other components that uniquely identifies an object in the directory and represents its exact location in the tree.
5. RDN (Relative Distinguished Name): This is a component of the DN that distinguishes the entry from its siblings, i.e., other entries that have the same parent.
6. LDIF (LDAP Data Interchange Format): This is a standard plain-text format for representing LDAP directory entries and updates to those entries.
7. DIT (Directory Information Tree): This is the hierarchical tree-like structure of entries in an LDAP directory.


### Attributes

Attributes hold the data for an entry. Each attribute has an attribute type, zero or more attribute options, and a set of values that comprise the actual data.

* Attribute types are schema elements that specify how attributes should be treated by LDAP clients and servers.
* All attribute types must have an object identifier (OID) and zero or more names that can be used to reference attributes of that type.
* They must also have an attribute syntax, which specifies the type of data that can be stored in attributes of that type, and a set of matching rules, which indicate how comparisons should be performed against values of attributes of that type.
* Attribute types may also indicate whether an attribute is allowed to have multiple values in the same entry, and whether the attribute is intended for holding user data (a user attribute) or is used for the operation of the server (an operational attribute). Operational attributes are typically used for configuration and/or state information.

Here are some commonly used LDAP attributes:

1. cn (Common Name): This attribute is used to hold the name of the object. For a person, it could be their full name.
2. sn (Surname): This attribute is used to hold the family name of a person.
3. givenName: This attribute is used to hold the personal or first name of a person.
4. mail: This attribute is used to hold the email address of a person.
5. uid (User ID): This attribute is used to hold the unique identifier of a user.
6. dn (Distinguished Name): This attribute is used to hold the unique path of the entry in the directory.
7. ou (Organizational Unit): This attribute is used to group related entries together.
8. dc (Domain Component): This attribute is used to form the names of domains.
9. sAMAccountName: This attribute is used in Microsoft Active Directory to hold the logon name used to support clients and servers from a previous version of Windows.
10. objectClass: This attribute defines the type of the object, or the schema, that the entry adheres to.


### [Schema](https://ldap.com/understanding-ldap-schema)

Equivalent to relational database schema that contains information about the tables, columns, and the data types and constraints of each of those columns.

An LDAP schema may contain several types of elements. Every schema must include at least the following:

* Attribute Syntaxes This defines the kind of data that the attribute can hold and the rules for encoding and comparing that data. For example, the syntax could define if the data is a string, an integer, a boolean, a date/time, etc. It could also define how to compare the data, like case-sensitive or case-insensitive, numeric or lexicographic order, etc.
* Matching Rules define the kinds of comparisons that can be performed against LDAP data.
* Attribute Types: This defines the nature of the attribute and how it is used. Each attribute type has a unique name and an optional set of aliases. It also has an associated syntax and can have various other properties. For example, the attribute type could define if the attribute is single-valued or multi-valued, if it is readable or writable, etc. Examples of attribute types include "cn" for common name, "mail" for email address, and "ou" for organizational unit.
* Object Classes define named collections of attribute types which may be used in entries containing that class, and which of those attribute types will be required rather than optional. Every entry has a structural object class, which indicates what kind of object an entry represents, and may also have zero or more auxiliary object classes that suggest additional characteristics for that entry.


### Search Filters

Search filters are used to define criteria for identifying entries that contain certain kinds of information. There are a number of different types of search filters:

* Presence filters may be used to identify entries in which a specified attribute has at least one value.
* Equality filters may be used to identify entries in which a specified attribute has a particular value.
* Substring filters may be used to identify entries in which a specified attribute has at least one value that matches a given substring.
* Greater-or-equal filters may be used to identify entries in which a specified attribute has at least one value that is considered greater than or equal to a given value.
* Less-or-equal filters may be used to identify entries in which a specified attribute has at least one value that is considered less than or equal to a given value.
* Approximate match filters may be used to identify entries in which a specified attribute has a value that is approximately equal to a given value. Note that there is no official definition of “approximately equal to”, and therefore this behavior may vary from one server to another. Some servers use a “sounds like” algorithm like one of the Soundex or Metaphone variants.
* Extensible match filters may be used to provide more advanced types of matching, including the use of custom matching rules and/or matching attributes within an entry’s DN.
* AND filters may be used to identify entries that match all of the filters encapsulated inside the AND.
* OR filters may be used to identify entries that match at least one of the filters encapsulated inside the OR.
* NOT filters may be used to negate the result of the encapsulated filter (i.e., if a filter matches an entry, then a NOT filter encapsulating that matching filter will not match the entry, and if a filter does not match an entry, then a NOT filter encapsulating that non-matching filter will match the entry).

```{.bash}
ldapsearch -h esad.es.ad.adp.com -p 389 -D "autoxpert" -w Adpadp10 -b "" -s base "(objectclass=*)" "*" "+"
ldapsearch -h esad.es.ad.adp.com -p 389 -D "autoxpert" -w Adpadp10 -b "DC=ES,DC=AD,DC=ADP,DC=com"

ldapsearch -h esad.es.ad.adp.com -p 389 -D "autoxpert" -w Adpadp10 -b "DC=ES,DC=AD,DC=ADP,DC=com"

ldapsearch -h esad.es.ad.adp.com -p 389 -D "autoxpert" -w Adpadp10 -b "DC=ES,DC=AD,DC=ADP,DC=com" "((cn=John Doe)&(mail=*example.com*))" "*" "+"

ldapsearch -h esad.es.ad.adp.com -p 389 -D "autoxpert" -w Adpadp10 -b "DC=ES,DC=AD,DC=ADP,DC=com" "(sAMAccountName=sebastic)" "*" "+" ORldapsearch -h esad.es.ad.adp.com -p 389 -D "sebastic@es.ad.adp.com" -W -b "DC=ES,DC=AD,DC=ADP,DC=com" "(sAMAccountName=sebastic)" mail aDPDivisionDesc aDPManagerEmail department displayNamePrintable givenName displayName name sAMAccountName userPrincipalName title cn l c st distinguishedName

ldapsearch -h esad.es.ad.adp.com -p 389 -D "autoxpert" -w Adpadp10 -b "DC=ES,DC=AD,DC=ADP,DC=com" "(sAMAccountName=sebastic)" mail aDPDivisionDesc aDPManagerEmail department displayNamePrintable givenName displayName name sAMAccountName userPrincipalName title cn l c st distinguishedName

ldapsearch -h esad.es.ad.adp.com -p 389 -D "autoxpert" -w Adpadp10 -b "OU=Standard,OU=ADPUsers,DC=ES,DC=AD,DC=ADP,DC=com" "(memberOf=CN=masrnddevops,OU=ADPGroups,DC=ES,DC=AD,DC=ADP,DC=com)" name


givenName sn displayName name sAMAccountName userPrincipalName mail title l st c aDPManagerEmail department
```


## References

- List of Directory Servers
- OpenDJ directory server installation
