---
layout: post
title:  "Pentaho BI integration with Active Directory"
date:   2017-07-31 14:07:00 +0200
categories: pentaho
---
This is a quick guide of how to integrate the Pentaho BI platform with Active Directory. It has been tested on version 5 and 6, it's not yet tested on version 7.

The first file that has to be modified is `biserver-ce\security.properties` where we have to specify that we want to use LDAP authentication:

````
provider=ldap
````

Then we have to modifiy the file `biserver-ce\pentaho-solutions\system\applicationContext-security-ldap.properties` which has to be updated with the actual
Active Directory parameters.

- our domain name is `example.com` and our domain controllers will answer with `example.com`
- our user with read permissions on the AD schema is `browsingad@example.com`
- we have to create a OU `Example Users\Groups\Pentaho Roles` where we will create our groups (for example `Pentaho Administrators`, `Pentaho Read Only`, `Pentaho Dashboards` etc.) with a proper **description**


This is how the file will look like:

````
contextSource.providerUrl=ldap\://example.com\:389/DC\=example,DC\=com
contextSource.userDn=CN\=browsingad,OU\=Example Users,DC\=example,DC\=com
contextSource.password=password

userSearch.searchBase=OU=Example Users
userSearch.searchFilter=(samaccountname\={0})

populator.convertToUpperCase=false
populator.groupRoleAttribute=description
populator.groupSearchBase=OU\=Pentaho Roles,OU\=Groups,OU\=Example Users
populator.groupSearchFilter=(member\={0})
populator.rolePrefix=
populator.searchSubtree=true

allAuthoritiesSearch.roleAttribute=description
allAuthoritiesSearch.searchBase=OU\=Pentaho Roles,OU\=Groups,OU\=Example Users
allAuthoritiesSearch.searchFilter=(objectClass\=group)

allUsernamesSearch.usernameAttribute=samaccountname
allUsernamesSearch.searchBase=OU\=Example Users
allUsernamesSearch.searchFilter=(objectClass\=Person)

adminRole=cn\=Pentaho Administrators,OU\=Pentaho Roles,OU\=Groups,OU\=Example Users
adminUser=cn\=admin.user,OU\=IT Staff,OU\=Example Users
````

Finally we can modify the file `\biserver-ce\pentaho-solutions\system\pentaho.xml` to hide test users:

````html
<login-show-users-list>false</login-show-users-list> 
<login-show-sample-users-hint>false</login-show-sample-users-hint>
````

and also our logo:

````
biserver-ce/pentaho-solutions/system/common-ui/resources/themes/images/puc-login-logo.png
````