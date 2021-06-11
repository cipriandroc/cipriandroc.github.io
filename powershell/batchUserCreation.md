---
layout: sublayout
belongsto: powershell
title: Create Batch Users
filename: batchUserCreation.ps1
subitem_index: 4
post_date: 08.30.2020
tags: batch, user creation, ad
description: Controller script to create batch AD user accounts based on a CSV file. It reports the results to a CSV file. Script has multiple checks in place for error prevention.
highlight: true
highlight_index: 3
highlight_img: /assets/img/highlight_img/user_automation_transparent.png
---
`Warning!: This page contains migrated legacy information and script requires a revamp`

Let’s just start off with the results:
{% highlight powershell %}
PS C:\Users\cdroc-a\Documents\GitHub\PWSH_ScriptBucket> .\batchusercreation.ps1
Succesfully created account for: Tomato Toasted as ttoasted
Exported file to C:\DATA\OUTPUT as 09.15_batchUserReport.csv
Succesfully created account for: Tomatio Toasted as totoasted
Appending information: C:\DATA\OUTPUT to file 09.15_batchUserReport.csv
Succesfully created account for: Tomahawk Toasted as tomtoasted
Appending information: C:\DATA\OUTPUT to file 09.15_batchUserReport.csv
Succesfully created account for: Tomatique Toasted as tomatoasted
Appending information: C:\DATA\OUTPUT to file 09.15_batchUserReport.csv
Succesfully created account for: Tomatotters Toasted as tomattoasted
Appending information: C:\DATA\OUTPUT to file 09.15_batchUserReport.csv
Succesfully created account for: Tomalies Toasted as tomaltoasted
Appending information: C:\DATA\OUTPUT to file 09.15_batchUserReport.csv
Succesfully created account for: Tomatter Toasted as tomatttoasted
Appending information: C:\DATA\OUTPUT to file 09.15_batchUserReport.csv
Succesfully created account for: Bubba Smith as bsmith
Appending information: C:\DATA\OUTPUT to file 09.15_batchUserReport.csv
Succesfully created account for: Bubble Smith as busmith
Appending information: C:\DATA\OUTPUT to file 09.15_batchUserReport.csv
Succesfully created account for: Bubbalas Smith as bubsmith
Appending information: C:\DATA\OUTPUT to file 09.15_batchUserReport.csv
Succesfully created account for: Sardines Trott as strott
Appending information: C:\DATA\OUTPUT to file 09.15_batchUserReport.csv
Succesfully created account for: Ruffus Calmus as rcalmus
Appending information: C:\DATA\OUTPUT to file 09.15_batchUserReport.csv
{% endhighlight %}
{% highlight powershell %}
name	                SamAccountName	    password	Status      Details
Tomato Toasted	        ttoasted	    Be4av2q3bR	Succeeded	
Tomatio Toasted	        totoasted	    vmEkcCtFu5	Succeeded	
Tomahawk Toasted	tomtoasted	    537m98RTkS	Succeeded	
Tomatique Toasted	tomatoasted	    MAdk48j226	Succeeded	
Tomatotters Toasted	tomattoasted	    v3p9nRDK52	Succeeded	
Tomalies Toasted	tomaltoasted	    Pgr6sauJeG	Succeeded	
Tomatter Toasted	tomatttoasted	    gkZp25U3E3	Succeeded	
Bubba Smith	        bsmith	            C2HceV9r3h	Succeeded	
Bubble Smith	        busmith	            r27KvDdN69	Succeeded	
Bubbalas Smith	        bubsmith	    uDVNjM4JE5	Succeeded	
Sardines Trott	        strott	            MZqFN7kQcS	Succeeded	
Ruffus Calmus	        rcalmus	            94NV2958rq	Succeeded	
{% endhighlight %}
CSV created by the script, Details column shows error message if any
This was the input:
{% highlight powershell %}
FirstName       LastName
---------       --------
Tomato	        Toasted
Tomatio	        Toasted
Tomahawk	Toasted
Tomatique	Toasted
Tomatotters	Toasted
Tomalies	Toasted
Tomatter	Toasted
Bubba           Smith
Bubble	        Smith
Bubbalas	Smith
Sardines	Trott
Ruffus	        Calmus
{% endhighlight %}
All the script needs is a FirstName and LastName column, it takes care of the rest.
Now some details about the script: It doesn’t have parameters, for now all the properties are hard coded into the script. I didn’t feel this type of script needs to have parameters. There’s just a few $variables that need to be set:
$importfilename: This needs the CSV filename input
$importlocation: The folder location with no ‘\’ at the end
Why are they split? Because this is the type of script that will be used multiple times and the only variable that will change is the filename. I would imagine the location is always gonna be the same, so less pasting to do, less work. I might consider having the script check the creation date of the file as a safety net, essentially asking the user if they’re sure they want to use it if the file is older than the current day.
Also there’s a check in the script that gives out this warning in case it can’t find the import file so the script stops.
{% highlight powershell %}
WARNING: Cannot import CSV file
C:\Data\INPUTfI_8.30.csv is not valid
{% endhighlight %}

