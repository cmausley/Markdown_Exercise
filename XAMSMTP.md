# XAM SMTP Internet - Enterprise (EXO + EOP)
Monday, October 10, 2016
10:28 AM
 
## Monitor Goals
Monitor the path of mail transmission through internet submission of mail to EOP FDs through EXO HUBs. This Probe runs/alerts on the following Scopes:
1. **EXO Dag scoped**: This alerts on a Dag level when availability goes down.
2. **FOREST level RED ALERT**: A System wide Incident/Red Alert is issued when across a forest availability dips for any reason.
3. **EOP FOREST scoped**: This alerts on a EOP Forest level instead of conventional EXO focused scope. This will include issues faced until EOP FD successfully proxies the incoming mail to EXO HUB. 
 
## What does the probe do?

![](./assets/mailflow-image.jpg)

Each probe execution follows these steps:
1. Resolve MX of monitoring tenant.
2. Connect to monitoring tenant's MX host (this should resolve to an EOP FD).
3. Negotiate an SSL connection.
4. Submit email to monitoring mailbox.
 
## What triggers the alert?
This monitor targets a specific Dag/Exo forest/Eop Forest. An alert is raised when the percentage of successful probes in the last hour drops below 97.5% (monitoring profile specific threshold).
 
## Possible root causes
1. GLS issue: Cannot attribute mail because GLS is not accessible or monitoring tenant cannot not be found in GLS.
2. Networking issue: Cannot resolve tenant's MX (DNS++ issue), VIP not configured correctly, connectivity issues between Azure instance, EOP FD, EXO Café and/or EXO hub, etc.
3. Certificate issues: wrong or expired certificate, monitoring mailbox not provisioned/configured correctly.
4. Transport issues: EOP FD, EXO Café or hub are too busy or in a bad state (i.e. crashing), or code bug in any of these transport components.
 
## Link to OSP
[EXO Dag Scoped (XAM SMTP - Internet)](https://o365pulse.office.net/enterprisedashboard?probe=SMTP%20Internet%20-%20Enterprise&environment=Prod&scope=*.*.*)
 
[EOP Forest Scoped (XAM SMTP EOP - Internet)](https://o365pulse.office.net/enterprisedashboard?probe=SMTP%20Internet%20-%20Enterprise%20EOP&environment=Prod&scope=*.*.*)
 
## Diagnose and Recovery
### Step 1 - Determine prominent failure reason (Error Type) and Smtp error responses
* Check the alert mail table at the bottom to find the major cause of failure (prominent error type) or OSP Page link in email
* Identify the components involved in the failure e.g. 
    * UnableToConnect - Azure and EOP FD
    * ReplicationFailure - EXO Hub
    * ProxyDnsFailure - EOP FD, EXO Café
    * ProbeTimeout - Check if majority of last response is "250 Recipient OK" in OSP - EOP FD, EXO Hub
        * Use "Get-NetworkConfig.ps1 -Server CY1USG02FT013 -PortConnectivity (2001:489a:2200:408::3):25 -ShowConnectionResponse" to see if the connection can be established and if we can read the banner successfully.
  `Port 25 Connectivity Status: Passed uses 12.803 ms Can't get remote side answer.`
 
* **Prominent failure reasons can be found in the body of the alert email or in OSP.** See Outside-In probe prominent failure reason for more information.
 
#### Mitigation steps and description of each failure reason / error type / smtp response
Once you've determined the failure reason / error type, see the following pages for added information on the error type.
[Common Outside In SMTP Probe Failures]
[Understand Outside-In Class of Failures]
 
### Step 2 - Retreive probe smtp sessions from protocol logs
* After identifying the major area of failure - pick one error (from OSP) to get the protocol logs
    * Use Ehlo domain used for the probe
    * Pick up time of the probe 
    * Get Protocol logs
* Use On-call script to diagnose the logs 
* Help for using the script: [Get a XAM Probe's Protocol logs]
 
 
## Diagnose Eop Only issues
 
Smtp Internet - Enterprise workflow starts an SMTP conversation with EOP Fd in a sequence. EOP FD sets up a proxy connection to EXO Café and proxies the SMTP commands on BDAT. 
Issues Pre BDAT command are specifically EOP Only issues. 
Issues Post BDAT command can be differentiated between Eop, and Exo related
* EOP related issues - Setting up a proxy (Error Type PROXYDNSFAILURE) 
* EXO related issues - Hub server busy/replication failure
* To further identify the EOP/EXO ONLY issues take the following steps
    * Go to OSP
    * Look at scopes of the failures at EOP Forest level and check for availability at Dag level on the Right column of OSP health page
    * If errors are concentrated on a Dag - Check availability of SMTP Internet - Enterprise (Dag). 
    * If errors are distributed over multiple dags - then verify the prominent error type happens before BDAT command Or it is a proxy connection failure.