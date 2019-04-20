# CreateUser.ps
#Craete AD user
function createUser{
    param(
        [string]$Domain_local,
        [string]$Dom_ext,
        [string]$Server,
        [string]$Path,
        [string]$PassWord,
        [string]$FirstName,
        [string]$LastName 
        )

    $name = "$fname $lname"
    $samName = ($fname[0] + $lname).ToLower()
    $email = $samName + '@' + $Dom_ext 
    $upn = $samName + '@' + $Domain_local

    $usrAttribs= @{
        AccountPassword = ConvertTo-SecureString $PassWord -AsPlainText -Force
        PasswordNeverExpires = $False
        DisplayName = $name
        Name = $name
        GivenName = $FirstName
        Surname = $LastName
        Path = $Path
        SamAccountName = $samName
        Server = $Server
        Type = "User"
        UserPrincipalName = $upn
        Email = $email
        }

    try{
        New-ADUser @usrAttribs #-ChangePasswordAtLogon $true
        Enable-ADAccount -Identity $samname -Server $pdc
        $time = Get-Date
        Write-Host "`n$time ->> An AD account for user $name was created in $uOU"
        }
    catch [Microsoft.ActiveDirectory.Management.ADIdentityNotFoundException]{
        $msg = "Line Number: $($_.InvocationInfo.ScriptLineNumber) ->> $($_.Exception.Message) "
        Write-warning -Message "`n$msg"
        }

}
#static Variable
    $pwd = 'Pa$$w0rd' #OR    read-host -prompt "The Common Password, make sure you change them"
    
# 0. User input
    $arrUsr = @{
    'FirstName' = 'Bernard'
    'LastName' = 'Han'
    }
    $fname = $arrUsr['FirstName']
    $lname = $arrUsr['LastName']

# 1. Query Domain and User
    $d = get-addomain
    $u = get-aduser $env:USERNAME

# 2. Retrieve/Furnish Data:
    $dom = $d.Forest
    $dom_ext = $dom.Split('.')[0] + '.edu'
    $pdc = $d.PDCEmulator
    
    $uOU = (($u.DistinguishedName).TrimStart(($u.DistinguishedName).Split(',')[0])).trimstart(',')
    $uOU = 'OU=Tech-Support-Users,OU=Tech-Support,DC=mars,DC=local'

createUser -Domain_local $dom -Dom_ext $dom_ext -Server $pdc -Path $uOU -PassWord $pwd -FirstName $fName -LastName $lname
