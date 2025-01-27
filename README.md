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
