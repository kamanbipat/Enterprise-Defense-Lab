# Enterprise Defense Lab: Identity, SIEM, & Disaster Recovery
## Debrief
This is a sample cybersecurity lab demonstrating a defensive-first approach to simulate strengthing corporate infrastructure. This lab covers the deployment of a Windows Domain environment, secured by Zero Trust policies, monitored by a SIEM, and protected by an enterprise-grade backup solution. The goal of this lab is to move beyond a simple network and transition it into a secured enterprise environment. By integrating Active Directory with modern cloud-based patching and local security policies, this lab replicates the layered defense-in-depth strategies used in professional SOC and SysAdmin environments.

## Table of Contents

Architecture & Topology\
Phase 1: Domain Infrastructure\
Phase 2: Vulnerability Management (Action1)\
Phase 3: Zero-Trust Enforcement (AppLocker)\
Phase 4: SIEM & Monitoring (Wazuh)\
Phase 5: Backup & Disaster Recovery (Veeam)\
Conclusion & Portfolio Summary

## Architecture, Topology, and Tools
This lab is isolated on a custom virtual network to simulate a production LAN.\
We will be utilizing 4 virtual machines for this lab.\
If you would like to follow along, now is the time to spin up VMs of your own using Hyper-V or VMware:\
(My Domain Controller and Workstation VMs are prefixed with my initials, you can omit or use your own initials)

1. Domain Controller (KB-DC01): Windows Server 2022 (DNS, AD DS, DHCP)
2. Standard Workstation (KB-WS01): Windows Pro (Domain-joined)
3. Security Manager (Wazuh-Srv): Wazuh OVA (Linux-based SOC Analytics)
4. Backup Server (Veeam-Srv): Windows Server (Veeam B&R Community Edition)

We will also be utilizing a handful of programs and tools:
1. Action1, for vulnerability management, patching, and deployment
2. AppLocker, for application allowlisting and Zero Trust ideology
3. Wazuh, for SIEM and compliance
4. Veeam, for backup and restores

## Phase 1: Domain Infrastructure
#### Setting up and initializing the lab environment. (This assumes you have already built out the Domain Controller and Workstation VMs)

