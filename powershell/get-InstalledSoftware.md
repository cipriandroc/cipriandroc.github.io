---
layout: sublayout
belongsto: powershell
title: Installed Software Report
filename: get-InstalledSoftware.ps1
subitem_index: 3
post_date: 08.30.2020
tags: installed software, search, report
description: Controller script that reports installed software on target machine (local or remote), accepts parameters for software name, target machine, export location.
highlight: true
highlight_index: 4
highlight_img: /assets/img/highlight_img/software_transparent.png
github_link: https://github.com/cipriandroc/Get-InstalledSoftware.ps1
---
Purpose of this script is to output information regarding installed software on the local computer or remote computers. It has the ability to output on screen as well as export to a CSV file.

Let’s start off with the basics
{% highlight powershell %}
PS C:\Users\cdroc-a\Documents\GitHub\Get-InstalledSoftware.ps1P .\Get-InstalledSoftware.ps1 edge
ComputerName    method      searched   DisplayName             DisplayVersion       ApplicationGUID         InstallOate 
ORBI7450        registry    edge       Microsoft Edge          85.8.564.51          Microsoft Edge          9/19/2020 
ORBI7450        registry    edge       Microsoft Edge Update   1.3.135.29           Microsoft Edge Update 
ORBI7450        get-package edge       Microsoft Edge          85.8.564.51                                  N/A 
ORBI7450        get-package edge       Microsoft Edge Update   1.3.135.29                                   N/A 
{% endhighlight %}
Pretty simple, run the script, input the search term as a positional parameter and get the results. Let’s discuss the results and the layout.
We’ll start off one by one.

`ComputerName`: since this script can retrieve information from multiple computers using the parameter -ComputerName it also outputs the name of the computer it ran the query on. This is very useful when the tool is used for reporting.

`method`: Since there’s no sure way of getting installed software information I’m using the 2 best methods to retrieve it. One is registry where we have the keys for installed software, this is the exact information that’s being displayed in control panel / appwiz.cpl . There’s a catch though, they’re stored in 2 locations in registry: one for 32bit and another for 64bit. The script retrieves information from both locations and concatenates the information. The downside to retrieving information from registry is that it might not be accurate. You can easily delete the key and the application would continue to function normally as it’s not impacted. It also depeneds on the developer how well they built their installer, there could be apps that won’t add anything to registry. Come in the second method using the cmdlet get-package builtin Powershell. This will retrieve the software you input in a similar fashion this script behaves, bar quering remote computers and reporting. The biggest downside to it is that it won’t output the install date, so it would hinder the reporting capabilities.
<div style="background-color:#FFFFE0;line-height:1;">
<small>
LaterEdit: I actually did find the object produced has a InstallDate value, it’s nested very deep into one of the properties: get-package *chrome* | Select-Object -expand meta | select -expand attributes | select -expand values. For the sake of showing my knowledge at the time, I won’t modify this article further
There’s still point to all of this though. As of now (9.11.20) I didn’t extract the date yet as I just found out and basically you have to match the right value found in the Values property to it’s Key counterpart found in the Key property. I have a guess that you can match the position and I will work on that, however, there’s another major factor. In Powershell (Core) 7 Get-Package doesn’t support the Programs and Msi providers https://github.com/PowerShell/PowerShell/issues/7844
This enforces a reason to have both options available for redundancy.
</small>
</div>

Since these two methods compliment eachother I decided to use both, more information is always better than less so we’re making sure we’re not missing out on anything.

`searched`: as the name suggests, this is the search term that generated the output on that specific line, to easily track the results. The -software parameter accepts multiple items separated by commas

`DisplayName`, `DisplayVersion`, `Application GUID`, `InstallDate`: as the name suggests, these are the default property names queried from registry. The cmdlet get-package outputs slightly different and less information so I used a calculated property selection to manipulate the results into using the same property name and thus output in the same columns. I decided to output the GUID just for the convenience in case you’re trying to uninstall an application based on its GUID. Instead of having to go look it up you have a one stop place for it.

<img src="https://cipriandroc.files.wordpress.com/2020/09/screen-shot-2020-09-13-at-7.13.32-pm.png">

<img src="https://cipriandroc.files.wordpress.com/2020/09/screen-shot-2020-09-13-at-7.14.54-pm.png">

