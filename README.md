# Mac OS Qemu Virtualization
A Virtualization Solution I used for running ubuntu on my mac

## Install Qemu through Homebrew.
- brew install qemu

## Create a local directory for your VM
* I created a folder in my user directory call "username/ubuntu/"
* go into the directory in your terminal
* * cd username/ubuntu/

### create the DISK
- inside "username/ubuntu/" run the following command
- qemu-img create -f qcow2 msl.qcow2 40G

# Download a VM image and start the VM
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
