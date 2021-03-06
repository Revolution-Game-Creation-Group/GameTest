# Guest Installation Guide: Windows 7 Enterprise / Ultimate SP1

## Configure host

In an Administrative PowerShell prompt, you must run `winrm s winrm/config/client '@{TrustedHosts="*"}'` to enable connections to the VM.

The Hyper-V host must have a hardware accelerated GPU in order to provide DirectX support to the guest.  We run an ASUS NVIDIA GT710 (x8 PCI-e) GPU in a Dell R710, which works for this purpose.

Windows 7 guests **only** support RemoteFX, which in turn requires the Windows 7 edition to be either Enterprise or Ultimate (only these editions support running as a RemoteFX client).  DDA for GPU passthrough is not available to Windows 7 guests (even if the host supports it).

Please note that without this configured correctly, the guest will be unable to create DirectX devices, which will severely limit your ability to automate the testing of games.

## Set up the baseline image

1. Install Windows 7 Ultimate in Hyper-V using the ISO provided by MSDN.
    - Do not turn on updates.  We will manually install Service Pack 1 to ensure that is the baseline.
    - GameTest uses Hyper-V snapshotting to restore the guest to it's original state after each run.
2. Set the username to be "qa" and the password to be "qa".
3. Once the VM has installed and booted, you will notice there is no network connectivity.
    - You will not be able to install Hyper-V integration services yet as they require Windows 7 SP1.
4. Shutdown the VM.
5. In Hyper-V settings, add a Legacy Network Adapter and a RemoteFX Video Adapter.
6. Boot the VM and login.  You should notice you now have network connectivity.  When prompted, select "Work Network".
7. Open an Administrative PowerShell prompt and run the following:
   ```
   Enable-PSRemoting -Force
   ```
8. Open Windows Update, and change the settings to never check for Windows Updates by default.
    - We want to be very specific about the updates we're installing on this image, so we don't want either automatic download or installation.
9. Click "Check for Updates".
10. You will see that Windows Update itself requires an update; click "Install now" to proceed.
11. Windows Update will automatically restart itself.
12. Once Windows Update finishes checking for updates, click on the "N Important Updates" link.  DO NOT CLICK "Install updates" YET!
13. Untick everything except "Windows 7 Service Pack 1 for x64-based Systems (KB976932)".  Make sure everything else is unticked in both Important and Optional updates.
14. Click Ok.
15. Above the "Install updates" button, it should say "1 important update selected".  If this is the case, click "Install updates".
16. After the update is downloaded and installed, click "Restart now".
17. The machine will automatically reboot.
18. Open Windows Update again, and click "Check for updates".
19. Again click the "Important updates" link and unselect everything other than "Windows 7 Service Pack 1 for x64-based Systems (KB976932)".  Make sure everything else is unticked in both Important and Optional updates.
20. Click Ok.
21. Above the "Install updates" button, it should say "1 important update selected".  If this is the case, click "Install updates".
22. After the update is downloaded and installed, click "Restart now".
23. The machine will automatically reboot and install SP1.
24. Login again, then from the "Action" menu select "Install Integration Services Setup Disk".
25. Start the installation via the Auto-run prompt, and click "Ok" to confirm the upgrade of integration services.
26. After installation, click "Yes" to restart the machine.
27. Once the machine restarts, you should be able to see the "Network Adapter" under the "Networking" tab in Hyper-V has an "Ok" status.  Once the login screen appears, you should notice that IP addresses are shown against each of the network adapters.
29. Login.
30. Follow the instructions at this URL to enable automatic login to the "QA" account: https://technet.microsoft.com/en-us/library/ee872306.aspx
31. Reboot the machine and verify that it automatically logs in.
32. Turn off UAC in the Control Panel.  There is no known way to automate through UAC dialogs, and these may appear when Steam attempts to install redistributable packages.
33. Shutdown the machine.
34. Remove the Legacy Network Adapter from the machine.  It is no longer needed.
35. Start the machine.
36. From Start Menu, Right click on computer and select properties
37. Click Remote Settings
38. Click Allow Connections from any version of Remote Desktop (less secure)
39. Shutdown the machine
40. Under machine settings, add RemoteFX adapter (configure it however you like resolution, VRAM)
41. Start the machine again
42. After it logs in, you'll be prompted to restart (it has just installed the RemoteFX driver).  Click Restart Now
43. From here on out, you'll need to RDP to the machine to use it (Hyper-V will tell you the IP address to connect to, just use the QA account to login).
44. RDP'ing into the machine, shut it down.
45. With the VM turned off, from "Action" menu select "Checkpoint..." and name the checkpoint "Baseline".

## Test the baseline image

From this repository, run `.\TestGuest.ps1 -GuestType Windows7EntOrUlt -GuestName NAME_OF_VM -Target Baseline`.

This will validate that the machine can startup, that network adapters are configured correctly, and that GameTest can communicate with the VM using Guest Integrations and PowerShell.

## Install additional software

1. Start the machine.
2. Install Steam from the website: https://steamcdn-a.akamaihd.net/client/installer/SteamSetup.exe.
3. Run Steam and allow it to perform updates.
4. Close Steam without logging in.
5. Install AutoIt 3 from the website: https://www.autoitscript.com/cgi-bin/getfile.pl?autoit3/autoit-v3-setup.exe.
6. Restart the machine from the Start Menu.
7. Shutdown the machine.
8. With the VM turned off, from "Action" menu select "Checkpoint..." and name the checkpoint "ReadyToRun".