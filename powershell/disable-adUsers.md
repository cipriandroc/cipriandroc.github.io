---
layout: sublayout
belongsto: powershell
title: Disable ADUsers
filename: disable-adUsers.ps1
subitem_index: 4
post_date: 04.02.2021
tags: batch, user disable, ad
description: Disables a batch of ADUsers based on samAccountName, sets description, moves OU, reports result, backs up user info
highlight: true
highlight_index: 1
highlight_img:
livemode: true
related_articles: 
  - get-inactiveADUser 
  - convert read data to ADUser
github_link: github link placeholder
---
<script>
function range(start, end) {
  return Array(end - start + 1).fill().map((_, idx) => start + idx)
}
function interpret_toggle(item_id) {
 if( document.getElementById(item_id).style.display=='none' ){
   document.getElementById(item_id).style.display = '';
 }else{
   document.getElementById(item_id).style.display = 'none';
 }
}
async function toggle() {
  var nr_range = range(4, 16);
  for(number of nr_range){
    create_id = 'hidethis'+number
    interpret_toggle(create_id)
  }
}
</script>

<hr>
<h5>Description</h5>
<hr>
This script `disables` ADUsers based on an input list (csv) containing samAccountName values.
It also `changes the description` of the targeted user/s, has the ability to `move OU` to designated Disable OU and most importantly `backs up user data` to a text file before it performs any action on the targeted user.

Finally it will `export a report` of user properties after action is performed and lists action taken on the accounts.
Through the course of runtime it will `display detailed information` on the console for every action being performed.

There are a couple of `safety nets` put in place to prevent the script from running if any important parameter/variable has been omitted, contains wrong information or if list provided has invalid formatting.

This script was run multiple times in production and it performed up to expectations.
In this article I will be detailing every aspect of it.


The structure of it is made out of:
- input variables (user list, description, export locations)
- classes and functions
- initialization (performs safetynet checks)
- script block

`Logic Diagram` will show detailed information on this subject. <a href="#logic_diagram" style="color:black;">`[ view now ]`</a>
<hr>
<h5>Configuration</h5>
<hr>
When running this script in a new environment make sure to configure the variables. After it's been configured with the right values the script can be run from the console requiring minimal input.

<table class="table">
  <thead>
    <tr>
      <th scope="col">parameter name</th>
      <th scope="col">description</th>
    </tr>
  </thead>
  <tbody>
    {% assign variable_table = site.data.powershell.disable-adUsers_data | where: "type","variable" %}
    {% assign parameter_table = site.data.powershell.disable-adUsers_data | where: "type","parameter" %}
    {% for variable in parameter_table %}
      {% assign link_button = nil %}
      {% if variable.link_id %}
        {% assign link_button = ' (<a href="#' | append: variable.link_id   | append: '" style="color:blue">' | append: 'more info' | append: '</a>)' %}
      {% endif %}
      <tr>
        <td><code class="language-plaintext highlighter-rouge" style="word-break:unset;">{{variable.name}}</code></td>
        <td>{{variable.description}}{{link_button}}</td>
      </tr>
    {% endfor %}
  </tbody>
    <thead>
    <tr>
      <th scope="col">variable name</th>
      <th scope="col">description</th>
    </tr>
  </thead>
  <tbody>
    {% for variable in variable_table %}
      {% assign link_button = nil %}
      {% if variable.link_id %}
        {% assign link_button = ' (<a href="#' | append: variable.link_id   | append: '" style="color:blue">' | append: 'more info' | append: '</a>)' %}
      {% endif %}
      <tr>
        <td><code class="language-plaintext highlighter-rouge" style="word-break:unset;">{{variable.name}}</code></td>
        <td>{{variable.description}}{{link_button}}</td>
      </tr>
    {% endfor %}
  </tbody>
</table>

