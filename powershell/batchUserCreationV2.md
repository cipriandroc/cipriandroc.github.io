---
layout: sublayout
belongsto: powershell
title: Create Batch Usersv2
filename: batchUserCreation.ps1
subitem_index: 4
post_date: 08.30.2020
tags: batch, user creation, ad
description: Controller script to create batch AD user accounts based on a CSV file. It reports the results to a CSV file. Script has multiple checks in place for error prevention.
highlight: true
highlight_index: 3
highlight_img: /assets/img/highlight_img/user_automation_transparent.png
---
In this article, we will go over a script that is used to create multiple user accounts in a batch process. The script takes a list of names as input and generates unique accounts for each individual, outputting the results to a CSV file.

Here is an example of the output of the script:

{% highlight powershell %}
PS C:\Users\cdroc-a\Documents\GitHub\PWSH_ScriptBucket> .\batchusercreation.ps1
Successfully created account for: Tomato Toasted as ttoasted
Exported file to C:\DATA\OUTPUT as 09.15_batchUserReport.csv
Successfully created account for: Tomatio Toasted as totoasted
Appending information: C:\DATA\OUTPUT to file 09.15_batchUserReport.csv
Successfully created account for: Tomahawk Toasted as tomtoasted
Appending information: C:\DATA\OUTPUT to file 09.15_batchUserReport.csv
Successfully created account for: Tomatique Toasted as tomatoasted
Appending information: C:\DATA\OUTPUT to file 09.15_batchUserReport.csv
Successfully created account for: Tomatotters Toasted as tomattoasted
Appending information: C:\DATA\OUTPUT to file 09.15_batchUserReport.csv
Successfully created account for: Tomalies Toasted as tomaltoasted
Appending information: C:\DATA\OUTPUT to file 09.15_batchUserReport.csv
Successfully created account for: Tomatter Toasted as tomatttoasted
Appending information: C:\DATA\OUTPUT to file 09.15_batchUserReport.csv
Successfully created account for: Bubba Smith as bsmith
Appending information: C:\DATA\OUTPUT to file 09.15_batchUserReport.csv
Successfully created account for: Bubble Smith as busmith
Appending information: C:\DATA\OUTPUT to file 09.15_batchUserReport.csv
Successfully created account for: Bubbalas Smith as bubsmith
Appending information: C:\DATA\OUTPUT to file 09.15_batchUserReport.csv
Successfully created account for: Sardines Trott as strott
Appending information: C:\DATA\OUTPUT to file 09.15_batchUserReport.csv
Successfully created account for: Ruffus Calmus as rcalmus
Appending information: C:\DATA\OUTPUT to file 09.15_batchUserReport.csv
{% endhighlight %}

The resulting CSV file looks like this:

{% highlight powershell %}
name SamAccountName password Status Details
Tomato Toasted ttoasted Be4av2q3bR Succeeded
Tomatio Toasted totoasted vmEkcCtFu5 Succeeded
Tomahawk Toasted tomtoasted 537m98RTkS Succeeded
Tomatique Toasted tomatoasted MAdk48j226 Succeeded
Tomatotters Toasted tomattoasted v3p9nRDK52 Succeeded
Tomalies Toasted tomaltoasted Pgr6sauJeG Succeed
{% endhighlight %}
{% highlight powershell %}
FirstName LastName

Tomato Toasted
Tomatio Toasted
Tomahawk Toasted
Tomatique Toasted
Tomatotters Toasted
Tomalies Toasted
Tomatter Toasted
Bubba Smith
Bubble Smith
Bubbalas Smith
Sardines Trott
Ruffus Calmus
{% endhighlight %}

This script can be useful for system administrators who need to quickly create multiple user accounts at once, rather than having to manually create each one individually. It can save a significant amount of time, especially in large organizations where there may be hundreds or thousands of users to be added.