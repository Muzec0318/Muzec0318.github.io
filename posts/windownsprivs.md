---
layout: default
title : muzec - Windows Privesc
---

During a penetration test or a CTF, you will often find that obtaining an initial foothold on the target system is relatively easy. Probably not easy at times but man you have the fun and learn something new i know most of us always have issue in windows privilege escalation but worry no more am here to cover some part to make it easy i will keep updating the post to fit in well if i got any new tricks.

Windows systems have different user privilege levels. Accounts can belong to regular users, who would only have enough privileges to log into the system. Some user levels you will most commonly see are listed below: 

1. `Administrator (local):` This is the user with the most privileges.
2. `Standard (local):`  These users can access the computer but can only perform limited tasks. Typically these users can not make permanent or essential changes to the system.
3. `Guest:`  This account gives access to the system but is not defined as a user.
4. `Standard (domain):` Active Directory allows organizations to manage user accounts. A standard domain account may have local administrator privileges.
5. `Administrator (domain):`  Could be considered as the most privileged user. It can edit, create, and delete other users throughout the organization's domain.

### WINDOWS PRIVILEGE ESCALATION METHODOLOGY

1. Enumerate the current user's privileges
2. If the antivirus software allows it, run an automated enumeration script such as winPEAS or PowerUp.ps1
3. If the initial enumeration and scripts do not uncover an obvious strategy, try a different approach


Automated scripts, manually enumerating the target system or a hybrid approach, whichever road you take, carefully read the output you see on the screen. The solution to your privilege escalation problem will often be printed there; you need to see it. 

### User Enumeration:-

Other users that can access the target system can reveal interesting information. A user account named “Administrator” can allow you to gain higher privileges, or an account called “test” can have a default or easy to guess password. Listing all users present on the system and looking at how they are configured can provide interesting information.

The following commands will help us enumerate users and their privileges on the target system.

Current user’s privileges:-

```
whoami /priv
```

List users:-

```
net users
```

List details of a user:-

```
net user username (e.g. net user Administrator)
```

Other users logged in simultaneously:-

```
qwinsta (the query session command can be used the same way)
```

User groups defined on the system:-

```
net localgroup
```

List members of a specific group:

```
net localgroup groupname (e.g. net localgroup Administrators)
```

### Collecting System Information:-

The `systeminfo`  command will return an overview of the target system. On some targets, the amount of data returned by this command can be overwhelming, so you can always grep the output as seen below:

```
systeminfo | findstr /B /C:"OS Name" /C:"OS Version"
```

In a corporate environment, the computer name can also provide some idea about what the system is used for or who the user is. The `hostname` command can be used for this purpose. Please remember that if you have proceeded according to a proper penetration testing methodology, you probably know the `hostname` at this stage.


### Searching Files:-

Configuration files of software installed on the target system can sometimes provide us with cleartext passwords. On the other hand, some computer users have the unsafe habit of creating and using files to remember their passwords (e.g. passwords.txt). Finding these files can shorten your path to administrative rights or even easy access to other systems and software on the target network.

The `findstr` command can be used to find such files in a format similar to the one given below:-


```
findstr /si password *.txt
```

Command breakdown:

```
findstr: Searches for patterns of text in files.

/si: Searches the current directory and all subdirectories (s), ignores upper case / lower case differences (i)

password: The command will search for the string “password” in files

*.txt: The search will cover files that have a .txt extension
```


The string and file extension can be changed according to your needs and the target environment, but “.txt”, “.xml”, “.ini”, “*.config”, and “.xls” are usually a good place to start.

### Patch Level

Microsoft regularly releases updates and patches for Windows systems. A missing critical patch on the target system can be an easily exploitable ticket to privilege escalation. The command below can be used to list updates installed on the target system.

```
wmic qfe get Caption,Description,HotFixID,InstalledOn
```

WMIC is a command-line tool on Windows that provides an interface for Windows Management Instrumentation (WMI). WMI is used for management operations on Windows and is a powerful tool worth knowing. WMIC can provide more information than just installed patches. For example, it can be used to look for unquoted service path vulnerabilities we will see in later tasks. WMIC is deprecated in Windows 10, version 21H1 and the 21H1 semi-annual channel release of Windows Server. For newer Windows versions you will need to use the WMI PowerShell cmdlet.

### Network Connections:-

According to the most widely accepted methodologies, by this stage of the penetration testing process, you should already have conducted a comprehensive scan on the target system. In some cases, we see that some services run locally on a system and can only be accessible locally. System Administrators that lack basic cyber security knowledge tend to be laxer when setting services that are only accessible over the system (e.g. only responding to requests sent to 127.0.0.1). As we have access to the target system, such services can provide a ticket to a higher privileged user.

The netstat command can be used to list all listening ports on the target system. The netstat -ano command will return an output similar to the one listed below:

```
netstat -ano
```

The command above can be broken down as follows;-

```
-a: Displays all active connections and listening ports on the target system.
-n: Prevents name resolution. IP Addresses and ports are displayed with numbers instead of attempting to resolves names using DNS.
-o: Displays the process ID using each listed connection. 
```

Any port listed as “LISTENING” that was not discovered with the external port scan can present a potential local service.


If you uncover such a service, you can try port forwarding to connect and potentially exploit it. The port forwarding process will allow tunnelling your connection over the target system, allowing you to access ports and services that are unreachable from outside the target system. 

### Scheduled Tasks:-

Some tasks may be scheduled to run at predefined times. If they run with a privileged account (e.g. the System Administrator account) and the executable they run can be modified by the current user you have, an easy path for privilege escalation can be available.

The schtasks command can be used to query scheduled tasks.-

```
schtasks /query /fo LIST /v
```

### Drivers:-