$exportlocation: this one is self explanatory, just input a location. The script has a builtin check to test the path, if it doesn’t exist it will output a message and won’t process anything as the process relies heavily on the output information.

{% highlight powershell %}
WARNING: export location invalid
C:\DATA\OUTPUT does not exist!
Won’t be able to export results list
{% endhighlight %}
$OUpath: same deal as before. It needs an OU path in its canonical form, script once again checks to see if the path exists in the directory before it attepts to write anything
{% highlight powershell %}
WARNING: Invalid OU specified!
Please check your variable:
OU=ORBIsStaff,DC=orbi,DC=home
{% endhighlight %}
$mainemail: This one is required to form the userPrincipalName, use the main tenant domain, eg. @chocolate.com
$filename: finally the filename variable, default is ‘batchUserReport’. The script will add the current date to the filename, eg. 09.15_batchUserReport.csv
$userlist: this is the actual CSV import code, it will provide the values in the previous variables. It surpresses the error if it can’t import the file as the precode builtin check will handle the error output better and it will stop the code from actually running.
$setnewuserproperties: this is a hash table containing the default user choices, such as password never expires and user cannot change password. It’s a requirement for these types of accounts. More can be added or existing ones can be changed to $false.

The following lines of code are functions:
Get-AliasUnassigned: the purpose of this function is to create a username/alias based off the FirstName and LastName input values. Under the hood it will also check if there’s an existing username and if there is, it will add the next FirstName letter, after the initial. It will keep doing this until it finds a username that’s not taken. One more thing it does, it sets the username to lowercase letters.
get-CDPassword: this function will generate a 10 character password consisting of numbers and letters/capital letters. It will avoid using characters that can be easily mistaken for instance ‘l’ can be mistaken for 1 or capital i.
Export-Results: this function does a few things to handle the export:
– it will generate the current date and add it to the filename
– it will test the export location provided and it will give a visual warning if it doesn’t exist, along with giving the ‘wrong’ location so you can correct it at a glance.
– it will check to see if there’s an existing file already in the folder with the same name. If there is one already the message on the screen will say Appending to file, in case there isn’t one the message will say Exporting file. As the script runs the first message will likely be exporting and the ones afterwards will show appending.

Now the code itself:
– It starts off by checking to see if a $userlist exists, it will stop the script from running and will give out an error saying Cannot import CSV file in case there’s no file. This is the reason why the import-csv has the Catch error method;
– Then it checks if the $OUPath provided exists it will stop the script from running and display an error along with the path provided if the path is not valid;
– The following test checks to see if the $exportlocation provided exists, once again it will stop the script from running and display an error in case the path is invalid.

The processing part will loop through the list of names and for each object it will:
– create an alias;
– create a password;
– build a hash table that includes the collected properties for the current user;
– attempt to create the user;
– if succesfull it will display a message with showing Full Name and username, will store the information for output and will apply the properties specified at the top of the script in $setnewuserproperties
– if user cannot be created it will display a warning message saying it cannot create the Full name with the username and it will create an error object that will be used for output.
– last part of the loop and the code is using the previously built function Export-Results to export the current user to the CSV file

Here’s what’s left to be done:
– Improve the password generate function. Will be using regex to make sure there’s no more than two consecutive letters and numbers and also to make sure there won’t be cases of 12, ab.
– New-AdUser -samaccountname parameter has a limit of 20 characters. Have to see if there’s a way to bypass that limit. It’s rare that a user has more characters than this limit but it happens. Creating it in the interface doesn’t error out so this is likely because the parameter is forcing the NT depreciated property to use the same name and it fails.