<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1> automatic Bulk users creation on active directory using power shell and establishing group policy (Azure)</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.<br />



<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>High-Level Deployment and Configuration Steps</h2>

- Step 1 — Enable RDP for Domain Users
- Step 2 — Run the Random User Creation Script on DC-1
- Step 3 — Verify Accounts in ADUC
- Step 4 — Test Login with a Standard User

<h2>Deployment and Configuration Steps</h2>
<img width="1642" height="840" alt="Screenshot (122)" src="https://github.com/user-attachments/assets/de4a813a-c129-42d0-bc0f-797ae49257ac" />

Step 1 — Enable RDP for Domain Users

Log in to Client-1 as mydomain\jane_admin.

Open System Properties → Remote Desktop.

Enable Remote Desktop.

Click “Select users that can remotely access this PC” → Add → mydomain\Domain Users.
Also in the active directory manger right under clients and add a group policy labeled "allow remote desk top"
<br />

--
Step 2 — Run the Random User Creation Script on DC-1

Open PowerShell ISE as Administrator on DC-1 and paste this script:
 # ----- Edit these Variables for your own Use Case ----- #
$PASSWORD_FOR_USERS   = "Password1"
$NUMBER_OF_ACCOUNTS_TO_CREATE = 10000
# ------------------------------------------------------ #

Function generate-random-name() {
    $consonants = @('b','c','d','f','g','h','j','k','l','m','n','p','q','r','s','t','v','w','x','z')
    $vowels = @('a','e','i','o','u','y')
    $nameLength = Get-Random -Minimum 3 -Maximum 7
    $count = 0
    $name = ""

    while ($count -lt $nameLength) {
        if ($($count % 2) -eq 0) {
            $name += $consonants[$(Get-Random -Minimum 0 -Maximum $($consonants.Count - 1))]
        }
        else {
            $name += $vowels[$(Get-Random -Minimum 0 -Maximum $($vowels.Count - 1))]
        }
        $count++
    }

    return $name

}

$count = 1
while ($count -lt $NUMBER_OF_ACCOUNTS_TO_CREATE) {
    $fisrtName = generate-random-name
    $lastName = generate-random-name
    $username = $fisrtName + '.' + $lastName
    $password = ConvertTo-SecureString $PASSWORD_FOR_USERS -AsPlainText -Force

    Write-Host "Creating user: $($username)" -BackgroundColor Black -ForegroundColor Cyan
    
    New-AdUser -AccountPassword $password `
               -GivenName $firstName `
               -Surname $lastName `
               -DisplayName $username `
               -Name $username `
               -EmployeeID $username `
               -PasswordNeverExpires $true `
               -Path "ou=_EMPLOYEES,$(([ADSI]`"").distinguishedName)" `
               -Enabled $true
    $count++
}

if you have a weaker computer you or a smaller businees you lower the number
</p>
<br />

<p>

<p>
Step 3 — Verify Accounts in ADUC

Open Active Directory Users and Computers.
Navigate to OU=_EMPLOYEES.
Confirm all generated users are created and active.
</p>
<br />
