# How to Fix Offboarding Error: *Target user 'xxx' already has a primary mailbox* 
Monday, July 20, 2020
2:04 PM

Reference: [Offboarding Error: Target user 'nnnUser' already has a primary mailbox](https://microsoft.sharepoint.com/_forms/default.aspx)
 
**To fix for a single user**: Follow instructions in reference link above.
 
**To fix for a large group of users**:
 *Example for carentan machine*
1. **Connect the on premises machine**
2. **Retrieve on premises mailbox**: Open a Powershell window as Admin and run the following:

`cd C:\Users\exo\Desktop`

`. .\dotsource-exchangeshell.ps1 -onpremsession`

`$gmb = get-mailbox -Resultsize Unlimited | ?{$_.Name -match 'ctantest'}`

`$onpremMB = $gmb | ?{$_.Name -match 'ctantest'} | Select-Object -Property` 

`@{Name="onpremMB";Expression = {$_.Name}}` 

`$onpremMB | export-clixml CTAN_onpremMB_0720.xml`
 
3. **Retrieve cloud mailbox and find the duplicate**
Open another Powershell window as Admin and run:

`cd C:\Users\exo\Desktop`

`. .\dotsource-exchangeshell.ps1 -onlinesession`

`$gmb = get-mailbox -Resultsize Unlimited | ?{$_.Name -match 'ctantest'}`

`$onpremMB = import-clixml CTAN_onpremMB_0720.xml`

`$dupes = $gmb | ?{$_.Name -in $onpremMB.onpremMB}`

`$dupes | Select-Object -Property @{Name="dupeMB";Expression = {$_.Name}} | export-clixml`

`CTAN_dupeMB_0720.xml`

for ($i=0; $i -lt $dupes.Count; $i++) {
$upn = $dupes[$i].Name + "@carentancasino.com"
Remove-MsolUser -UserPrincipalName $upn -Force:$true
Remove-MsolUser -UserPrincipalName $upn -RemoveFromRecycleBin -Force:$true }Start-ADSyncSyncCycle
 
 
4. **Note: Execution may take as long as 5-10 minutes.**
Check if the execution has run properly by completing the following steps:

`$gmb = get-mailbox -Resultsize Unlimited | ?{$_.Name -match 'ctantest'}`

`$dupes = $gmb | ?{$_.Name -in $onpremMB.onpremMB}`

`$dupes.Count`

The result should equal 0.

If the duplicate has not yet erased, try the following steps:
* Run the Sync call again. Wait, and check for success.
* Run the Remove-MsolUser again.
