# overlayRoot.sh
Squash File System over OverlayFS

Forked from: https://wiki.psuter.ch/doku.php?id=solve_raspbian_sd_card_corruption_issues_with_read-only_mounted_root_partition

# Installation
1. Copy this script to **/sbin/**
    - I should copy to **/home/user** folder by:
        ```
        scp -r /overlayRoot_path user@ip_address:/destination_path
        ```
    - After copy the script, simply move to **/sbin** path runing:
        ```
        sudo mv /home/user/overlayRoot.sh /sbin/
        ```
2. Make it executable:
    ```
    chmod +x /sbin/overlayRoot.sh
    ```
3. Change your boot parameter in **/boot/cmdline.txt**. Adding `init=/sbin/overlayRoot.sh`:
    - **cmdline.txt** with OverlayFS
    ```
    console=serial0,115200 console=tty1 root=PARTUUID=97652995-02 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait init=/sbin/overlayRoot.sh
    ```
    - **cmdline.txt** without OverlayFS
    ```
    console=serial0,115200 console=tty1 root=PARTUUID=97652995-02 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait 
    ```
4. Reboot time }: )

# What does it actually do
The script will mount a SquashFS in OverlayFS over the original root file system using tmpfs as upper layer.
1. Get all useful rootfs information from `/etc/fstab`.
2. Mount a tmpfs at `/mnt/overlay`, create `/mnt/overlay/upper`, `/mnt/overlay/work`, `/mnt/overlay/newroot`.
3. Bind mount rootfs to `/mnt/lower`.
4. Mount OverlayFS using `/mnt/lower` as lower, `/mnt/overlay/upper` and `/mnt/overlay/work` as upper and work directory, `/mnt/overlay/newroot` as merged destination.
5. `pivot_root` to `/mnt/overlay/newroot` and put old root to `/mnt`.
6. Move mount point `/mnt/mnt/lower`(the original `/mnt/lower`) to `/lower`, `/mnt/mnt/overlay`(the original `/mnt/overlay`) to `/overlay`.
7. Move all other useful virtual filesystems like procfs or devfs to new root.
8. Unmount `/mnt`(the original root file system).
9. Exec `/sbin/init` and continue the init process.
