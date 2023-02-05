---
layout: sublayout
belongsto: projects
title: Okta to M365 Immutable ID match
filename: oktaM365ImmutableID.ps1
subitem_index: 1
post_date: 02.05.2023
tags: okta, m365, mfa, federation, ad, active directory
description: Script to correct M365 migrated user immutable ID to match Okta
highlight: false
highlight_index: 0
highlight_img: /assets/img/highlight_img/user_automation_transparent.png
---

Premise: 

An Organization aquires mutliple companies and migrates the aquired users into its environment. The process is to create Active Directory accounts and sync them to M365 as placeholders for the migration. During the migration, data is overwritten in both AD and M365 user profiles, including the Azure attribute OnPremisesImmutableId, but this is not immediately an issue because Azure will sync back to AD into the custom attribute 'ms-DS-ConsistencyGuid'. This attribute acts as an anchor and maintains the connection between Azure and the AD account.

However, when Okta is introduced for federated M365 MFA, the overwritten data in Azure causes a problem. Okta maps the immutableId for M365 by converting the source AD user profile's GUID to base64. Because of the data overwrite during the migration it leads to a mismatch between the AD user GUID and the Azure Immutable ID. This prevents Okta from handing off the user's authentication back to M365. A temporary solution is to copy the Azure Immutable ID into the app assignment in Okta, but this is not a permanent solution as it leaves the mismatch between the AD user GUID and the Azure Immutable ID. 

Additionally, there were cases where users were already in Okta and during the migration process their profile was linked to the new Active Directory but weren't disconnected from the previous one. In this case the manual overwrite becomes impacted as in the case of two active AD's the immutableId field is blank or in case the previous AD was declared inactive in Okta the value derives from the remaining active AD and cannot be overwritten. 

Solution: 

The obvious solution would be to export all Azure Users and search them by their Azure 'OnPremisesImmutableId' to the AD user that has the matching 'ms-DS-ConsistencyGuid'. A further check can be put in place to see if other properties match such as samAccountName, etc. 
Then after confirming a user mapping, converting the users AD GUID to base64 and running the AzureAD command Set-AzureADUser -objectid azureUPN -immutableId base64ConvertedGUID. This in turn will make AzureADConnect push the new OnPremisesImmutableID to AD and overwrite 'ms-DS-ConsistencyGuid' thus keeping the anchor active with no further intervention needed.

This doesn't cover Okta and needs to be addressed separately in Okta by properly disconnecting these users from their inactive/additional AD and unasigning/reasigning the M365 app/group. Whichever way it's been mapped through.

I would like to address below a different approach that targets affected users directly based on the additional Okta AD.
To be continued.