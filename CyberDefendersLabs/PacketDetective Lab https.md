## [PacketDetective Lab](https://cyberdefenders.org/blueteam-ctf-challenges/packetdetective/)

### Description

`In September 2020, your SOC detected suspicious activity from a user device, flagged by unusual SMB protocol usage. Initial analysis indicates a possible compromise of a privileged account and remote access tool usage by an attacker.`

`Your task is to examine network traffic in the provided PCAP files to identify key indicators of compromise (IOCs) and gain insights into the attacker’s methods, persistence tactics, and goals. Construct a timeline to better understand the progression of the attack by addressing the following questions.`

**Category:** Network Forensics 
**Tactics:** Execution, Defense Evasion, Command and control `used by the threat actor.`
**Tools:** Wireshark, Google  
**Author:** CyberDefender  


<h2> Traffic 1 data </h2> 
(`Traffic data generated from wireshark`)

## Question 1
 - The attacker’s activity showed extensive SMB protocol usage, indicating a potential pattern of significant data transfer or file access. 
``What is the total number of bytes of the SMB protocol?``
 
> Approach:
 - Located SMB protocol data from wireshark interface `statistics > protocol hierachy`
 
 **result**
 <img src='../lab_Snapshots/'1.SMB_bytes.png' />  
 ![IP Statistics](lab_Snapshots/1.SMB_bytes.png) 
 
 
## Question 2
 - Authentication through SMB was a critical step in gaining access to the targeted system. Identifying the username used for this authentication will help determine if a privileged account was compromised. 
 ``Which username was utilized for authentication via SMB?``
 
> Approach:
 - Since the attacker has used SMB to setup request on the server and was able to bypass the authN protocols to find the username used: filtered through the packets that contains NTLM AuthN_signatures - `ntlmssp.auth.username`
 
 **result** 
 <img src='../lab_Snapshots/2.UsernameAuthN_via_SMB.png' /> 
 Answer: Administrator
`Authentication method used was NTLM New Technology LAN Manager, is a set of Microsoft protocols that authenticate users and computers on a network`

## Question 3
 - During the attack, the adversary accessed certain files. Identifying which files were accessed can reveal the attacker's intent. 
 ``What is the name of the file that was opened by the attacker?``
 
> Approach:
 - Navigated through `File > Export Objects > SMB.` to know which file was opened by the threat actor.
 
 **result** 
  ![Eventlog](lab_Snapshots/3.File_accessedby_threatactor.png) 
 
 Answer: eventlog

## Question 4
 - Clearing event logs is a common tactic to hide malicious actions and evade detection. Pinpointing the timestamp of this action is essential for building a timeline of the attacker’s behavior. 
 ``What is the timestamp of the attempt to clear the event log? (24-hour UTC format) ``
 
> Approach:
 - To pinpoint when an attacker attempted to clear the event log, I discovered i needed to identify a specific Remote Procedure Call (RPC) operation in the network traffic. In Windows, RPC manages functions like log handling, and each action is identified by an operation number (opnum).
 
## Key DCE/RPC Operations for Analysis:
 - Opnum 0 - ClearEventLog
 - Opnum 7 - ReadEventLog 
 - Opnum 1 - BackupEventLog
 - Opnum 4 and 5 - GetNumberOfEventLogRecords and GetOldestEventLogRecord
 
Which i used `dcerpc.opnum == 0 ` to identify.
DCERPC - Distributed computing environment/Remote Procedure Calls
 
 **result** 
  ![Eventlog](lab_Snapshots/4. Timestamp.png) 
 
 Answer: 2020-09-23 16:50:16
 
## Question 5
 - The attacker used "named pipes" for communication, suggesting they may have utilized Remote Procedure Calls (RPC) for lateral movement across the network. RPC allows one program to request services from another remotely, which could grant the attacker unauthorized access or control. 
 ``What is the name of the service that communicated using this named pipe?``

Revisited: _Named Pipes are a method of inter-process communication (IPC) that allows different processes (programs or services) on the same machine, or across a network, to communicate with each other in a secure, structured way._
 
> Approach:
 - Navigated through `File > Export Objects > SMB.` to know which file was opened by the threat actor.
 
 **result** 
  ![Eventlog](lab_Snapshots/3.File_accessedby_threatactor.png) 
 
