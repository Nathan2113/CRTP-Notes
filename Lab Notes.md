# SID of the member of the Enterprise Admins group

the Enterprise Admin is only in the root domain, so checking the domain that student is in won't work
`(Get-ADForest).RootDomain`

get all users in the enterprise admins group on the root domain
`Get-ADGroupMember -Identity "Enterprise Admins" -Server "root-dc.corp.local" | Select-Object Name, SID`


# ActiveDirectory Rights for RDPUsers group on the users named ControlxUser




# Display name of the GPO applied on StudentMachines OU



# Trust Direction for the trust between dollarcorp.moneycorp.local and eurocorp.local



# Service abused on the student VM for local privilege escalation



# Script used for hunting for admin privileges using PowerShell Remoting



# Jenkins user used to access Jenkins web console



# Domain user used for running Jenkins service on dcorp-ci



# Name of the Group Policy attribute that is modified



# Process using svcadmin as service account



# NTLM hash of svcadmin account



# We tried to extract clear-text credentials for scheduled tasks from? Flag value is like lsass, registry, credential vault etc.



# # NTLM hash of srvadmin extracted from dcorp-adminsrv



# NTLM hash of websvc extracted from dcorp-adminsrv



# NTLM hash of appadmin extracted from dcorp-adminsrv



# NTLM hash of krbtgt



# NTLM hash of domain administrator - Administrator



# The service whose Silver Ticket can be used for winrs or PowerShell Remoting



# Name of the account whose secrets are used for the Diamond Ticket attack



# # Name of the Registry key modified to change Logon behavior of DSRM administrator



# Attack that can be executed with Replication rights (no DA privileges required)



# SDDL string that provides studentx same permissions as BA on root\cimv2 WMI namespace. Flag value is the permissions string from (A;CI;Permissions String;;;SID)



# SPN for which a TGS is requested



# Domain user who is a local admin on dcorp-appsrv



# Which user's credentials are compromised by using the printer bug for compromising dollarcorp



# Value of msds-allowedtodelegate to attribute of dcorp-adminsrv



# Alternate service accessed on dcorp-dc by abusing Constrained delegation on dcorp-adminsrv



# Computer account on which ciadmin can configure Resource-based Constrained Delegation



# SID history injected to escalate to Enterprise Admins



# NTLM hash of krbtgt of moneycorp.local



# Service for which a TGS is requested from eurocorp-dc



# Contents of secret.txt on eurocorp-dc



# Name of the AD CS template that has ENROLLEE_SUPPLIES_SUBJECT



# Name of the AD CS template that has EKU of Certificate Request Agent and grants enrollment rights to Domain Users



# Name of the CA attribute that allows requestor to provide Subject Alternative Names



# Name of the group that has enrollment rights on the CA-Integration template



# First SQL Server linked to dcorp-mssql



# Name of SQL Server user used to establish link between dcorp-sql1 and dcorp-mgmt



# Name of SQL Server user used to establish link between dcorp-sql1 and dcorp-mgmt



# Privileges on operating system of eu-sql

