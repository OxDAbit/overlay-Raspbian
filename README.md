# overlayRoot.sh
Squash File System over OverlayFS

Forked from: https://wiki.psuter.ch/doku.php?id=solve_raspbian_sd_card_corruption_issues_with_read-only_mounted_root_partition

# Repository folder structure
```
/src
├── overlay
│   └── overlayRoot.sh                  # Original script v1.1 Created by Pascal Suter
│
└── overlay_sfs                         # Custom resources
    ├── boot                            # Contain boot files
    │   ├── cmdline-no_overlay.txt      # Boot Raspberry without OverlayFS
    │   └── cmdline-overlay.txt         # Boot Raspberry with OverlayFS
    └── sbin                            # Script to create OverlayFS and SquashFS
        └── overlayRoot.sh              # SquashFS over OverlayFS script v1.4
```

# Installation
**NOTE:** _This script has been tested with Raspbian OS (32-bit) Lite (Debian Buster)_

1. First of all, create the folder structure:
    ```
    /lib/live/
    ├── mounted
    │   ├── ro          # Where Lower overlay folder is located
    │   ├── rw          # Where Upper and Work overlay folder is located
    │   └── squashed    # Where .sfs file is unsquashed
    └── squashfs        # Where .sfs file is located
    ```

    To do that:
    ```
    sudo -s
    mkdir /lib/live
    mkdir /lib/live/mounted
    mkdir /lib/live/mounted/ro
    mkdir /lib/live/mounted/rw
    mkdir /lib/live/mounted/squashed
    mkdir /lib/live/squahfs
    ```
2. Install Squasfs 
    ``` 
    sudo apt-get install squashfs-tools -y 
    ```
3. Copy [overlayRoot.sh](https://github.com/OxDAbit/overlayRoot.sh/blob/master/src/overlay_sfs/sbin/overlayRoot.sh) script into **/sbin/**
    - I usually copy from my computer to Raspberry **/home/user** path using:
        ```
        # From computer terminal
        sudo scp -r /pc_overlayRoot rsp_user@rsp_ip_address:/destination_path
        ```
    - After copy the script, simply move to **/sbin** path runing:
        ```
        sudo mv /home/user/overlayRoot.sh /sbin/
        ```
    - Make it executable:
        ``` 
        chmod +x /sbin/overlayRoot.sh
        ```
4. Disable **swap**
    ```
    dphys-swapfile swapoff
    dphys-swapfile uninstall
    update-rc.d dphys-swapfile remove
    systemctl stop dphys-swapfile
    systemctl disable dphys-swapfile
    ```
4. Change your boot parameter in **/boot/cmdline.txt**. Adding `init=/sbin/overlayRoot.sh`:
    - **cmdline.txt** with OverlayFS
    ```
    console=serial0,115200 console=tty1 root=PARTUUID=97652995-02 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait init=/sbin/overlayRoot.sh
    ```
    - **cmdline.txt** without OverlayFS
    ```
    console=serial0,115200 console=tty1 root=PARTUUID=97652995-02 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait 
    ```
    **NOTE:** _Can copy [cmdline-overlay.txt](https://github.com/OxDAbit/overlayRoot.sh/blob/master/src/overlay_sfs/boot/cmdline-overlay.txt) and [cmdline-no_overlay.txt](https://github.com/OxDAbit/overlayRoot.sh/blob/master/src/overlay_sfs/boot/cmdline-no_overlay.txt) from this repository and paste in your system_
4. Reboot time }: )
    ```
    reboot 
    ```

# Enable / Disable OverlayFS
Copy **cmdline-no_overlay.txt** and **cmdline-overlay.txt** into **/boot/** path.

After runing the script **/boot/** partition is mounted as a RW. To Enable or Disable OverlayFS:
- **Enable** OverlayFS:
    ``` 
    cp /boot/cmdline-overlay.txt /boot/cmdline.txt
    ```
- **Disable** OverlayFS:
    ```
    cp  /boot/cmdline-no_overlay.txt /boot/cmdline.txt
    ```
- After modify **cmdline.txt** file reboot Rapsberry `sudo reboot`

# Create Squash file
For this example I've created a 3 test files which will be located in **/home** and **/etc** folders after mount OverlayFS.
1. Create a temp folder in **/home/user** (for exemple):
    ```
    sudo -s
    mkdir /home/user/squahsfs_tmp 
    ```
2. Inside the squash file we will add a file which will be located inside **/etc/** and **/home/** folders (just for testing), so we should create 2 new folders inside **squashfs_tmp**:
    ``` 
    cd /home/user/squahsfs_tmp
    mkdir etc
    mkdir home
    ```
3. Create some test files inside this folders:
    ```
    touch etc/hello_etc
    touch home/home_file_01
    touch home/home_file_01
    ```
4. We've implemented the changes so is time to create a SFS file:
    ```
    cd ..                                               # We're located in /home/user path right now
    squashfs squahsfs_tmp file_system.sfs -comp xz      # Creating SFS File }:)
    ```
    This command generate a _sfs_ file call **file_system.sfs** based in **squahfs_tmp** folder content.
5. In the script the **/lib/live/squahsfs** directory is defined as the location where the file will be found the squash files so we should move the file.
    ```
    mv /home/user/squahsfs_tmp /lib/live/squashfs
    ```
6. Remove **squasfs_tmp** folder
    ```
    rm -rf /home/user/squashfs_tmp 
    ```
7. Enable overlay
    ```
    cp /boot/cmdline-overlay.txt /boot/cmdline.txt
    ```
8. Reboot time }: )
    ```
    reboot 
    ```
9. If everything has worked as we expected we will see:
    ```
    ls /home
     ├── home_file_01
     ├── home_file_02
     └── other /home files...

    ls /etc
     ├── hello_etc
     └── other /etc files...
    ```

# Update Squash file
Update squashs file adding or removing some file. To do that:
1. Reboot system without OverlayFS
    ```
    sudo -s
    cp  /boot/cmdline-no_overlay.txt /boot/cmdline.txt
    reboot
    ```
2. Go to **squashfs** default path
    ```
    sudo -s
    cd /lib/live/squashfs
    ```
3. Unsquash **file_system.sfs**
    ```
    unsquashfs -f file_system.sfs
    ```
    After run this command a folder **squashfs-root** will appear.
    ```
    /lib/live/squashfs
      ├── file_sytem.sfs
      └── squashfs-root
    ```
4. Modify **squashfs-root** content removing **home_file_02** (for exemple)
    ```
    rm -rf squashfs-root/home/home_file_02 
    ```
5. Remove old **file_system.sfs**
    ```
    rm -rf file_system.sfs
    ```
6. Create the new _sfs_ file
    ```
    squashfs squashfs-root file_system.sfs -comp xz
    ```
7. Now modify **cmdline.txt** to mount overlayFS after reboot and apply the sfs changes
    ```
    cp /boot/cmdline-overlay.txt /boot/cmdline.txt
    ```
8. Rebot time }: )
    ```
    reboot 
    ```

The other way to modify squashfs:
1. Remount RO partition
    ```
    sudo -s
    mount -o remount,rw /lib/live/mounted/ro 
    ```
2. Go to **squashfs** folder
    ```
    cd /lib/live/mounted/ro/lib/live/squashfs
    ```
3. Repeat steps from 3 until 8 indicated in the previous section

Contact
=======
- Twitter. [**@0xDA_bit**](https://twitter.com/0xDA_bit)
- Github. [**OxDAbit**](https://github.com/OxDAbit)
- Mail. **0xdabit@gmail.com**