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
<div>
  <h5>Related articles</h5>
  <a href="#"><small>get inactive ADUser</small></a><span> | </span>
  <a href="#"><small>convert email address to ADUser</small></a><span> | </span>
  <a href="#"><small>convert DisplayName to ADUser</small></a><span> | </span>
  <a href="#"><small>convert FirstName LastName to DisplayName</small></a>
</div>
<hr>
<h5>Description</h5>
<hr>
This script `disables` ADUsers based on an input list (csv) containing samAccountName values.
It also `changes the description` of the targeted user/s, has the ability to `move OU` to designated Disable OU and most importantly `backs up user data` to a text file before it performs any action on the targeted user/

Finally it will `export a report` of user properties after action is performed and lists action taken on the accounts.
Through the course of runtime it will `display detailed information` on the console for every action being performed.

There are a couple of `safety nets` put in place to prevent the script from running if any important parameter has been omitted or if list provided has invalid formatting.

This script was run multiple times in production and it performed up to expectations.
In this article I will be detailing every aspect of it.


The structure of it is made out of:
- input variables (user list, description, export locations)
- initialization (performs safetynet checks)
- classes and functions
- script block

When running this script in a new environment make sure to configure the variables. After it's been configured with the right values the script can be run from the console requiring minimal input.
<hr>
<h5>Output</h5>
<hr>
This is a sample of the CSV exported (`4.3.2021_disableReport.csv`)

<a href="#" onclick="toggle()" style="color:black; float:right; margin:auto;">`[ expand table ]`</a>

<table class="table">
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
<a href="#" onclick="interpret_toggle('backup_user_data_export')" style="color:black">`[ show txt file ]`</a>
<div id="backup_user_data_export" style="display:none;">
{% highlight powershell %}
{% include /powershell_props/disable-adUsers_props/backup.txt %}
{% endhighlight %}
<a href="#" onclick="interpret_toggle('backup_user_data_export')" style="color:black">[ hide txt file ]</a>
</div>
