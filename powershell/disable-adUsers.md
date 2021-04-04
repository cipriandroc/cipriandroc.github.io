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

This is a sample of the CSV (`4.3.2021_disableReport.csv`) output <button onclick="toggle()" class="btn btn-secondary btn-sm">click here to show full csv result</button>

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
    {% for user in site.data.disable-adUsers_csvResult %}
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

Below is a sample of the backup user data that appends to the output text file (`4.3.2021_backupUserData.txt`):<br>
It's all of the adusers's properties before any action is being taken, just in case it needs to be referenced or reverted.
<button onclick="interpret_toggle('backup_user_data_export')" class="btn btn-secondary btn-sm">expand txt file</button>
<div id="backup_user_data_export" style="display:none;">
{% highlight powershell %}
AccountExpirationDate                : 
accountExpires                       : 9223372036854775807
AccountLockoutTime                   : 
AccountNotDelegated                  : False
AllowReversiblePasswordEncryption    : False
AuthenticationPolicy                 : {}
AuthenticationPolicySilo             : {}
BadLogonCount                        : 0
badPasswordTime                      : 0
badPwdCount                          : 0
CannotChangePassword                 : False
CanonicalName                        : orbi.home/_M365x565000.onmicrosoft.com/users/Adele Vance
Certificates                         : {}
City                                 : 
CN                                   : Adele Vance
codePage                             : 0
Company                              : 
CompoundIdentitySupported            : {}
Country                              : 
countryCode                          : 0
Created                              : 2/27/2021 5:33:26 PM
createTimeStamp                      : 2/27/2021 5:33:26 PM
Deleted                              : 
Department                           : 
Description                          : 
DisplayName                          : Adele Vance
DistinguishedName                    : CN=Adele Vance,OU=users,OU=_M365x565000.onmicrosoft.com,DC=orbi,DC=home
Division                             : 
DoesNotRequirePreAuth                : False
dSCorePropagationData                : {2/27/2021 8:07:22 PM, 12/31/1600 4:00:00 PM}
EmailAddress                         : AdeleV@M365x565000.OnMicrosoft.com
EmployeeID                           : 
EmployeeNumber                       : 
Enabled                              : True
Fax                                  : 
GivenName                            : Adele
HomeDirectory                        : 
HomedirRequired                      : False
HomeDrive                            : 
HomePage                             : 
HomePhone                            : 
Initials                             : 
instanceType                         : 4
isDeleted                            : 
KerberosEncryptionType               : {}
LastBadPasswordAttempt               : 
LastKnownParent                      : 
lastLogoff                           : 0
lastLogon                            : 0
LastLogonDate                        : 
LockedOut                            : False
logonCount                           : 0
LogonWorkstations                    : 
mail                                 : AdeleV@M365x565000.OnMicrosoft.com
Manager                              : 
MemberOf                             : {CN=Ask HR,OU=groups,OU=_M365x565000.onmicrosoft.com,DC=orbi,DC=home, 
                                       CN=Operations,OU=groups,OU=_M365x565000.onmicrosoft.com,DC=orbi,DC=home, 
                                       CN=Leadership,OU=groups,OU=_M365x565000.onmicrosoft.com,DC=orbi,DC=home}
MNSLogonAccount                      : False
MobilePhone                          : 
Modified                             : 2/27/2021 6:28:10 PM
modifyTimeStamp                      : 2/27/2021 6:28:10 PM
mS-DS-ConsistencyGuid                : {137, 145, 42, 112...}
msDS-User-Account-Control-Computed   : 0
Name                                 : Adele Vance
nTSecurityDescriptor                 : System.DirectoryServices.ActiveDirectorySecurity
ObjectCategory                       : CN=Person,CN=Schema,CN=Configuration,DC=orbi,DC=home
ObjectClass                          : user
ObjectGUID                           : 702a9189-219f-4cf8-bfc2-1616a6e0de63
objectSid                            : S-1-5-21-3974584507-216888213-1680076431-6169
Office                               : 
OfficePhone                          : +1 425 555 0109
Organization                         : 
OtherName                            : 
PasswordExpired                      : False
PasswordLastSet                      : 2/27/2021 5:33:26 PM
PasswordNeverExpires                 : False
PasswordNotRequired                  : False
POBox                                : 
PostalCode                           : 
PrimaryGroup                         : CN=Domain Users,CN=Users,DC=orbi,DC=home
primaryGroupID                       : 513
PrincipalsAllowedToDelegateToAccount : {}
ProfilePath                          : 
ProtectedFromAccidentalDeletion      : False
pwdLastSet                           : 132589496065143380
SamAccountName                       : AdeleV
sAMAccountType                       : 805306368
ScriptPath                           : 
sDRightsEffective                    : 15
ServicePrincipalNames                : {}
SID                                  : S-1-5-21-3974584507-216888213-1680076431-6169
SIDHistory                           : {}
SmartcardLogonRequired               : False
sn                                   : Vance
State                                : 
StreetAddress                        : 
Surname                              : Vance
telephoneNumber                      : +1 425 555 0109
Title                                : 
TrustedForDelegation                 : False
TrustedToAuthForDelegation           : False
UseDESKeyOnly                        : False
userAccountControl                   : 512
userCertificate                      : {}
UserPrincipalName                    : AdeleV@orbi.home
uSNChanged                           : 1121427
uSNCreated                           : 1121214
whenChanged                          : 2/27/2021 6:28:10 PM
whenCreated                          : 2/27/2021 5:33:26 PM
{% endhighlight %}
<button onclick="interpret_toggle('backup_user_data_export')" class="btn btn-secondary btn-sm">hide txt file</button>
</div>