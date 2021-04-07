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
Every so often we receive a request to perform some actions on a batch of users. Usually these requests don't quite come in the format that we need to use, they won't include samAccountNames, userPrincipalNames or DistinguishedNames.

Regular users, heck even IT Staff, will submit a request based on a copy of unformatted names, email addresses. Anything they can conveniantly grab. I've received requests that are straight copy pastas from cc'd fields or some odd excel columns that are upside down such as just a column that contains Last Name First Name , in that order. So isntead of googling Excel formuals for the nth time I found it more conveniant to write scripts around these instances. 

I found this useful even before those odd requests, I was working on a case where the email list I was provided with was accurate, but even those emails had to be matched against an user object to even perform the actions needed. 
<hr>
<h5>Configuration</h5>
<hr>
There's really not a lot to configure here. We just have 2 variables:
<table class="table">
  <thead>
    <tr>
      <th scope="col">parameter name</th>
      <th scope="col">description</th>
    </tr>
  </thead>
  <tbody>
    {% assign variable_table = site.data.powershell.parseReadDataToADUser_data | where: "type","variable" %}
    {% assign parameter_table = site.data.powershell.parseReadDataToADUser_data | where: "type","parameter" %}
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