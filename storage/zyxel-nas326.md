Model: Zyxel NAS326  
Firmware: V5.21(AAZF.18) Hotfix 01

## Storage configuration

Hard Disk (1) 4 TB ATAST4000VN008-2DR1 SC60  
Hard Disk (2) 4 TB ATAST4000VN008-2DR1 SC60

Disk Group in a RAID 1 mirrored array

Volume 1 on Disk Group, ext4 file system

## Packages

### NFS

Share: homelab  
Protocol: NFS  
Client: lab-core01

Actual client addressing is intentionally omitted from this public documentation.

## App Package Repository Workaround

The Zyxel NAS326 is EOL/EOS. The app manager no longer works. This is the workaround to download the latest package library to run locally.

1-2. Steps removed to condense the steps below

3. Login as admin on NAS web interface from local PC or Mac

4. Check for latest firmware installed - NAS326_V5.21(AAZF.18)C0  
   Latest Firmware: https://download.zyxel.com/NAS326/firmware/NAS326_V5.21(AAZF.18)C0.zip

6. Enable SSH in NAS Control panel

7. Enable Anonymous FTP in NAS Control panel

8. Login as admin on NAS web interface from local PC or Mac

9. From Control panel – shared folders create folder NAS326 in same path as admin folder and give read/write access to all users

10. From NAS File Browser create folder zypkg under NAS326 folder

11. Download zip file from NAS326 link: https://download.zyxel.com/NAS326/zypkg/NAS326_zypkg_5.21.zip

12. From NAS File Browser upload zip file to zypkg folder

13. From NAS File Browser decompress zip folder and observe folder `NAS326_zypkg_5.21` with subfolder 5.21 under zypkg folder

14. From NAS File Browser move 5.21 folder to zypkg folder

15. Delete zip file and `NAS326_zypkg_5.21` folder. The resulting structure is a NAS326 folder containing zypkg, with a 5.21 folder containing the package files.

16. Start an SSH client and connect to the NAS using the appropriate local administrative account.

17. Execute:

```bash
mkdir /i-data/sysvol/admin/zy-pkgs
```

18. Create the local package repository prefix using the NAS's actual local repository address. The real address is intentionally omitted from this public documentation.

19. Disconnect from the SSH session

20. Check from NAS File Browser and validate the admin folder is in folder zy-pkgs with file web_prefix

21. Check the web_prefix file includes the local FTP repository address

22. In the App Center try to update all apps. If you get Download list success then the repository is working.

Reference: https://community.zyxel.com/en/discussion/29667/important-announcement-end-of-ftp-service-support-for-home-nas-series
