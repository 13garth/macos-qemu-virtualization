# Mac OS Qemu Virtualization
A Virtualization Solution I used for running ubuntu on my mac

## Install Qemu through Homebrew.
- $ brew install qemu

## Create a local directory for your VM
* I created a folder in my user directory call "username/ubuntu/"
* go into the directory in your terminal
* * $ cd username/ubuntu/

### create the DISK
- inside "username/ubuntu/" run the following command
- $ qemu-img create -f qcow2 msl.qcow2 40G

# Download a VM image and start the VM
- Verifying which accelerators are available
- - $ qemu-system-x86_64 -accel help
- run the following command as is. Copy and paste. (If you have an issue. Try changing the "-accel tcg" to "-accel hvf")
```
qemu-system-x86_64 -accel tcg \
-m 4096 \
-hda msl.qcow2 \
-cdrom I-pasted-my-iso-image-inside-my-ubuntu-folder-and-just-pasted-the-iso-image-file-name-here.iso \
-boot d \
-smp 2 \
-net nic \
-net user,hostfwd=tcp::4022-:22 \
```

# Starting the VM
Set up the VM as normal, making sure to check these: 
a) leave the root password blank in order to allow for sudo access
b) make sure that the "username" of Debian is the same as your macOS username
c) uncheck all desktop environments and check the "SSH server" option.

Restart the VM with -boot d changed to -boot c and the -cdrom line removed. For example:
```
qemu-system-x86_64 -accel tcg \
-m 4096 \
-hda msl.qcow2 \
-boot c \
-smp 2 \
-net nic \
-net user,hostfwd=tcp::4022-:22 \
```

# Credit to: 
https://github.com/macbian-admin/macos-subsystem-for-linux
