# prenum
![Main](https://github.com/4ndr34z/prenum/blob/main/images/pic.png?raw=true)
## The perils of the Pre-Windows 2000 compatible access group in a Windows Domain.
<p>On Windows 2000 and probably 2003/2008 too (not verified), that group contained groups: Anonymous/Everyone/Authenticated Users
Even if you have upgraded your Domain Controllers to Windows 2022, that group is kept unchanged.</p>

That means a anonymous user can query domain information, like userinfo, group membership, trusts, etc.
Even newly installed domains running Windows 2022, will still have Authenticated users as members in that group.

So if you have user credentials, you can still query all that information.

Prenum is a script exploiting this, by requesting information that might be useful for an attacker.
It can search for computer account with password the same as computername or no password at all. 

This could be the situation if some computer-accounts are pre-created with the box ```Assign this computer account as a pre-Windows 2000 computer``` are ticked, or computer-accounts are created with other tools.

<img src="https://github.com/4ndr34z/prenum/blob/main/images/precomp.png" height="280">

Some automation-tools also create users, leaving the ```Password-Not-Required``` attribute enabled. This means that users **may** set a blank password and **are allowed to do so**, regardless of what kind of password policy is in place. You can and should test for this in your Active Directory: 

    Get-ADUser -Filter {PasswordNotRequired -eq $true}
And fix it:

    Get-ADUser -Identity username | Set-ADUser -PasswordNotRequired $false

All of this should be checked in an old Active Directory.

Prenum is still in early development...


## Functions
- Full AMSI-Bypass
- Reflectively loading [Rubeus](https://github.com/GhostPack/Rubeus) and [Certify](https://github.com/GhostPack/Certify)
- Enumerate and test all computers in AD; check if their password is the same as the computername
- Enumerate all users in AD; check if the password is blank
- Passwordspray all users in AD
- Request Kerberos TGT for computer and/or user-accounts found vulnerable (Using Rubeus)
- Test for vulnerable certificate templates (Using Certify)
- Do simple LDAP searches
- Run any Rubeus command
- Run any Certify command

## Usage
### Enumerate and test users, computers and passwordspraying

    .\Prenum.ps1' -DC menhit -Domain windcorp.htb -Users -Computers -Spraypass 'WelcomeToWindcorp#2023'
### Enumerate computers and request Kerberos TGT if vulnerable computer-accounts are found

    .\Prenum.ps1' -DC menhit -Domain windcorp.htb -Computers -Asktgt

### Running Rubeus

    .\Prenum.ps1 -DC menhit -Domain windcorp.htb -Rubeus "triage"

### Running Certify

    .\Prenum.ps1 -Certify "cas /domain:windcorp.htb"

### LDAP-search for Domain-controllers

    .\Prenum.ps1 -DC menhit -Domain windcorp.htb -ldap "(&(objectCategory=Computer)(userAccountControl:1.2.840.113556.1.4.803:=8192))"

