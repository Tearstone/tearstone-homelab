Model: Zyxel NAS326
Firmware: V5.21(AAZF.18) Hotfix 01

## Storage configuration
Hard Disk (1) 4 TB ATAST4000VN008-2DR1 SC60  
Hard Disk (2) 4 TB ATAST4000VN008-2DR1 SC60 

Disk Group in a Raid 1 Mirrored Array

Volume 1 on Disk Group ext4 file system

## Packages
### NFS
Share: homelab  
Protocol: NFS  
Client: lab-core01  
Client IP: 192.168.12.244

## App Package Repository Workaround

The Zyxel NAS 326 is EOL/EOS. The app manager no longer works. This is the workaround to download the latest package library to run locally. 

1-2. Steps removed to condense the steps below

3. Login as admin on NAS web-interface from local PC or Mac

4. Check for latest firmware installed - NAS326_V5.21(AAZF.18)C0
   Latest Firmware: https://download.zyxel.com/NAS326/firmware/NAS326_V5.21(AAZF.18)C0.zip

6. Enable SSH in NAS Control panel

7. Enable Anonymous FTP in NAS Control panel

8. Login as admin on NAS web-interface from local PC or Mac

9. From "Control panel – shared folders“ create folder NAS326 in same path as admin folder and give read/write access to all users

10. From "NAS File Browser“ Create folder zypkg under NAS326 folder

11. Download zip file from NAS326 link: https://download.zyxel.com/NAS326/zypkg/NAS326_zypkg_5.21.zip

12. From "NAS File Browser“ upload zip file to zypkg folder

13. From "NAS File Browser“ decompress zip folder and observe folder „NAS326_zypkg_5.21“ with subfolder 5.21 under zypkg folder

14. From "NAS File Browser“ move 5.21 folder to zypkg folder

15. Delete zip file and "NAS326_zypkg_5.21“ folder and situation is: In folder NAS326 is folder zypkg. In zypkg folder is 5.21 folder with 18 files (17 zpkg files and 1 tgz file)

16. Start SSH client, on my server 192.168.x.x, username=root, password=same as admin password

17. Execute command: mkdir /i-data/sysvol/admin/zy-pkgs

18. Execute command: echo "ftp://192.168.1.99" > /i-data/sysvol/admin/zy-pkgs/web_prefix

19. Disconnect from SSH session

20. Check from "NAS File Browser“ and validate the admin folder is in folder zy-pkgs with file web_prefix

21. Check the web_prefix file includes the text ftp://192.168.x.x

22. In the App centar try to update all apps. If you get "Download list success“ then you are good to go!

Reference: https://community.zyxel.com/en/discussion/29667/important-announcement-end-of-ftp-service-support-for-home-nas-series
