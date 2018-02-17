    #!/usr/bin/env python3
    """ror.py - Read-Only Root

    This script automates the steps described by ejolson in this thread:
    https://www.raspberrypi.org/forums/viewtopic.php?f=63&t=161416

    Usage:
        sudo ./ror.py create
            Initializes and enables read-only root file system.
        sudo ./ror.py disable
            Disables read-only root file system.
        sudo ./ror.py enable
            Enables read-only root file system.
        sudo ./ror.py destroy
            Cleans up all changes made by the script (read-only root must be disabled!).
    """



    import sys
    import os
    import shutil
    import subprocess



    def edit(filename, old, new):
        with open(filename, "r") as f: data = f.read()
        for i in zip(old, new):
            data = data.replace(i[0], i[1])
        with open(filename, "w") as f: f.write(data)
    def append(filename, s):
        with open(filename, "a") as f: f.write(s)
    def prepend(filename, s):
        with open(filename, "r") as f: data = f.read()
        with open(filename, "w") as f: f.write(s + data)



    def create():
       
        print("setting up read only root...")
       
        shutil.copyfile("/usr/share/initramfs-tools/hook-functions", "/usr/share/initramfs-tools/hook-functions._ror_backup_")
        shutil.copyfile("/boot/config.txt", "/boot/config.txt._ror_backup_")
        shutil.copyfile("/boot/cmdline.txt", "/boot/cmdline.txt._ror_backup_")
        print("backup created!")
       
        old = ['modules="$modules ehci-pci ehci-orion ehci-hcd ohci-hcd ohci-pci uhci-hcd usbhid"']
        new = ['modules="$modules ehci-pci ehci-orion ehci-hcd ohci-hcd ohci-pci uhci-hcd usbhid overlay"']
        edit("/usr/share/initramfs-tools/hook-functions", old, new)
        print("hook-functions edited!")
       
        shutil.copyfile("/usr/share/initramfs-tools/scripts/local", "/usr/share/initramfs-tools/scripts/overlay")
        shutil.copytree("/usr/share/initramfs-tools/scripts/local-premount", "/usr/share/initramfs-tools/scripts/overlay-premount")
        shutil.copytree("/usr/share/initramfs-tools/scripts/local-bottom", "/usr/share/initramfs-tools/scripts/overlay-bottom")
        print("overlay created!")
       
        old = [
            'ROOT=$(resolve_device "$ROOT")\n\n\tif [ "${readonly}" = "y" ]; then\n\t\troflag=-r\n\telse\n\t\troflag=-w\n\tfi',
            '# Mount root\n\tif [ "${FSTYPE}" != "unknown" ]; then\n\t\tmount ${roflag} -t ${FSTYPE} ${ROOTFLAGS} ${ROOT} ${rootmnt}\n\telse\n\t\tmount ${roflag} ${ROOTFLAGS} ${ROOT} ${rootmnt}\n\tfi'
            ]
        new = [
            'ROOT=$(resolve_device "$ROOT")\n\n\t#if [ "${readonly}" = "y" ]; then\n\t\troflag=-r\n\t#else\n\t#\troflag=-w\n\t#fi',
            '# Mount root\n\tmkdir /upper /lower\n\tif [ "${FSTYPE}" != "unknown" ]; then\n\t\tmount ${roflag} -t ${FSTYPE} ${ROOTFLAGS} ${ROOT} /lower\n\telse\n\t\tmount ${roflag} ${ROOTFLAGS} ${ROOT} /lower\n\tfi\n\tmodprobe overlay\n\tmount -t tmpfs tmpfs /upper\n\tmkdir /upper/data /upper/work\n\tmount -t overlay -olowerdir=/lower,upperdir=/upper/data,workdir=/upper/work overlay ${rootmnt}'
            ]
        edit("/usr/share/initramfs-tools/scripts/overlay", old, new)
        print("overlay edited!")
       
        # todo: support BCM27835!
       
        release = os.uname().release
        subprocess.call(["update-initramfs", "-c", "-k", release])
        shutil.move("/boot/initrd.img-" + release, "/boot/initrd7.img")
        print("initramfs created!")
       
        append("/boot/config.txt", "\n\n\nkernel=kernel7.img\ninitramfs initrd7.img\n")
        print("config.txt edited!")
       
        prepend("/boot/cmdline.txt", "boot=overlay ")
        print("cmdline.txt edited!")
       
        print("done! please reboot!")



    def disable():
        shutil.move("/boot/config.txt._ror_backup_", "/boot/config.txt")
        shutil.move("/boot/cmdline.txt._ror_backup_", "/boot/cmdline.txt")
        print("root will be writeable after the next reboot!")



    def enable():
        if os.path.exists("/boot/config.txt._ror_backup_") and os.path.exists("/boot/cmdline.txt._ror_backup_"):
            print("already enabled!")
            return
        shutil.copyfile("/boot/config.txt", "/boot/config.txt._ror_backup_")
        shutil.copyfile("/boot/cmdline.txt", "/boot/cmdline.txt._ror_backup_")
        append("/boot/config.txt", "\n\n\nkernel=kernel7.img\ninitramfs initrd7.img\n")
        prepend("/boot/cmdline.txt", "boot=overlay ")
        print("root will be read-only after the next reboot!")



    def destroy():
       
        print("cleaning up...")
       
        shutil.move("/usr/share/initramfs-tools/hook-functions._ror_backup_", "/usr/share/initramfs-tools/hook-functions")
       
        os.remove("/usr/share/initramfs-tools/scripts/overlay")
        shutil.rmtree("/usr/share/initramfs-tools/scripts/overlay-premount")
        shutil.rmtree("/usr/share/initramfs-tools/scripts/overlay-bottom")
       
        os.remove("/boot/initrd7.img")

        print("done!")



    if __name__ == "__main__":
       
        if sys.argv[1] == "enable": enable()
        elif sys.argv[1] == "disable": disable()
        elif sys.argv[1] == "destroy": destroy()
        elif sys.argv[1] == "create": create()
