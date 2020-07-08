# Merge-Hard-sync-Azure-AD-to-Local-premise
This is a script designed to merge or Hard sync multiple local premise accounts to the Azure AD cloud. (Might also work for 

1. Open a powershell instance as an administrator on your local domain controller.
2. Make sure your powershell version is up to date as outdated powershell versions will not automaticall Install-Module AzureAD
3. Run the Initialization

[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
Import-Module ActiveDirectory
Install-Module -Name AzureAD
Connect-AzureAD

4. Enter Your Domain and Path to your user folder

If you are unsure what the path to your user folder is run "Get-ADUser -Identity john.doe" 
on the users account and copy the desired subpath ex."OU=2021,OU=High School 8-12,OU=Students,OU=User Accounts,DC=DOMAIN,DC=NET"

5. Run the script

$Domain = "@domain.net"
$Path = "OU=2021,OU=High School 8-12,OU=Students,OU=User Accounts,DC=DOMAIN,DC=NET"

$Users = Get-ADUser -Filter * -SearchBase $Path | Select-Object -expand SamAccountName
$i=0
$User = "null"
$UsersLength = $Users.length
while($i -lt $UsersLength) {
    $User = $Users[$i];
    $User
    $UserGUID = Get-ADUser -Identity $User | Select-Object -expand ObjectGUID
    $guid = [GUID]$UserGUID
    $bytearray = $guid.tobytearray()
    $immutableID = [system.convert]::ToBase64String($bytearray)
    $immutableID
$CloudID = Get-AzureADUser -ObjectId ($User + $Domain) | Select-Object -expand ObjectId
Set-AzureADUser -UserPrincipalName ($User + $Domain) -ImmutableId $immutableID -ObjectId $CloudID
    $i++
}

6. You may encounter occasional errors depending on syntax, matching accounts or multiple accounts on azure AD. 
ex. a user has to long a username and it has been shortened, middle initials, uppercase, diffrent email adresses...
