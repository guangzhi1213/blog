---
title: "OpenldapCentos6"
date: 2021-04-25T11:13:57+08:00
lastmod: 2021-04-25T11:13:57+08:00
draft: false
keywords: []
description: "manjoc'blog"
tags: []
categories: []
author: "王清"
---

# Setting up OpenLDAP on CentOS 6

These instructions are intended to help first-time LDAP administrators get up and running. The following procedures contain instructions for getting started using OpenLDAP on a CentOS 6 system. For more complete information on how to set up OpenLDAP see the [OpenLDAP documentation](http://www.openldap.org/doc/admin24/).

- [Installing and configuring OpenLDAP on Centos 6](http://docs.adaptivecomputing.com/viewpoint/hpc/Content/topics/1-setup/installSetup/settingUpOpenLDAPOnCentos6.htm#installing)
- [Adding an organizational unit (OU)](http://docs.adaptivecomputing.com/viewpoint/hpc/Content/topics/1-setup/installSetup/settingUpOpenLDAPOnCentos6.htm#addOU)
- [Adding a user](http://docs.adaptivecomputing.com/viewpoint/hpc/Content/topics/1-setup/installSetup/settingUpOpenLDAPOnCentos6.htm#addUser)
- [Adding a group](http://docs.adaptivecomputing.com/viewpoint/hpc/Content/topics/1-setup/installSetup/settingUpOpenLDAPOnCentos6.htm#addGroup)
- [Adding a user to a group](http://docs.adaptivecomputing.com/viewpoint/hpc/Content/topics/1-setup/installSetup/settingUpOpenLDAPOnCentos6.htm#addUserToGroup)

![img](http://docs.adaptivecomputing.com/viewpoint/hpc/Content/Resources/Graphics/templateIcons/note.png)

Adaptive Computing is not responsible for creating, maintaining, or supporting customer LDAP or Active Directory configurations.

Installing and configuring OpenLDAP on Centos 6

First, you will need to install OpenLDAP. These instructions explain how you can do this on a CentOS 6 system.

To install and configure OpenLDAP on Centos 6

1. Run the following command:

2. ```
   [root]# yum -y install openldap openldap-clients openldap-servers
   ```

3. Generate a password hash to be used as the admin password. This password hash will be used when you create the root user for your LDAP installation. For example:

4. ```
   [root]# slappasswd
   New password : p@ssw0rd
   Re-enter new password : p@ssw0rd
   {SSHA}5lPFVw19zeh7LT53hQH69znzj8TuBrLv
   ```

5. Add the root user and the root user's password hash to the OpenLDAP configuration in the

    

   olcDatabase={2}bdb.ldif

    

   file. The root user will have permissions to add other users, groups, organizational units, etc. Do the following:

   1. Run this command:
   2. ```
      [root]# cd /etc/openldap/slapd.d/cn\=config
      [root]# vi olcDatabase\=\{2\}bdb.ldif
      ```

   3. If the **olcRootPW** attribute does not already exist, create it. Then set the value to be the hash you created from slappasswd. For example:
   4. ```
      olcRootPW: {SSHA}5lPFVw19zeh7LT53hQH69znzj8TuBrLv
      ...
      ```

6. While editing this file, change the distinguished name (DN) of the **olcSuffix** to something appropriate. The suffix typically corresponds to your DNS domain name, and it will be appended to the DN of every other LDAP entry in your LDAP tree.

7. For example, let's say your company is called Acme Corporation, and that your domain name is "acme.com." You might make the following changes to the olcDatabase={2}bdb.ldif file:

8. ```
   olcSuffix: dc=acme,dc=com
   ...
   olcRootDN: cn=Manager,dc=acme,dc=com
   ...
   olcRootPW: {SSHA}5lPFVw19zeh7LT53hQH69znzj8TuBrLv
   ...
   ```

9. ![img](http://docs.adaptivecomputing.com/viewpoint/hpc/Content/Resources/Graphics/templateIcons/note.png)

10. Throughout the following examples in this topic, you will see dc=acme,dc=com. "acme" is only used as an example to illustrate what you would use as your own domain controller if your domain name was "acme.com." You should replace any references to "acme" with your own organization's domain name.

11. ![img](http://docs.adaptivecomputing.com/viewpoint/hpc/Content/Resources/Graphics/templateIcons/warning.png)

12. Do not set the cn of your root user to "root" (cn=root,dc=acme,dc=com), or OpenLDAP will have problems.

13. Modify the DN of the root user in the olcDatabase={1}monitor.ldif file to match the **olcRootDN** line in the olcDatabase={2}bdb.ldif file. Do the following:

14. 1. Run this command to edit the olcDatabase={1}monitor.ldif file:
    2. ```
       [root]# vi olcDatabase\=\{1\}monitor.ldif
       ```

    3. Modify the **olcAccess** line so that the **dn.base** matches the **olcRootDN** from the olcDatabase={2}bdb.ldif file. (In this example, **dn.base** should be "cn=Manager,dc=acme,dc=com".)
    4. ```
       olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth" read by dn.base="cn=Manager,dc=acme,dc=com" read by * none
       ```

    5. Now the root user for your LDAP is cn=Manager,dc=acme,dc=com. The root user's password is the password that you entered using slappasswd (see step [2](http://docs.adaptivecomputing.com/viewpoint/hpc/Content/topics/1-setup/installSetup/settingUpOpenLDAPOnCentos6.htm#pw)), which, in this example, is **p@ssw0rd**

15. Hide the password hashes from users who should not have permission to view them.

16. ![img](http://docs.adaptivecomputing.com/viewpoint/hpc/Content/Resources/Graphics/templateIcons/note.png)

17. A full discussion on configuring access control in OpenLDAP is beyond the scope of this tutorial. For help, see [the OpenLDAP Access Control documentation](http://www.openldap.org/doc/admin24/access-control.html).

18. 1. Run this command to edit the

        

       oclDatabase\=\{2\}bdb.ldif

        

       file: 

       ```
       [root]# vi olcDatabase\=\{2\}bdb.ldif
       ```

    2. Add the following two lines to the end of the file to restrict users from viewing other users' password hashes.

    3. olcAccess: {0}to attrs=userPassword by self write by dn.base="cn=Manager,dc=acme,dc=com" write by anonymous auth by * none

    4. olcAccess: {1}to * by dn.base="cn=Manager,dc=acme,dc=com" write by self write by * read

    5. These lines allow a user to read and write his or her own password. It also allows a manager to read and write anyone's password. Anyone, including anonymous users, is allowed to view non-password attributes of other users.

19. Make sure that OpenLDAP is configured to start when the machine starts up, and start the OpenLDAP service.

20. ```
    [root]# chkconfig slapd on
    [root]# service slapd start
    ```

21. Now, you must manually create the "dc=acme,dc=com" LDAP entry in your LDAP tree.

22. An LDAP directory is analogous to a tree. Nodes in this tree are called LDAP "entries" and may represent users, groups, organizational units, domain controllers, or other objects. The attributes in each entry are determined by the LDAP schema. In this tutorial we will build entries based on the InetOrgPerson schema (which ships with OpenLDAP by default).

23. In order to build our LDAP tree we must first create the root entry. Root entries are usually a special type of entry called a domain controller (DC). Because we are assuming that the organization is called Acme Corporation, and that the domain is "acme.com," we will create a domain controller LDAP entry called dc=acme,dc=com. Again, you will need to replace "acme" with your organization's domain name. Also note that dc=acme,dc=com is what is called an LDAP distinguished name (DN). An LDAP distinguished name uniquely identifies an LDAP entry.

24. Do the following:

25. 1. Create a file called acme.ldif. (You can delete this file once its content has been added to LDAP, so in this example, we will create it in the /tmp folder.)
    2. ```
       [root]# cd /tmp
       [root]# vi acme.ldif
       ```

    3. Add the following lines in acme.ldif:
    4. ```
       dn: dc=acme,dc=com
       objectClass: dcObject
       objectClass: organization
       dc: acme
       o : acme
       ```

    5. Now add the contents of this file to LDAP. Run this command:
    6. ```
       [root]# ldapadd -f acme.ldif -D cn=Manager,dc=acme,dc=com -w p@ssw0rd
       ```

    7. Verify that your entry was added correctly.
    8. ```
       [root]# ldapsearch -x -LLL -b dc=acme,dc=com
       dn: dc=acme,dc=com 
       objectClass: dcObject
       objectClass: organization
       dc: acme
       o: acme
       ```

26. 

27. By default, the CentOS 6 firewall will block external requests to OpenLDAP. In order to allow MWS to access LDAP, you will have to configure your firewall to allow connections on port 389. (Port 389 is the default LDAP port.)

28. Configuring your firewall is beyond the scope of this tutorial; however, it may be helpful to know that the default firewall on CentOS is a service called iptables. (For more information, see the documentation on [iptables](http://wiki.centos.org/HowTos/Network/IPTables).) In the most basic case, you may be able to add a rule to your firewall that accepts connections to port 389 by doing the following:

29. 1. Edit your iptables file:
    2. ```
       [root]# vi /etc/sysconfig/iptables
       ```

    3. Add the following line *after* all the **ACCEPT** lines but *before* any of the **REJECT** lines in your iptables file:
    4. ```
       # ... lines with ACCEPT should be above
       -A INPUT -p tcp --dport 389 -j ACCEPT
       # .. lines with REJECT should be below
       ```

    5. For example, here is a sample iptables file with this line added:

    6. ```
       *filter
       :INPUT ACCEPT [0:0]
       :FORWARD ACCEPT [0:0]
       :OUTPUT ACCEPT [0:0]
       -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
       -A INPUT -p icmp -j ACCEPT
       -A INPUT -i lo -j ACCEPT
       -A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT
       -A INPUT -p tcp --dport 389 -j ACCEPT
       -A INPUT -j REJECT --reject-with icmp-host-prohibited
       -A FORWARD -j REJECT --reject-with icmp-host-prohibited
       
       COMMIT
       ```

    7. Now flush iptables.
    8. ```
       [root]# iptables --flush
       ```

30. 

31. ![img](http://docs.adaptivecomputing.com/viewpoint/hpc/Content/Resources/Graphics/templateIcons/note.png)

32. Although providing instructions is beyond the scope of this tutorial, it is also highly recommended that you set up OpenLDAP to use SSL or TLS security to prevent passwords and other sensitive data from being sent in plain text. For information on how to do this, see the [OpenLDAP TLS documentation](http://www.openldap.org/doc/admin24/tls.html).

Now that you have installed and set up Open LDAP, you are ready to add organizational units (see [Adding an organizational unit (OU)](http://docs.adaptivecomputing.com/viewpoint/hpc/Content/topics/1-setup/installSetup/settingUpOpenLDAPOnCentos6.htm#addOU)).

Adding an organizational unit (OU)

These instructions will describe how to populate the LDAP tree with organizational units (OUs), groups, and users, all of which are different types of LDAP entries. The examples that follow also presume an InetOrgPerson schema, because the InetOrgPerson schema is delivered with OpenLDAP by default.

To add an organizational unit (OU) entry to the LDAP tree

In this example, we are going to add an OU called "Users."

1. Create a temporary file called users.ldif. (You can delete this file once its content has been added to LDAP, so in this example, we will create it in the /tmp folder.)
2. ```
   [root]# cd /tmp
   [root]# vi users.ldif
   ```

3. Add these lines to users.ldif:
4. ```
   dn: ou=Users,dc=acme,dc=com
   objectClass: organizationalUnit
   ou: Users
   ```

5. Add the contents of users.ldif file to LDAP.
6. ```
   [root]# ldapadd -f users.ldif -D cn=Manager,dc=acme,dc=com -w p@ssw0rd
   ```

Adding a user

To add a user to LDAP

In this example, we will add a user named "Bob Jones" to LDAP inside the "Users" OU.

1. Create a temporary file called bob.ldif. (You can delete this file once its content has been added to LDAP, so in this example, we will create it in the /tmp folder.)
2. ```
   [root]# cd /tmp
   [root]# vi bob.ldif
   ```

3. Add these lines to bob.ldif:
4. ```
   dn: cn=Bob Jones,ou=Users,dc=acme,dc=com
   cn: Bob Jones
   sn: Jones
   objectClass: inetOrgPerson
   userPassword: p@ssw0rd
   uid: bjones
   ```

5. Add the contents of bob.ldif file to LDAP.
6. ```
   [root]# ldapadd -f bob.ldif -D cn=Manager,dc=acme,dc=com -w p@ssw0rd
   ```

Adding a group

To add a group to LDAP

In this example, we will add a group called "Engineering" to LDAP inside the "Users" OU.

1. Create a temporary file called engineering.ldif. (You can delete this file once its content has been added to LDAP, so in this example, we will create it in the /tmp folder.)
2. ```
   [root]# cd /tmp
   [root]# vi engineering.ldif
   ```

3. Add these lines to engineering.ldif:
4. ```
   dn: cn=Engineering,ou=Users,dc=acme,dc=com
   cn: Engineering
   objectClass: groupOfNames
   member: cn=Bob Jones,ou=Users,dc=acme,dc=com
   ```

5. Add the contents of engineering.ldif file to LDAP.
6. ```
   [root]# ldapadd -f engineering.ldif -D cn=Manager,dc=acme,dc=com -w p@ssw0rd
   ```

Adding a user to a group

To add a user to an LDAP group

In this example, we will add an LDAP member named "Al Smith" to the "Engineering" LDAP group. This example assumes that user, Al Smith, has already been added to LDAP.

![img](http://docs.adaptivecomputing.com/viewpoint/hpc/Content/Resources/Graphics/templateIcons/note.png)

Before you add a user to an LDAP group, the user must first be added to LDAP. For more information, see [Adding a user](http://docs.adaptivecomputing.com/viewpoint/hpc/Content/topics/1-setup/installSetup/settingUpOpenLDAPOnCentos6.htm#addUser).

1. Create a temporary file called addUserToGroup.ldif. (You can delete this file once its content has been added to LDAP, so in this example, we will create it in the /tmp folder.)
2. ```
   [root]# cd /tmp
   [root]# vi addUserToGroup.ldif
   ```

3. Add these lines to addUserToGroup.ldif:
4. ```
   dn: cn=Engineering,ou=Users,dc=acme,dc=com
   changetype: modify
   add: member
   member: cn=Al Smith,ou=Users,dc=acme,dc=com
   ```

5. Now add the contents of addUserToGroup.ldif file to LDAP.
6. ```
   [root]# ldapadd -f addUserToGroup.ldif -D cn=Manager,dc=acme,dc=com -w p@ssw0rd
   ```
