---
layout: sublayout
belongsto: powershell
title: Installed Software Report
filename: get-InstalledSoftware.ps1
post_date: 08.30.2020
tags: installed software, search, report
description: Controller script that reports installed software on target machine (local or remote),accepts parameters for software name, target machine, export location.
---

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