<hr>
<h5>Execution</h5>
<hr>
At runtime the script will perform the following checks and stop in case any of them fail and provide console information:
- imports the userlist CSV file
- checks if the provided CSV file contains the samAccountName property
- tests the provided export location folder if exists and if you have RW permission to it
- verifies if the connected domain matches the domain specified in the variable
- if -disableOU parameter is specified, verifies if the OU is valid and exists; currently the script doesn't check if the user that's running the script has the permission to move the object but if that's the case the console will show a warning message that moving the user failed and the report will also specify this information as well as it can be checked from the user distingusihed name that's being outputed

Below is a part of the console display information at runtime:

{% highlight powershell %}
> .\Disable-ADUsers.ps1 -ticketNumber hd-235 -moveOU
VERBOSE: Attempting to import userdata file .\userlist.csv
Succesfully imported userdata file.                                                                                     
VERBOSE: Verifying user data file for samAccountName property.                                                          
Property found.                                                                                                         
VERBOSE: Verifying export folder location.                                                                              
Test exportPath OK                                                                                                      
VERBOSE: Verifying connected domain
Domain connection OK
VERBOSE: Gathering user data for: adelev
VERBOSE: Backing up user data to file: 4.5.2021_backupUserData.txt
VERBOSE: Attempting to change description filed for user: adelev
Changed description to: Disabled per AD Cleanup Project - hd-235
VERBOSE: Attempting to disable user: adelev
adelev has been disabled.
VERBOSE: Attempting to move user to disable OU
adelev has been moved to disable OU
VERBOSE: Exporting disable information to file
VERBOSE: Gathering user data for: awilber
VERBOSE: Backing up user data to file: 4.5.2021_backupUserData.txt
VERBOSE: Attempting to change description filed for user: awilber
Changed description to: Disabled per AD Cleanup Project - hd-235
VERBOSE: Attempting to disable user: awilber
awilber has been disabled.
VERBOSE: Attempting to move user to disable OU
awilber has been moved to disable OU
VERBOSE: Exporting disable information to file
{% endhighlight %}

<hr>
<h5>Output</h5>
<hr>
This is a sample of the CSV exported (`4.3.2021_disableReport.csv`)

<!-- table block -->
<a href="#" onclick="toggle(); return false;" style="color:black; float:right; margin:auto;">`[ expand table ]`</a>
<table class="table" id="csvOutput">
  <thead>
    <tr>
      <th scope="col">#</th>
      <th scope="col">samAccountName</th>
      <th scope="col">DistinguishedName</th>
      <th scope="col">Enabled</th>
      <th scope="col">Description</th>
      <th scope="col">action</th>
    </tr>
  </thead>
  <tbody>
    {% assign i = 1 %}
    {% for user in site.data.powershell.disable-adUsers_csvResult %}
    {% if i > 3 %}
      {% assign hide_tb_row = 'id="hidethis' | append: i | append: '"' | append: ' style="display:none;"' | append: ' class="set-transition"'%}
    {% endif %}
    <tr {{hide_tb_row}}>
      <th scope="row">{{i}}</th>
      <td>{{user.samAccountName}}</td>
      <td>{{user.DistinguishedName}}</td>
      <td>{{user.Enabled}}</td>
      <td>{{user.Description}}</td>
      <td>{{user.action}}</td>
    </tr>
    {% assign i = i | plus:1 %}
    {%endfor%}
  </tbody>
</table>

Next is a sample of the backup user data that appends to the output text file (`4.3.2021_backupUserData.txt`)<br>
It's all of the adusers's properties before any action is being taken, just in case it needs to be referenced or reverted.
<a href="#" onclick="interpret_toggle('backup_user_data_export'); return false;" style="color:black" id="userBackupExport">`[ show txt file ]`</a>
<!-- text block -->
<div id="backup_user_data_export" style="display:none;">
{% highlight powershell %}
{% include /powershell_props/disable-adUsers_props/backup.txt %}
{% endhighlight %}
<a href="#" onclick="interpret_toggle('backup_user_data_export'); return false;" style="color:black">[ hide txt file ]</a>
</div>

<hr>
<h5 id="logic_diagram">Logic Diagram</h5>
<hr>

<img src="/assets/img/disable-adUsers_img/flowchart.svg" style="size:100%;">