# Domain Trust Mapping

Get a list of all domain trusts for the current location
`Get-DomainTrust
`Get-DomainTrust -Domain <DOMAIN>`
`Get-ADTrust`
`Get-ADTrust -Identity <user>`


# Forest Mapping
Get all global catalogs for the current forest
`Get-ForestGlobalCatalog`
`Get-ForestGlobalCatalog -Forest <DOMAIN>`
`Get-ADForest | select -ExpandProperty GlobalCatalogs`

Map trusts of a trust (no Forest trusts in the lab)
`Get-ForestTrust`
`Get-ForestTrust -Forest <DOMAIN>`
`Get-ADTrust -Filter 'msDS-TrustForestTrustInfo -ne "$null"'`


# Learning Objectives

Enumerate all domains in the moneycorp.local forest
`Get-ForestDomain -Verbose`

Map the trusts of the dollarcorp.moneycorp.local domain
`Get-DomainTrust` | ?\{$.TrustAttributes -eq "FILTER_SIDS"\}

Map external trusts in moneycorp.local.forest
`Get-ForestDomain -Forest <DOMAIN> | %{Get-DomainTrust -Domain $_.Name}`

Identify external trusts of dollarcorp domain. Can you enumerate trusts for a trusting forest?
![[assets/Pasted image 20260131020851.png|Pasted image 20260131020851.png]]
- you cannot enumerate trusts for a trusting forest (i.e. getting eu.eurocorp.local from dollarcorp) -- not transitive
- can get almost anything else though -> build bloodhound database and extract credentials to then access eurocorp
`Get-DomainUser -Domain eurocorp.local`
![[assets/Pasted image 20260131021129.png|Pasted image 20260131021129.png]]