Onto a more advanced output. This one had 3 search terms, 2 computers and the parameter to export results to a location.

The script has a builtin function to export a csv file. It takes -results, -exportlocation and -filename parameters. Basically you either input an export location or chose a predefined one eg. c:\data\output . The script itself already has -filename declared inside of it as ‘SoftwareReport’, it’s hardcoded as the name is self descriptive of the output. Then the function builds itself the entire location path along with the CSV filename, it adds the date in the filename eg. 08.30_SoftwareReport.csv . The function also checks if the path exists and sends out an error if it’s not accessible and lists the path so you can view what was input. If the path exists then it proceeds to export as CSV with the parameters: append and notypeinformation hard coded. Append is needed as the software lookup function is using a foreach to search and output so it keeps adding to the CSV. Notypeinformation is simply because there’s no need to view the object time in the CSV and just get the columns directly.
The results parameter is the data going into the function, it was meant to be descriptive and self explenatory as the script

### BuildLog - notes

this was created from a need one day to determine if a certain package version was deployed to a pool of workstations.

The challenge here is the computers had to be on VPN to be able to append the information to a central location

Let’s dive right into it and then we can explain the thought process and elements

##Replace framework in $software with desired application name
{% highlight powershell %}
$software = '*yoursoftware*'
$ExportLocation = "c:\yourexportlocation.csv"

$location1 = Get-ItemProperty HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*
$location2 = Get-ItemProperty HKLM:\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*
$results = $null;
$results = $location1 + $location2 | Where-Object {($_.displayname -like "$software")} 

if ($results -ne $null) {
    $results | Select-Object $Env:ComputerName,DisplayName, DisplayVersion, 
    @{Name="InstallDate"; Expression={([datetime]::ParseExact($_.InstallDate, 'yyyyMMdd', $null)).toshortdatestring()}} | Format-Table -AutoSize  | out-file -append $ExportLocation
}
else {
    write-host "$Env:ComputerName doesn't have the $software installed";
}
{% endhighlight %}
Note that this script was built with an export function in mind, if you only want Powershell to display the information then remove the last line inside the IF statement (out-file)

We start off by declaring two variables:
$software , type the name of the software you’re looking for here, leave the single quotes and the * in there so that the script can search for all forms of that name. eg. Microsoft will give results for Microsoft Office, Microsoft .NET Framework, etc.
$location , enter your export location where the CSV appends. Notice I used to word append, this means that the file won’t get overwritten but rather information will keep adding to it.

OK so let’s talk how this script operates. Well, there’s a few ways to get installed software, some of which involve WMI Objects. Now if we start looking for information on this we’ll find a lot of articles that advice against. It goes as far as running the WMI query somehow activates the installers of some applications. WMI is such an old tech that we’re gonna try to go around it.

Now one good place to look for installed software is the Windows registry, but there’s two locations to look into, one is for 32bit apps and one’s for 64bit apps. Interesting they’re not in a central location but it makes sense. So because of this we have 2 variables that point to these locations individually and we end up creating a variable that includes them both. I actually created this before I started learning the books so I was just practicing with the shell and figured I’d try to add them (+). It’s quite enjoyable to discover things like these.
So moving on, we do a sort with Where-Object for the displayname property that’s -like our $software variable. I used -like not -equal just so I can be more flexible on results.

Next we use the IF construct. Just in case there’s no results I want some kind of visual information rather than looking like the script didn’t do anything.
So inside the IF we grab our location sum variable, $results, and do a selection on: ComputerName (for reporting), DisplayName, DisplayVersion (this was a requirement as the version had to be very specific) and finally InstallDate. By default the install date shows up as an unreadable format so we had to create a hash table @{} where we declared the name of our custom property as InstallDate and it runs the expression by converting our property to a human readable output. Then it continues to pipe to a autosized table. And lastly this is all piped to the out-file command where it appends to the selected location at the start of the script.

The Else just reports the current computer doesn’t have the $software installed.

So there you have it, this can be built upon in quite a few different ways, at the time I didn’t know about functions and parameters which would’ve been nicer to have. I do just want to capture my knowledge at the time this was created. In case I will modify this script it’s likely I will continue to post it under here, or at the very least include this iteration.

