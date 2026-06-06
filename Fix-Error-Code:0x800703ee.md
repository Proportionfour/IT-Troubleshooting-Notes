# [Windows] "We can't tell if your PC has enough storage to continue installing Windows 11" — Corrupted BCD
 
> *"We can't tell if your PC has enough storage to continue installing Windows 11."*
> This error message tormented me for quite some time. Below I'll go over the common fixes I tried, what didn't work, and ultimately the solution that resolved the issue.
 
---
 
## Problem summary
 
Windows 10 cannot complete the upgrade to Windows 11, throwing the error: **"We can't tell if your PC has enough storage to continue installing Windows 11."** The error is misleading — the root cause is a corrupted Boot Configuration Data (BCD) file, not a storage issue.
 
---
 
## Environment
 
- **OS / Platform**: Windows 10 (upgrading to Windows 11)
- **Hardware** (if relevant): Client PC
- **Software / App** (if relevant): Windows Update, Windows 11 Media Creation Tool, RUFUS
- **Network context** (if relevant): N/A
---
 
## Symptoms
 
- Windows Update fails mid-upgrade with the message: *"We can't tell if your PC has enough storage to continue installing Windows 11."*
- Error persists regardless of available disk space
- Upgrade fails whether initiated through Windows Update, the Media Creation Tool, or the Update Assistant
---
 
## What I tried
 
1. **Updating through different installation media** — Instead of clicking the Update button, I tried mounting the Windows 11 ISO using the Media Creation Tool and also used Microsoft's Update Assistant Tool. Neither worked.
2. **Running the Windows Update Troubleshooter** — Found no errors.
3. **Unplugging all USB devices and reinstalling USB drivers** via Device Manager — Didn't work.
4. **Disabling antivirus** — Antivirus software can sometimes block Windows Update from accessing important OS files, causing update errors. Disabling it made no difference.
5. **Repairing the file system on the C: drive** — Open File Explorer, right-click the C: drive, select **Properties → Tools tab → Check** under Error Checking. Found no issues.
6. **Running SFC to repair corrupted system files** — In an Administrator Command Prompt:
   ```
   sfc /scannow
   ```
   No errors found.
7. **Performing a Windows clean boot** — Disabled unnecessary startup services. Didn't work.
8. **Extending the boot EFI partition** — Didn't work.
---
 
## Root cause
 
The Windows setup error log at `C:\$WINDOWS.~BT\Sources\Panther\setuperr.log` contained the entry:
 
```
BCD: File is not system store.
```
 
This points to a corrupted or missing BCD (Boot Configuration Data) file. Running the following command in an Administrator Command Prompt confirmed the diagnosis — instead of returning a list of boot entries, it returned an error:
 
```
bcdedit /enum all
```
 
The BCD needed to be fully rebuilt.
 
---
 
## Resolution
 
**Requirement:** A Windows 11 bootable USB drive. The Media Creation Tool can create one, but in this case it produced a corrupted bootloader. [RUFUS](https://rufus.ie) (a free utility for creating bootable USB drives) was used instead and worked reliably.
 
### Step 1 — Boot from the USB
 
Plug the USB drive into the PC, restart the computer, and press the boot menu key (usually **F11**, **F12**, or **ESC**) to enter the boot menu. Select the USB drive. When the Windows Setup screen appears, click **Next**, then choose **"Repair your computer"**. Do **not** click Install.
 
### Step 2 — Open Command Prompt
 
Navigate to: **Troubleshoot → Advanced Options → Command Prompt**
 
### Step 3 — Rebuild the BCD
 
Run these commands one at a time:
 
```
bootrec /fixmbr
bootrec /fixboot
bootrec /scanos
bootrec /rebuildbcd
```
 
If prompted to add an installation to the boot list, press **A** to add all.
 
### Step 4 — Force a clean BCD export
 
```
bcdedit /export C:\BCD_Backup
del C:\boot\bcd
bootrec /rebuildbcd
```
 
### Step 5 — Restart and retry
 
Remove the USB drive, boot Windows normally, and attempt the upgrade again.
 
---
 
## Time to resolve
 
- **Total time**: ~15 minutes once diagnosed
- **Difficulty**: Medium — straightforward fix, but diagnosing the BCD as the cause behind a misleading storage error message takes time
---
 
## Notes & caveats
 
- The error message ("storage") is a red herring — don't let it send you down the wrong path. Always check the setup error log first.
- The Windows 11 Media Creation Tool produced a corrupted bootloader in this case. RUFUS is a more reliable alternative.
- The `setuperr.log` file (`C:\$WINDOWS.~BT\Sources\Panther\setuperr.log`) is invaluable for diagnosing upgrade failures — check it early before trying generic fixes.
- This fix requires a bootable Windows USB drive, so have one ready or factor in time to create one.
---
 
## Tags
 
`windows` `windows-11` `windows-update` `bcd` `boot` `upgrade`
