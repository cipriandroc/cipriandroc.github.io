---
layout: sublayout
belongsto: powershell
title: Disable ADUsers
filename: disable-adUsers.ps1
subitem_index: 4
post_date: 04.02.2021
tags: batch, user disable, ad
description: Disables batch of ADUsers based on samAccountName
highlight: 
highlight_index: 
highlight_img:
livemode: true
---
This script disables ADUsers based on an input list (csv) containing samAccountName values.
It also changes the description of the targeted user/s, has the ability to move OU to designated Disable OU and most importantly backs up user data to a text file before it performs any action on the targeted user.

Finally it will output a report of user properties after action is performed and lists action taken on the account.
Through the course of runtime it will display detailed information on the console for every action being performed.

There are a couple of safety nets put in place to prevent the script from running if any important parameter has been omitted or if list provided has invalid formatting.

This script was run multiple times in production and it performed up to expectations.
In this article I will be detailing every aspect of it.

The structure of it is made out of:
- input variables (user list, description, export locations)
- initialization (performs safetynet checks)
- classes and functions
- script block

When running this script in a new environment make sure to configure the variables. After it's been configured with the right values the script can be run from the console requiring minimal input.

This is a sample of a CSV output:

<table class="table">
  <thead>
    <tr>
      <th scope="col">#</th>
      <th scope="col">DisplayName</th>
      <th scope="col">samAccountName</th>
      <th scope="col">Enabled</th>
      <th scope="col">Description</th>
      <th scope="col">ActionTaken</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">1</th>
      <td>Mark Otto</td>
      <td>motto</td>
      <td>False</td>
      <td>Disabled per HD-432</td>
      <td>disabled</td>
    </tr>
    <tr>
      <th scope="row">2</th>
      <td>Jacob Thornton</td>
      <td>jthornton</td>
      <td>True</td>
      <td></td>
      <td>couldn't disable</td>
    </tr>
    <tr>
      <th scope="row">3</th>
      <td>Larry Bird</td>
      <td>lbird</td>
      <td>False</td>
      <td></td>
      <td>already disabled</td>
    </tr>
  </tbody>
</table>