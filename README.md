<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1> Automatic Bulk users creation on active directory using power shell and establishing group policy (Azure)</h1>
This lab demonstrates how to automate bulk user creation in Active Directory using PowerShell and apply account lockout policies through Group Policy. It also covers unlocking and disabling accounts, testing security responses, and reviewing event logs for forensic visibility.<br />



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
- step 5 - making a password lock out
- Step 6 — Unlocking the Account
- Step 7 — Enabling and Disabling Accounts
- Step 8 — Observing Security Logs

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
<img width="1661" height="995" alt="Screenshot (123)" src="https://github.com/user-attachments/assets/60281757-1137-4294-bc9c-479036475a99" />

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
<img width="1661" height="987" alt="Screenshot (124)" src="https://github.com/user-attachments/assets/bbee980a-7c95-4a43-8201-2d994f045db3" />

Open Active Directory Users and Computers.
Navigate to OU=_EMPLOYEES.
Confirm all generated users are created and active.
</p>
<br />

Step 4 — Test Login with a Standard User

On a different RDP session, log in to Client-1 using one of the generated accounts:

Example: mydomain\leko.romi

Password: Password1

Confirm successful sign-in with no administrative privileges.

 RDP access for Domain Users is working.
Large-scale user creation was successful.

       Group policy testing 
-----------------------------------------

<img width="1667" height="990" alt="Screenshot (125)" src="https://github.com/user-attachments/assets/f8b6b34f-e66c-4a2b-a64b-4affa36a8d14" />


#step 5 - making a password lock out
open active dicterory manger head to Computer Configuration > Policies > Windows Settings > Security Settings > Account Policies > Account Lockout Policy.
make it so it locks after 5 incorcet attempts and they have to wait 10 mintues in between every incorrect attempt for the timer to reset
-use command prompt gpupdate / force to up date group policy
- then fail the log in 6 times and obeserve

#Step 6 — Unlocking the Account
<img width="1684" height="1056" alt="Screenshot (126)" src="https://github.com/user-attachments/assets/8a140c03-e819-466e-b46a-fc02c454c543" />

In Active Directory Users and Computers (ADUC), locate the locked account.

Unlock the account and reset the password.

Attempt to log in again with the new password to verify successful recovery.

#Step 7 — Enabling and Disabling Accounts
<img width="1666" height="981" alt="Screenshot (127)" src="https://github.com/user-attachments/assets/13233d89-8b44-4a03-9c04-46d59ed47d60" />

Disable the same account in ADUC.

Attempt to log in — observe the “account disabled” error.

Re-enable the account.

Attempt to log in again and confirm access is restored.

#Step 8 — Observing Security Logs
<img width="1789" height="1033" alt="Screenshot (128)" src="https://github.com/user-attachments/assets/b5b47b14-e9a0-4069-adeb-35f922ff15b1" />

On the Domain Controller, open Event Viewer → Windows Logs → Security.

Check for event(account locked out).

Check for event failed logons).

On the Client machine, open Event Viewer.

Review event to trace the failed login attempts.

  What we leanred
   --------
 Identity lifecycle ops: bulk user provisioning with PowerShell and OU targeting.

Access control hardening: enforced Account Lockout Policy via GPO, validated on a domain-joined client.

Incident response drill: triaged account lockout, unlocked & reset credentials, and restored access cleanly.

 forensics: correlated DC and Client Security logs to identify source host and failure reasons.

Operational hygiene: tested disable/enable flows and confirmed end-user impact/messages. 
