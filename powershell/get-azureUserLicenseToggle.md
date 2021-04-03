---
layout: sublayout
belongsto: powershell
title: Azure License Assignment Report
filename: get-AzureUserLicenseToggleDetails.ps1
subitem_index: 1
post_date: 09.05.2020
tags: o365, azure, license, report
description: Reports o365 licenses assigned to users. Provide a list of user names and the results are exported to a CSV file.
highlight: true
highlight_index: 1
highlight_img: /assets/img/highlight_img/report_transparent.png
---

<https://github.com/cipriandroc/Get-AzureUserLicenseToggleDetails.ps1>

This script was created to report Office365 User License assigments. It’s purpose is to capture the license assignments to users wether it’s needed for reporting or before a major change such as perhaps switching to an Azure Group that has default assigments.
The script displays a breakdown of all the license object IDs along with their Sublicenses and their status.
By default the command Get-AzureADUserLicenseDetail takes an ObjectID parameter and returns licenses along with their sublicenses as a sub array. It makes it difficult to export to an easy readable format.

This script makes it easy to export all this information. First of all it takes a `userPrincipalName` as input. So you don’t have to run separately the `Get-MSOLUser` command to get the user `ObjectID`. That command itself has a limitation as the `-userPrincipalName` parameter doesn’t support multiple strings. So by default you’re stuck bringing one user at a time or at the other end of the spectrum, running the command by itself and bringing all the users in the tenant.

This script has a function built inside that accepts multiple username input and generates an object for each of them. I built an extra functionality inside the function where it can accept just the username part, without the need of `@domain.com`, although I still recommend writing the full `UPN`. The function extracts the default domain from the tenant and adds it to the username if the input user is missing the domain part. Although in a tenant with multiple subdomains, where users have their `UPN` as a different domain than the default one you’ll still need to provide the full `UPN` for the user. A future release could include looping through all subdomains and testing if the user exists, I didn’t see a necessity for it at the moment.
By default the `-UserList` parameter will load up content of a text file located in the predefined path. The text file needs to include a list of users, each line having an email address specified, no column title needed.

Specifying the `-ExportPath` along with the path name will export the file. The filename is hardcoded inside the script and is indicative of the scripts purpose `‘licenseTogglesExport’`.
The builtin function adds the current date to the filename and also handles the path. The user has to input the path in the form of `C:\` or `C:\<folder>`. The function will detect if there’s a backslash at the end and add it if it’s not present so it won’t matter if it’s `C:\<folder>\` or `C:\<folder>` .

There are a few builtin checks that give a warning wether the path is not correct or if the `-exportpath` parameter is missing after the script runs, it won’t block it’s functionality from running.
{% highlight powershell %}
WARNING: No -exportpath parameter provided, results have not be exported
WARNING: c:\data\outputfsdf not found! Results have not been exported!
{% endhighlight %}
There is another warning message when there’s no license found
{% highlight powershell %}
WARNING: No licenses found
{% endhighlight %}
In a future realease I want to generate an object when a user is not found or it doesn’t have licenses assigned, so it can be reported as well.

the script displays users without licenses as well as user not found error for better reporting. By default the command doesn’t return any result if the user doesn’t have a license and gives an error in the shell if the objectID is not valid.
Note that the shell will likely not fit all the information, this script is optimized for outputting to a `CSV` file for readability. Shell display is usefull to see the results at a glance and see if there are any errors.
a closer look at results objects, the controller script uses the formatting shown above, these are results from the functions themselves ran independantly.

Example of user that doesn’t have licenses assigned:
{% highlight powershell %}
userPrincipalName : cdroc@cipriandroclivecom.onmicrosoft.com
userObjectID : 9ab97a7e-9352-45d9-a82b-afc6470c25e2
LicenseObjectId :
AppliesTo :
ProvisioningStatus :
ServicePlanId :
ServicePlanName :
{% endhighlight %}
Example of user that doesn’t exist in the domain. Notice the function inside the script took the username string and turned it into userPrincipalName format using the default domain inside the tenant. To avoid this input the userPrincipalName
{% highlight powershell %}
userPrincipalName : fdasf@cipriandroclivecom.onmicrosoft.com
userObjectID : error! user not found
LicenseObjectId :
AppliesTo :
ProvisioningStatus :
ServicePlanId :
ServicePlanName :
{% endhighlight %}
This is an example what the function is also capable of, it works standalone and you can feed it an `objectID` directly using the `-objectid` parameter. The controller script utilizes its `-azureuser` parameter which accepts an array of Objects that contain `username`,`userPrincipalName` and `ObjectID`
{% highlight powershell %}
userPrincipalName : error: invalid ObjectID
userObjectID : b58e041f-4ece-44b7-8523-6593f1af0f4d
LicenseObjectId :
AppliesTo :
ProvisioningStatus :
ServicePlanId :
ServicePlanName :
{% endhighlight %}

Example of output `CSV` showing a mix of all user output possibilites
Running `get-help .\Get-AzureUserLicenseToggleDetails.ps1 -full` will detail parameters, examples, scopes, all the needed information to run the script.

Hope you find this script useful and thank you for reading!