Drivers are additional software installed to allow the operating system to interact with an external device. Printers, web cameras, keyboards, and even USB memory sticks can need drivers to run. While operating system updates are usually made relatively regularly, drivers may not be updated as frequently. Listing available drivers on the target system can also present a privilege escalation vector.


```
driverquery
```

The command will list drivers installed on the target system. You will need to do some online research about the drivers listed and see if any presents a potential privilege escalation vulnerability.

### Antivirus:-

While you will seldom face an antivirus in CTF events, a real-world penetration testing engagement will often require you to deal with some form of antivirus. Various reasons will cause an antivirus to miss your shell access without you trying to evade it. For example, the antivirus software will not detect your presence if you have accessed the target system without using a trojan (e.g. using credentials and connect over RDP). However, to reach a higher privilege level, you may need to run scripts or other tools on the target system. It is, therefore, good practice to check if any antivirus is present.


Typically, you can take two approaches: looking for the antivirus specifically or listing all running services and checking which ones may belong to antivirus software.


The first approach may require some research beforehand to learn more about service names used by the antivirus software. For example, the default antivirus installed on Windows systems, Windows Defender’s service name is windefend. The query below will search for a service named “windefend” and return its current state.

```
sc query windefend
```

While the second approach will allow you to detect antivirus software without prior knowledge about its service name, the output may be overwhelming.

```
sc queryex type=service
```

### Vulnerable Software:- 

Software installed on the target system can present various privilege escalation opportunities. As with drivers, organizations and users may not update them as often as they update the operating system. You can use the `wmic` tool seen previously to list software installed on the target system and its versions. The command below will dump information it can gather on installed software.


```
wmic product
```

This output is not easy to read, and depending on the screen size over which you have access to the target system; it can seem impossible to find anything useful. You could filter the output to obtain a cleaner output with the command below.

```
wmic product get name,version,vendor
```

Be careful; due to some backward compatibility issues (e.g. software written for 32 bits systems running on 64 bits), the `wmic product` command may not return all installed programs.  Therefore, It is worth checking running services using the command below to have a better understanding of the target system.

```
wmic service list brief
```

As the output of this command can be overwhelming, you can grep the output for running services by adding a findstr command as shown below.

```
wmic service list brief | findstr  "Running"
```

If you need more information on any service, you can simply use the `sc qc` command as seen below.

![image](https://user-images.githubusercontent.com/69868171/153878798-cf4bc2aa-03a5-443c-91ca-e97d6b9c4ba3.png)


```
sc qc RemoteMouseService
```

At this point, you have a few options to find any possible privilege escalation exploit that can be used against software installed on the target system.

    Searchsploit
    Metasploit
    Exploit-DB
    Github
    Google

Be careful using exploit code that is not verified or is part of the Metasploit framework, as it can contain malicious code that could affect your attacking system. Be sure you understand the exploit code well, go over any obfuscated parts, and have a good understanding of all commands the exploit code will attempt to run.

### Below Are A Few Tools Commonly Used To Identify Privilege Escalation Vectors:-

1. WinPEAS:- WinPEAS is a script developed to enumerate the target system to uncover privilege escalation paths. You can find more information about winPEAS and download either the precompiled executable or a .bat script. Please note, Windows Defender detects and disables winPEAS. WinPEAS will run commands similar to the ones listed in the previous task and print their output. The output from winPEAS can be lengthy and sometimes difficult to read. This is why it would be good practice to always redirect the output to a file, as shown below: 

![image](https://user-images.githubusercontent.com/69868171/153871059-4bee2d24-0777-403d-a669-5e78880a50e2.png)

```
winpeas.exe > outputfile.txt
```

2. PowerUp:- PowerUp is a PowerShell script that searches common privilege escalation on the target system. You can run it with the `Invoke-AllChecks` option that will perform all possible checks on the target system or use it to conduct specific checks (e.g. the `Get-UnquotedService` option to only look for potential unquoted service path vulnerabilities).

![image](https://user-images.githubusercontent.com/69868171/153871367-eedf135c-94e0-40fd-8a3a-beccb60e91ca.png)

`Reminder:` To run PowerUp on the target system, you may need to bypass the execution policy restrictions. To achieve this, you can launch PowerShell using the command below.

![image](https://user-images.githubusercontent.com/69868171/153871500-dd402738-6e5f-4766-a220-d87b4be5fe44.png)


### Windows Exploit Suggester:-

Some exploit suggesting scripts (e.g. winPEAS) will require you to upload them to the target system and run them there. This may cause antivirus software to detect and delete them. To avoid making unnecessary noise that can attract attention, you may prefer to use Windows Exploit Suggester, which will run on your attacking machine.

![image](https://user-images.githubusercontent.com/69868171/153873560-3881aa43-0348-42d9-8e88-8445aec52315.png)

Once installed, and before you use it, type the `windows-exploit-suggester.py –update` command to update the database. The script will refer to the database it creates to check for missing patches that can result in a vulnerability you can use to elevate your privileges on the target system.

To use the script, you will need to run the `systeminfo` command on the target system. Do not forget to direct the output to a `.txt` file you will need to move to your attacking machine.

Once this is done, windows-exploit-suggester.py can be run as follows;

```
windows-exploit-suggester.py --database 2021-09-21-mssb.xls --systeminfo sysinfo_output.txt
```

A newer version of Windows Exploit Suggester is available [here](https://github.com/bitsadmin/wesng). Depending on the version of the target system, using the newer version could be more efficient.

![image](https://user-images.githubusercontent.com/69868171/153875493-768817c7-78dd-4bee-a9bc-41cd86b7b62e.png)

### Metasploit:-

If you already have a Meterpreter shell on the target system, you can use the `multi/recon/local_exploit_suggester` module to list vulnerabilities that may affect the target system and allow you to elevate your privileges on the target system. 
