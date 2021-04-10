---
layout: sublayout
belongsto: powershell
title: Parse Read Data To ADUser
filename: parseReadDataToADUser.ps1
subitem_index: 2
post_date: 04.06.2021
tags: convert, parse, match, aduser
description: Multiple scripts in article. Used to parse information such as Display Names, email addresses and match to their respective ADUser
highlight: false
highlight_index: 
highlight_img:
livemode: true
related_articles: 
  - disable-adUsers
github_link: github link placeholder
---
<hr>
<h5>Description</h5>
<hr>
Every so often we receive a request to perform some actions on a batch of users. Usually these requests don't quite come in a desirable format, they won't include samAccountNames, userPrincipalNames or DistinguishedNames and ultimately these are the values we need to work with to perform actions on user accounts.

Users will submit a request based on a copy of unformatted names, email addresses. Anything they can conveniantly grab. I've received requests that are straight copy pastas from cc'd fields or some odd excel columns that are upside down such as just a column that contains Last Name First Name , so reverse. Intead of googling Excel formuals for the nth time I found it more conveniant to write scripts around these instances. 

<hr>
<h5>Configuration</h5>
<hr>
{% assign parameter_table = site.data.powershell.parseReadDataToADUser_data | where: "type","parameter" %}
{% assign variable_table = site.data.powershell.parseReadDataToADUser_data | where: "type","variable" %}
<table class="table">
  {% if parameter_table %}
  <thead>
    <tr>
      <th scope="col">parameter name</th>
      <th scope="col">description</th>
    </tr>
  </thead>
  <tbody>
      {% for parameter in parameter_table %}
        {% assign link_button = nil %}
        {% assign tip = nil %}
        {% if parameter.link_id %}
          {% assign link_button = ' (<a href="#' | append: parameter.link_id   | append: '" style="color:blue">' | append: 'more info' | append: '</a>)' %}
        {% endif %}
        {% if parameter.tip %}
          {% assign tip = '<small>' | append: parameter.tip | append: '</small> ' %}
        {% endif %}
        <tr>
          <td><code class="language-plaintext highlighter-rouge" style="word-break:unset;">{{parameter.name}}</code></td>
          <td>{{tip}}{{parameter.description}}{{link_button}}</td>
        </tr>
    {% endfor %}
  </tbody>
  {% endif %}
  {% if variable_table %}
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
  {% endif %}
</table>

The `Mandatory set` means that either of the two switches are required in order for the script to run, but the two of them at the same time won't function. In the case where both switches need to be combined the -Recoursive switch comes into play. This is a fallback solution in case the selected switch fails to match or doesn't even exist.

<hr>
<h5>Execution</h5>
<hr>
At runtime the script will perform the following checks and stop in case any of them fail and provide console information:
- imports the userlist CSV file
- checks if the provided CSV file contains Email, DisplayName or both in case Recoursive is used
- checks if export CSV already exists and gives warning that the script will append information to it
- tries to check for connection to Active Directory by importing the AD pwsh module

<hr>
<h5>Output</h5>
<hr>
This is a sample of the CSV exported (`....csv`)