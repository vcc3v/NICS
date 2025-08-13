# New Image Compliance Suite

As directed, this is a suite to verify that existing security configurations still function on new images. It is not a comprehensive auditing tool and should not be used to determine the security of a device or image on its own.

All testing should be conducted in the context of a low-privilege or student account, to ensure that the results are as expected.

**Verify deny rules by attempting to run the following executables:**
* `C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe`  
* `C:\Program Files\Internet Explorer\iexplore.exe`  
* `C:\Windows\System32\msconfig.exe`

**Verify path-based deny rules by attempting to run `test.exe` in the following folders (you should be unable to write to certain folders):**
* `C:\Users\%USERNAME%\AppData\Local\Temp`  
* `C:\Windows\SysWOW64\Tasks\Microsoft\Windows\PLA\System`  
* `C:\Windows\System32\spool\drivers\color`  
* `C:\Program Files`  
* `C:\Users\%USERNAME%\Downloads`  
* `C:\Users\Public\`  
* Inside a USB drive/SD card

**Verify executable rules by attempting to run test files with the following extensions (varies by location):**
* `test.exe`  
* `test.msi` (it should run but the installation will fail)  
* `test.ps1`  
* `test.bat`  
* `test.vbs`

**Test shell restriction (varies by device location):**
* Attempt to open and use the terminal  
  * You should be able to open a powershell session  
  * You should NOT be able to open a command prompt session  
* Attempt to open and use PowerShell (this should be allowed)  
  * If you can open PowerShell, check if you can access version 2:  
  * Run: `Powershell.exe -Version 2`  
  * You should receive a .NET compatibility or permissions error  
* Attempt to open and use the command prompt (this should be **blocked**)

**Check if users can configure VPN & Proxy servers (varies by device location):**
* Press ⊞ Win \+ I to open settings  
* Select `Network & Internet` \> `Proxy`  
* Both `Automatic proxy setup` and `Manual proxy setup` should be [greyed out](https://learn.microsoft.com/en-us/answers/questions/4140192/how-can-i-prevent-user-access-to-proxy-settings-on)  
* This functionality should also be disabled in the control panel  
* Move back to `Network & Internet` and navigate to the `VPN` section  
* The button for `VPN Connections` should be greyed out

**Check that users cannot use** [command-line switches](https://peter.sh/experiments/chromium-command-line-switches/) **with Chrome:**
* Press ⊞ Win \+ R  
* Paste in:
```shell
C:\Program Files\Google\Chrome\Application\chrome.exe --no-proxy-server
```
* Try to navigate to a restricted site such as `gambling.com`  
* Lightspeed filter should still prevent you from accessing the webpage

**Verify that users cannot change UAC settings:**
* Open Powershell  
* Run:
```shell
Set-ItemProperty -Path REGISTRY::HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System -Name EnableLUA -Value 0 -Type DWORD -Force
```
* You should get a registry access error  
* Press ⊞ Win \+ R  
* Paste `UserAccountControlSettings` into the runbox or search "User Account Control Settings"  
* You should be unable to launch the application or view the current UAC configuration

**Check that BitLocker is enabled:**
* Open PowerShell and paste in the following command:
```shell
(New-Object -ComObject Shell.Application).NameSpace('C:').Self.ExtendedProperty('System.Volume.BitLockerProtection')
```
* The command should return a 1 indicating that BitLocker is enabled

**Verify that users cannot change registry keys:**
* Press ⊞ Win \+ R  
* Paste in regedit and hit enter  
* You should receive a permissions error  
* Open PowerShell and paste in the following commands:
```shell
Set-Location -Path 'HKLM:\Software\Policies\Microsoft\Windows'
Get-Item -Path 'HKLM:\Software\Policies\Microsoft\Windows' | New-Item -Name 'Windows Search' -Force
```
* You should be unable to access the registry and get a permissions error

**Check if users can view and edit the local security policy:**
* Open Powershell  
* Run `secpol`  
* You should receive a permissions error and be unable to view the security policy

**Verify that users cannot delete system tasks in the task scheduler:**
* Open PowerShell and run `taskschd.msc`  
* Navigate to `Task Scheduler Library > Microsoft > Windows > AppListBackup`  
* Attempt to delete `Backup`  
* You should receive a permissions error after being prompted to delete the task

**Confirm that LAPS is active:**
* Open PowerShell and run `Show-EventLog`  
* Navigate to `Applications and Services > Logs > Microsoft > Windows > LAPS > Operational`  
* You should see several items with the event code `10033` (LAPS is disabled at Sprauge)

**Verify that the BIOS is configured properly:**

**Hewlett-Packard:**

* Check that the TPM device is shown as enabled and available to the operating system  
* Check that `dynamic runtime scanning of boot block` is enabled  
* Check that `sure start secure boot keys protection` is enabled  
* Check that `enhanced HP firmware runtime intrusion prevention and detection` is enabled  
* Check that `secure boot` is enabled  
* Check that `USB storage boot` is disabled

**Dell:**

* Verify that `TPM 2.0 Security` is enabled and available to the operating system  
* Verify that `Absolute Computrace` is enabled  
* Verify that `UEFI Boot Path Security` is set to `Always Except Internal HDD&PXE`  
* Verify that `USB Booting` is not set as a boot option  
* Verify that `UEFI Network Stack` is enabled  
* Verify that `UEFI Bluetooth Stack` is enabled  
* Verify that `Password Bypass` is disabled

**Check that Lightspeed cannot be killed by users:**
* Open PowerShell and run `taskkill /Im LSSASvc.exe`  
* You should be unable to terminate the program  
* Now run `taskkill /Im ClassroomWindows.exe`  
* The program should be killed  
* After thirty seconds run the command again  
* The program should have been restarted during that timeframe and terminated again by your command

**Verify that users cannot be prompted to run elevated processes:**
* Open the start menu and search `powershell`  
* Click on the dropdown and select `run as administrator`  
* You should receive an applocker error  
* Open powershell and paste in:
```shell
Start-Process PowerShell -Verb RunAs
```
* You should receive a group policy error

**Check that the content of other users cannot be accessed:**
* Log out of the current account you are using for this test suite  
* Login with another test account.  
* Navigate to `C:\\Users` and attempt to access the homefolder of the first user  
* You should be unable to access the folder and receive a permissions error

**Verify that users cannot run scripts from the startup folder:**
* Press ⊞ Win \+ R  
* Paste in `Shell:Startup` and hit enter  
* Copy in `test.ps1`  
* The startup script should not launch on restart

**Check that forfiles cannot be used to run executables:**
* Copy `test.bat` to the Downloads folder  
* Open PowerShell and run the following commands:
```shell
forfiles /p "C:\Users\$env:USERNAME\Downloads" /m "test.ps1" /c "cmd /c @path"
```
* The command should fail and the powershell script should not run

**Check that users cannot bypass path rules:**
* Open PowerShell and run:
```shell
New-Item -ItemType SymbolicLink -Path "$env:USERPROFILE\Downloads" -Name "edge.exe" -Target "C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe"
```
* The command should fail and you should receive a permissions error.

**Verify that** `winget` **cannot be used to manage applications:**
* Open Powershell and run: `winget list`  
* Accept the terms and allow the utility to install  
* Now run: `winget install Mozilla.Firefox`  
* You should receive an applocker error after it finishes downloading the package. The installation should fail.  
* Run: `winget uninstall Google.Chrome`  
* The command should fail with error code 1625

**Check that** `mshta.exe` **cannot run HTA files:**
* Press ⊞ Win \+ R and paste in
```shell
mshta.exe vbscript:Execute("msgbox(""This should not execute.""):close")
```
* You should receive an applocker error

**Check that** `certutil.exe` **cannot be used to bypass the filter:**
* Open Powershell and run
```shell
certutil.exe -urlcache -split -f "https://www.google.com/robots.txt" C:\Users\%USERNAME%\Downloads\test.txt  
```
* You should receive a permissions error

**Verify that users cannot access the Microsoft Store:**
* Press ⊞ Win \+ R and paste in: ms-windows-store:  
* Keep the application open for twenty seconds  
* You should receive the error “this application has been restricted by your administrator” and be unable to access the stored content 

