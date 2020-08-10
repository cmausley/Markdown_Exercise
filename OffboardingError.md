# How to fix Offboarding Error: Target user 'xxx' already has a primary mailbox 
Monday, July 20, 2020
2:04 PM

Reference: [Offboarding Error: Target user 'nnnUser' already has a primary mailbox](https://microsoft.sharepoint.com/_forms/default.aspx)
 
**To fix for a single Users**: Do as the reference link writes.
 
**To fix for a large group of Users**:
 
`Example for carentan machine`
1. **Connect the onprem machine**
2. **Get onprem mailbox**
Open a Powershell window as Admin and run:
cd C:\Users\exo\Desktop
. .\dotsource-exchangeshell.ps1 -onpremsession 
$gmb = get-mailbox -Resultsize Unlimited | ?{$_.Name -match 'ctantest'} 
$onpremMB = $gmb | ?{$_.Name -match 'ctantest'} | Select-Object -Property @{Name="onpremMB";Expression = {$_.Name}} 
$onpremMB | export-clixml CTAN_onpremMB_0720.xml 
 
3. **Get cloud mailbox and find the dupe**
Open another Powershell window as Admin and run:
cd C:\Users\exo\Desktop
. .\dotsource-exchangeshell.ps1 -onlinesession
$gmb = get-mailbox -Resultsize Unlimited | ?{$_.Name -match 'ctantest'} 
$onpremMB = import-clixml CTAN_onpremMB_0720.xml 
$dupes = $gmb | ?{$_.Name -in $onpremMB.onpremMB} 
$dupes | Select-Object -Property @{Name="dupeMB";Expression = {$_.Name}} | export-clixml CTAN_dupeMB_0720.xml 
 
for ($i=0; $i -lt $dupes.Count; $i++)
{
$upn = $dupes[$i].Name + "@carentancasino.com"
Remove-MsolUser -UserPrincipalName $upn -Force:$true
Remove-MsolUser -UserPrincipalName $upn -RemoveFromRecycleBin -Force:$true
}
Start-ADSyncSyncCycle
 
 
4. **Note: It will take some time, say 5-10 minutes.**
You can check by running:
$gmb = get-mailbox -Resultsize Unlimited | ?{$_.Name -match 'ctantest'} 
$dupes = $gmb | ?{$_.Name -in $onpremMB.onpremMB} 
$dupes.Count
This should be 0.
If it's not gone by then, run the Sync call again, wait, check. Or run the remove-msoluser again.
