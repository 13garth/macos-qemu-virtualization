Using accelorator tcg is very inefficient. But using hvf can cause bugs.<br>
hvf is hyper visor technology built into mac. You can install it and if you can get qemu to boot with hvf instead. It is supposedly supposed to run much faster.

---
#### addtional resources: 
- (Official Github Repo) https://github.com/qemu/qemu
- (Other Useful Resource) https://github.com/macbian-admin/macos-subsystem-for-linux

---
# Mac OS QEMU Virtualization
A simple guide for running Ubuntu (or other Linux distributions) on macOS using QEMU.

---

## 1. Install QEMU via Homebrew
```bash
brew update
brew install qemu
```

---

## 2. Create a Local Directory
Create and navigate into a folder (e.g., `~/ubuntu`):
```bash
mkdir ~/ubuntu
cd ~/ubuntu
```

---

## 3. Create the Disk Image
Inside `~/ubuntu/`, run:
```bash
qemu-img create -f qcow2 msl.qcow2 40G
```
This creates a 40 GB disk named `msl.qcow2`.

---

## 4. Check Available Accelerators
To see which accelerators your system supports (e.g., `hvf` or `tcg`):
```bash
qemu-system-x86_64 -accel help
```
- **Intel Macs** often support `hvf`.
- **Apple Silicon** or unsupported setups may need `tcg` (software emulation).

---

## 5. Start the VM Installer
Example command (adjust ISO file name and memory as needed):
```bash
qemu-system-x86_64 -accel tcg \
  -m 4096 \
  -hda msl.qcow2 \
  -cdrom ubuntu-24.04.1-desktop-amd64.iso \
  -boot d \
  -smp 2 \
  -net nic \
  -net user,hostfwd=tcp::4022-:22
```
> **Tip**: Replace `-accel tcg` with `-accel hvf` if `hvf` is supported. Increase `-m` to allocate more memory (e.g., `8192` for 8 GB) but leave enough RAM for macOS.

---

## 6. Installing Ubuntu / Debian
Proceed with the installer. If prompted:
- **Root Password**: Leave blank (allows `sudo` access).
- **Username**: Match your macOS username if desired.
- **Installation Options**: For a minimal setup, uncheck desktop environments and include the **SSH server**.

---

## 7. Boot from the Installed OS
After installation completes, remove the `-cdrom` and switch `-boot d` to `-boot c`:
```bash
qemu-system-x86_64 -accel tcg \
  -m 4096 \
  -hda msl.qcow2 \
  -boot c \
  -smp 2 \
  -net nic \
  -net user,hostfwd=tcp::4022-:22
```
> This boots from the installed disk rather than the ISO.

---

## 8. Basic QEMU Commands

**View Created VMs**  
QEMU doesn’t manage VMs like VirtualBox. You manually track disk files:
```bash
ls *.qcow2
```

**Remove a VM**  
Delete the `.qcow2` file:
```bash
rm msl.qcow2
```

**Create a New VM**  
Create another disk image (e.g., 20 GB):
```bash
qemu-img create -f qcow2 newvm.qcow2 20G
```

**Start an Existing VM**  
```bash
qemu-system-x86_64 -accel tcg \
  -m 4096 \
  -hda newvm.qcow2 \
  -boot c \
  -smp 2
```

**Stop a Running VM**  
- Shutdown from inside the guest, or
```bash
killall qemu-system-x86_64
```

---

## 9. Improving Performance
1. **Hardware Acceleration**: Use `-accel hvf` if on an Intel Mac with HVF support.  
2. **Increase CPU Cores**: `-smp 4` or more if available.  
3. **Optimize Disk I/O**: Try virtio drivers:
   ```bash
   -drive file=msl.qcow2,if=virtio
   ```
4. **Allocate Adequate Memory**: `-m 8192` (8 GB) or more, but leave enough for macOS.

---

### Notes & Tips
- On **Apple Silicon**, x86_64 guests must often use `-accel tcg`, making performance slower. Tools like [UTM](https://github.com/utmapp/UTM) may perform better by using Apple’s Virtualization.framework under the hood.  
- Always keep your QEMU and macOS updated to ensure the latest hardware-acceleration support.

---

**Happy Virtualizing!**

---

# Extra 

QEMU does not have a centralized location to "store installations" like WSL or VirtualBox. Instead, **QEMU stores everything in the disk image file(s)** you create (e.g., `.qcow2`), which you specify in the command when starting a VM.

Here’s a breakdown of where files are stored:

---

### 1. **Disk Images**
- When you create a disk image with `qemu-img`, it is stored wherever you specify.
- Example: If you run:
  ```bash
  qemu-img create -f qcow2 ~/ubuntu/msl.qcow2 40G
  ```
  The disk image (`msl.qcow2`) is saved in the `~/ubuntu` directory.  
  This file contains the entire OS installation, including all its data.

---

### 2. **Configuration and Metadata**
QEMU doesn’t store configurations or metadata globally. Instead,<br>
all configurations (e.g., RAM, CPU, accelerators) are provided as **command-line arguments** each time you run the VM.<br>
To persist settings:
- Save your QEMU command to a script for reuse.

---

### 3. **ISO Files**
- The OS installation ISO is usually stored where you downloaded it. QEMU doesn’t copy the ISO;<br>
it only references the file during installation.
- Example: If your ISO is in `~/ubuntu`, QEMU will read it directly from there.

---

### 4. **Temporary and Log Files**
- **Snapshots**: If you take snapshots, these may be stored alongside the `.qcow2` file.
- **Temporary Files**: QEMU may store temporary files in `/tmp` or similar system directories, but these are cleared after shutdown.

---

### Summary
- **OS and data**: Stored in the `.qcow2` disk file you specify.
- **ISOs**: Stored where you download them (not moved by QEMU).
- **Configurations**: Provided via commands/scripts (no centralized config).
- **Temporary files**: Stored in system temp directories during runtime.

If you’d like to explore the contents of your `.qcow2` files, you can use QEMU tools like `qemu-img` to manage them.