1. Network Isolation: Create a custom Virtual Network (Host-only or NAT) with the subnet 10.0.0.0/24.
    - Open the settings to your hypervisor.
    - Create a new "host-only" or "NAT" network.
    - Set the subnet IP to 10.0.0.0 and the subnet mask to 255.255.255.0.
    - Disable the hypervisor's built-in DHCP to ensure you have full control over IP addressing.
      ![](https://github.com/kamanbipat/Enterprise-Defense-Lab/blob/main/Home%20Lab%201/Screenshot%202026-05-13%20204301.png?raw=true)
2. Static Configuration: Harden the Domain Controller by assigning a static IP of 10.0.0.1 and configuring DNS to point to itself.
    - On the Domain Controller, we will go to ```Settings > Network Connections > Properties > IPv4```.
    - Set the IP address to 10.0.0.1, the subnet mask to 255.255.255.0, and the preferred DNS to 127.0.0.1.
      ![](https://github.com/kamanbipat/Enterprise-Defense-Lab/blob/main/Home%20Lab%201/Screenshot%202026-05-13%20205242.png?raw=true)
3. Identity Setup: Install Active Directory Domain Services (AD DS). Promote the server to a Domain Controller for the upcoming lab.internal forest.
    - Search for and open Server Manager. Click "Add roles and features".
    - Follow the default prompts, but on Server Roles, select "Active Directory Domain Services" to install.
    - Click the flag notification and select "Promote this server to a domain controller".
    - Select "Add a new forest" and name it. I used "lab.internal".
    - Create a secure DSRM password, and reboot when finished.
      ![](https://github.com/kamanbipat/Enterprise-Defense-Lab/blob/main/Home%20Lab%201/Screenshot%202026-05-13%20205401.png?raw=true)
      ![](https://github.com/kamanbipat/Enterprise-Defense-Lab/blob/main/Home%20Lab%201/Screenshot%202026-05-13%20205512.png?raw=true)
      ![](https://github.com/kamanbipat/Enterprise-Defense-Lab/blob/main/Home%20Lab%201/Screenshot%202026-05-13%20210055.png?raw=true)
      ![](https://github.com/kamanbipat/Enterprise-Defense-Lab/blob/main/Home%20Lab%201/Screenshot%202026-05-13%20210259.png?raw=true)
4. OU Design: Implement a logical Organizational Unit (OU). All testing users and workstations are moved here to ensure targeted GPO application.
    - Now search for and open Actve Directory Users and Computers.
    - Right-click on lab.internal, create a new Organizational Unit, and name it. I used CORP.
    - Right-click on CORP OU, and create a new user. I used kbipat.
      ![](https://github.com/kamanbipat/Enterprise-Defense-Lab/blob/main/Home%20Lab%201/Screenshot%202026-05-13%20210758.png?raw=true)
      ![](https://github.com/kamanbipat/Enterprise-Defense-Lab/blob/main/Home%20Lab%201/Screenshot%202026-05-13%20210810.png?raw=true)
5. Domain Join: Configure the Workstation to use the DC as its primary DNS and successfully join it to the domain.
    - Set the Workstation IPv4 to 10.0.0.1.
    - Go to System Settings > About > Rename this PC.
    - Click "Change", select "Domain", enter "lab.internal", and provide the Domain Controller admin credentials.
    - After the Workstation joins the domain, go back in Active Directory Users and Computers on the Domain Controller, and drag the Workstations computer object from the Computers container into the CORP OU.
      ![](https://github.com/kamanbipat/Enterprise-Defense-Lab/blob/main/Home%20Lab%201/Screenshot%202026-05-13%20211149.png?raw=true)
      ![](https://github.com/kamanbipat/Enterprise-Defense-Lab/blob/main/Home%20Lab%201/Screenshot%202026-05-13%20211248.png?raw=true)

## Phase 2: Vulnerability Management
#### Utilize Action1 for centralized endpoint health and automated patch management.

1. Agent Deployment: Deploy the Action1 agent to both endpoints via Administrative sessions.
    - Log into your Action1 account on both the Domain Controller and Workstation endpoints.
    - Navigate to Endpoints > Add Endpoint, and download the .msi installer onto each endpoint.
    - Run the installer on both endpoints.
      ![](https://github.com/kamanbipat/Enterprise-Defense-Lab/blob/main/Home%20Lab%201/Screenshot%202026-05-13%20212946.png?raw=true)
      ![](https://github.com/kamanbipat/Enterprise-Defense-Lab/blob/main/Home%20Lab%201/Screenshot%202026-05-13%20213029.png?raw=true)
      ![](https://github.com/kamanbipat/Enterprise-Defense-Lab/blob/main/Home%20Lab%201/Screenshot%202026-05-13%20213127.png?raw=true)
2. Vulnerability Assessment: Conduct a baseline scan to identify missing Windows KBs and critical third-party software risks.
    - After Action1 has been installed on both endpoints, head back to the Endpoints tab in Action1 on either endpoint.
    - Wait for the agents to check in, typically takes a few minutes. Grab a coffee, tea, or snack in the meantime.
    - Click on the Vulnerabilities tab to see a list of missing security patches and outdated applications. The older .iso file you used for your VMs, the more likely you are you have a greater number.
      ![](https://github.com/kamanbipat/Enterprise-Defense-Lab/blob/main/Home%20Lab%201/Screenshot%202026-05-13%20213233.png?raw=true)
      ![](https://github.com/kamanbipat/Enterprise-Defense-Lab/blob/main/Home%20Lab%201/Screenshot%202026-05-13%20213303.png?raw=true)
      
3. Remediation: Orchestrate a centralized patch cycle from the cloud console. Verify that both the Domain Controller and Workstation reached a 100% "Patched" status.
    - Select all available patches, and click "Start Remediation".
    - Monitor the progress bar or until the dashboard shows 0 Vulnerabilities for both endpoints.
    - For future patches, you can take things a step further by creating a patch schedule, to deploy updates at a specific time!
      ![](https://github.com/kamanbipat/Enterprise-Defense-Lab/blob/main/Home%20Lab%201/Screenshot%202026-05-13%20213326.png?raw=true)

## Phase 3: Zero-Trust Ideology
#### Implementation of AppLocker for a Default Deny posture to prevent unauthorized executions.

1. GPO Configuration: Create a Group Policy Object named AppLocker_Rules linked to the CORP OU.
    - On the Domain Controller, open Group Policy Management.
    - Right-click the CORP OU, then select "Create a GPO in this domain, and link it here"
    - Name it "AppLocker_Rules"
      ![](https://github.com/kamanbipat/Enterprise-Defense-Lab/blob/main/Home%20Lab%201/Screenshot%202026-05-13%20211456.png?raw=true)
2. Service Automation: Configure the Application Identity service to "Automatic" via GPO to ensure the enforcement engine remains active.
    - Right-click the new GPO and select Edit.
    - Follow ```Computer Configuration > Policies > Windows Settings > Security Settings > System Services```.
    - Find "Application Identity", check "Definte this policy setting", and set it to "Automatic".
      ![](https://github.com/kamanbipat/Enterprise-Defense-Lab/blob/main/Home%20Lab%201/Screenshot%202026-05-13%20211648.png?raw=true)
3. Rule Logic: Create Default Executable and Packaged App Rules to allow files within C:\Windows and C:\Program Files and universal apps to ensure UI functionality.
    - Go back one time to Security Settings, and go into ```Application Control Policies > AppLocker```.
    - Right-click Executable Rules and select "Create Default Rules".
    - From AppLocker, navigate into "Packaged App Rules".
    - Right-click the empty right pane and select "Create Default Rules"
      ![](https://github.com/kamanbipat/Enterprise-Defense-Lab/blob/main/Home%20Lab%201/Screenshot%202026-05-13%20211829.png?raw=true)
4. Enforcement: Transition policy from "Audit Only" to "Enforce Rules".
    - Go to the top level AppLocker Folder > Properties
    - Check the configured box for both rule types and set it to "Enforce Rules".
    - Run ```gpupdate /force``` on the Workstation to force update the group policy.
      ![](https://github.com/kamanbipat/Enterprise-Defense-Lab/blob/main/Home%20Lab%201/Screenshot%202026-05-13%20211947.png?raw=true)
      ![](https://github.com/kamanbipat/Enterprise-Defense-Lab/blob/main/Home%20Lab%201/Screenshot%202026-05-13%20212032.png?raw=true)
      ![](https://github.com/kamanbipat/Enterprise-Defense-Lab/blob/main/Home%20Lab%201/Screenshot%202026-05-13%20212130.png?raw=true)
5. Validation: Verify the policy by attempting to run cmd.exe from a user-writable directory (Desktop) as a standard user. Confirm the system successfully blocked execution.
    - Login as your local user for lab.internal, for me it is lab\kbipat.
    - Go to C:\Windows\System32, copy cmd.exe, and paste it to the Desktop.
    - Attempt to run cmd.exe from the desktop - it should be blocked by the policy as we are running it from a prohibited location.
      ![](https://github.com/kamanbipat/Enterprise-Defense-Lab/blob/main/Home%20Lab%201/Screenshot%202026-05-13%20212200.png?raw=true)

## Phase 4: SIEM & Monitoring
#### Implementing a SIEM platform with Wazuh for real-time visibility.

1. Manager Deployment: Import the Wazuh OVA into the hypervisor, assigning 4GB RAM for optimal indexing performance.
    - Download the Wazuh OVA and open it with your hypervisor.
    - Edit settings to assign 2 CPUs and 4GB RAM.
    - Power on and run ```ip add``` to find the Manager's IP address.
      ![](https://github.com/kamanbipat/Enterprise-Defense-Lab/blob/main/Home%20Lab%201/Screenshot%202026-05-14%20154206.png?raw=true)
2. Agent Installation: Deploy Wazuh agents to all Windows endpoints using the dashboard's PowerShell deployment script.
    - Browse to the Manager IP in a web browser.
    - Select ```Deploy new agent > Windows```.
    - Copy the command provided and run it in an Administrator-level Powershell window on both the Domain Controller and Workstation endpoints.
      ![](https://github.com/kamanbipat/Enterprise-Defense-Lab/blob/main/Home%20Lab%201/Screenshot%202026-05-14%20155807.png?raw=true)
      ![](https://github.com/kamanbipat/Enterprise-Defense-Lab/blob/main/Home%20Lab%201/Screenshot%202026-05-14%20160316.png?raw=true)
      
3. AppLocker Event Auditing: Modify the agent ossec.conf file to include the AppLocker event channel: Microsoft-Windows-AppLocker/EXE and DLL.
    - Open ```C:\Program Files (x86)\ossec-agent\ossec.conf``` in Notepad (as Administrator).
    - Insert the following:
      
```
<localfile>
  <location>Microsoft-Windows-AppLocker/EXE and DLL</location>
  <log_format>eventchannel</log_format>
</localfile>
```
    
4. Restart the agent service to begin log ingestion.
    - Run ```Restart-Service -Name wazuh```
      ![](https://github.com/kamanbipat/Enterprise-Defense-Lab/blob/main/Home%20Lab%201/Screenshot%202026-05-14%20165028.png?raw=true)
5. Dashboard Analysis: Trigger "unauthorized" execution attempts on the workstation and verify that Event ID 8004 appeared in the Wazuh security dashboard.
    - In the Wazuh Dashboard, find the Workstation, then click on Threat Hunting > Events.
    - Ensure the block attempt for running cmd.exe on the Desktop from the previous phase appears as a security alert.
      ![](https://github.com/kamanbipat/Enterprise-Defense-Lab/blob/main/Home%20Lab%201/Screenshot%202026-05-14%20170428.png?raw=true)
      ![](https://github.com/kamanbipat/Enterprise-Defense-Lab/blob/main/Home%20Lab%201/Screenshot%202026-05-14%20170454.png?raw=true)

## Phase 5: Backup & Disaster Recovery
#### Ensuring business continuity and data integrity by utilizing Veeam Backup and Recovery.

1. Veeam Installation: Deploy Veeam Backup & Replication Community Edition.
    - Mount the Veeam Backup and Recovery .iso on the Domain Controller.
    - Run setup.exe, and select "Install Veeam Backup & Replication", following the prompts for a standalone installation.
      ![](https://github.com/kamanbipat/Enterprise-Defense-Lab/blob/main/Home%20Lab%201/Screenshot%202026-05-14%20195916.png?raw=true)
2. Repository Setup: Configure a dedicated virtual disk as the primary backup target.
    -  In the Veeam console, go to ```Backup Infrastructure > Backup Repositories```.
    -  Click "Add Repository", select "Direct attached storage", and choose your dedicated backup disk.
    - ![](https://github.com/kamanbipat/Enterprise-Defense-Lab/blob/main/Home%20Lab%201/Screenshot%202026-05-14%20203350.png?raw=true)
3. Workload Backup: Create an "Entire Computer" backup job for the Workstation, enabling Application-Aware Processing for consistent snapshots.
    - Go to ```Home > Jobs > Backup > Virtual Machine```
    - Add the Workstation to the job.
    - Check "Enable application-aware processing" (use domain admin credentials when prompted).
    - Start the job and wait for the "Success" status.
      ![](https://github.com/kamanbipat/Enterprise-Defense-Lab/blob/main/Home%20Lab%201/Screenshot%202026-05-14%20204003.png?raw=true)
4. Recovery Drill: Simulate a failure by deleting a critical system component, then utilize Instant VM Recovery to restore the workstation, verifying a recovery time objective (RTO) of under 5 minutes.
    - On the Workstation, delete something like ```C:\Windows\System32\drivers```.
    - In Veeam, right-click the backup, and select Restore.
    - Choose "Instant Recovery" to power the VM directly from the backup file for a rapid RTO.
      ![](https://github.com/kamanbipat/Enterprise-Defense-Lab/blob/main/Home%20Lab%201/Screenshot%202026-05-14%20204113.png?raw=true)

## Summary
This lab demonstrates a complete security and administrative lifecycle:

- Identify: Asset and Identity management via Active Directory.
- Protect: Proactive hardening via Action1 and AppLocker.
- Detect: Real-time visibility and SOC analytics via Wazuh SIEM.
- Recover: Reliable business continuity via Veeam.

By successfully bridging these technologies, this environment achieves a professional-grade security posture. The foundation is built on Active Directory Domain Services (AD DS). This establishes a centralized authority for identity management, ensuring that users and devices are governed by a single source of truth. By segmenting the network into a dedicated 10.0.0.0/24 subnet, we isolate lab traffic from the host network, simulating a real-world corporate LAN. Using Action1, we implement centralized Patch Management. This focuses on reducing the attack surface by identifying missing Windows updates and third-party software vulnerabilities (like outdated browsers), then remediating them through a single cloud console. Next, instead of relying on traditional antivirus, we implement Windows AppLocker. By moving to a Default Deny posture, the system is configured to inherently distrust any file that is not explicitly authorized. This effectively stops ransomware, unauthorized scripts, and shadow IT from executing in user-writable directories. By deploying Wazuh, we establish a Security Operations Center (SOC) framework. The Wazuh manager acts as the "Security Camera," ingesting logs from the Windows endpoints. This allows for real-time auditing of security events, such as unauthorized execution attempts (Event ID 8004), providing the forensic data needed to investigate potential breaches. Lastly, we utilize Veeam Backup & Replication to ensure that even in the event of a total system failure or a successful ransomware attack, the environment can be restored to a known-clean state in minutes. This validates the "Recovery Time Objective" (RTO) essential for business continuity.

*Developed by Kaman Bipat, 2026.*
