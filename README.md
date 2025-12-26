# OpenWrt Custom Build for Raspberry Pi 5 with Morse Micro Support

## Overview

This is a custom OpenWrt 24.10.5 firmware image built specifically for the **Raspberry Pi 5** with integrated support for **Morse Micro** wireless chipsets and **prplMesh** (Wi-Fi mesh networking).

### What is This Image?

This firmware replaces the standard Raspberry Pi OS with OpenWrt, a Linux-based operating system designed for embedded devices and routers. This custom build includes:

- **Base**: OpenWrt 24.10.5
- **Kernel**: Linux 6.6.119
- **Target Platform**: Broadcom BCM27xx (BCM2712 chipset)
- **Device**: Raspberry Pi 5
- **Special Features**:
  - Morse Micro wireless driver support (morse_driver v1.14.1)
  - prplMesh Wi-Fi mesh networking (v1.16.4)
  - Custom patches for kernel 6.6 compatibility

## Hardware Compatibility

### Supported Devices
- **Raspberry Pi 5** (BCM2712 chipset only)

### NOT Compatible With
- Raspberry Pi 4 (BCM2711)
- Raspberry Pi 3 (BCM2837)
- Earlier Raspberry Pi models

### Required Hardware
- Raspberry Pi 5 board
- MicroSD card (minimum 8GB recommended, 16GB+ preferred)
- Power supply (USB-C, 5V/3A minimum, 5V/5A recommended)
- Ethernet cable (for initial setup)
- Optional: Morse Micro wireless adapter (for wireless functionality)

## Chipset Details

### BCM2712 (Raspberry Pi 5)
- **CPU**: Quad-core ARM Cortex-A76 @ 2.4GHz
- **Architecture**: aarch64 (ARM 64-bit)
- **GPU**: VideoCore VII
- **RAM**: 4GB or 8GB LPDDR4X-4267

## Installation

### Prerequisites
- Linux, macOS, or Windows computer with SD card reader
- MicroSD card (8GB minimum)
- Basic command-line knowledge

### Step 1: Download the Image

Two images are provided:

1. **`openwrt-*-factory.img.gz`** - For fresh installations
2. **`openwrt-*-sysupgrade.img.gz`** - For upgrading existing OpenWrt installations

For first-time installation, use the **factory** image.

### Step 2: Write Image to SD Card

#### On Linux/macOS:

```bash
# Decompress the image
gunzip openwrt-24.10.5-morse-2.8.5-bcm27xx-bcm2712-rpi-5-squashfs-factory.img.gz

# Find your SD card device (be VERY careful - wrong device will destroy data!)
lsblk
# or
sudo fdisk -l

# Write the image (replace /dev/sdX with your SD card device, e.g., /dev/sdb)
# WARNING: This will ERASE ALL DATA on the target device!
sudo dd if=openwrt-24.10.5-morse-2.8.5-bcm27xx-bcm2712-rpi-5-squashfs-factory.img \
        of=/dev/sdX \
        bs=4M \
        status=progress \
        conv=fsync

# Ensure all data is written
sudo sync
```

#### On Windows:

1. Download and install [Etcher](https://www.balena.io/etcher/), [Rufus](https://rufus.ie/), or [RaspberryPiImager](Recommended)(https://www.raspberrypi.com/software/)
   - If you use RaspberryPiImager, skip to step 4. All steps after step 4 are specific for RaspberryPiImager ONLY.
2. Decompress the `.img.gz` file (use 7-Zip or similar)
3. Use Etcher/Rufus to write the `.img` file to your SD card
4. If you are using RaspberryPiImager, you do not need to Decompress or Unzip the `*.img.gz`files. Simply click-on and open whichever `*.img.gz`file you are going to use and follow the prompts through RaspberryPiImager.
5. Scroll to the bottom of the OS list and select 'Use Custom'
6. Then, select whichever `*.img.gz`file you want the RaspberryPi to boot from (Factory for new installs, upgrade for upgrades)
7. Next, select your choice of OS (64 bit recommended)
8. Mount it on your empty SD card
9. Complete your customization configurations
10. Finish writing your configurations to the SD card
   - If Windows offers to format the drive, cancel out of it and ignore it

#### On macOS (Alternative):

```bash
# Decompress
gunzip openwrt-24.10.5-morse-2.8.5-bcm27xx-bcm2712-rpi-5-squashfs-factory.img.gz

# Find disk identifier
diskutil list

# Unmount the disk (replace diskN with your SD card, e.g., disk2)
diskutil unmountDisk /dev/diskN

# Write image (use rdiskN for faster writing)
sudo dd if=openwrt-24.10.5-morse-2.8.5-bcm27xx-bcm2712-rpi-5-squashfs-factory.img \
        of=/dev/rdiskN \
        bs=4m \
        status=progress

# Eject safely
diskutil eject /dev/diskN
```

### Step 3: First Boot

1. **Insert SD card** into Raspberry Pi 5
2. **Connect Ethernet cable** to your network
3. **Power on** the Raspberry Pi 5
4. **Wait 1-2 minutes** for first boot to complete

### Step 4: Access OpenWrt

#### Default Network Configuration
- **LAN IP Address**: `192.168.1.1`
- **Default Gateway**: `192.168.1.1`
- **DHCP Range**: `192.168.1.100 - 192.168.1.250`

#### Web Interface (LuCI)

1. Connect your computer to the Raspberry Pi 5 via Ethernet
2. Open a web browser and navigate to: `http://192.168.1.1` or `http://10.42.0.1/`
3. **Default credentials**:
   - Username: `root`
   - Password: *(none - leave blank)*
4. **IMPORTANT**: Set a root password immediately after first login!

#### SSH Access

```bash
# From your computer connected to the same network
ssh root@192.168.1.1

# First login has no password (just press Enter)
# Set password immediately:
passwd
```

## Initial Configuration

### 1. Set Root Password (CRITICAL)

```bash
passwd
# Enter new password twice
```

### 2. Configure Internet Access

If your Raspberry Pi 5 has internet via Ethernet:

**Via Web Interface:**
1. Go to `Network` → `Interfaces`
2. Click `Edit` on the `WAN` interface
3. Configure according to your ISP (DHCP, Static, or PPPoE)
4. Save & Apply

**Via SSH:**
```bash
# Edit network configuration
vi /etc/config/network

# Restart networking
/etc/init.d/network restart
```

### 3. Update Package Lists

```bash
# Update package database
opkg update

# Install useful packages (optional)
opkg install nano htop wget curl
```

## Special Features

### Morse Micro Driver

This build includes the Morse Micro wireless driver (v1.14.1) with custom patches for kernel 6.6 compatibility.

**Patches Applied:**
- Enum type mismatch fixes (debug.h, firmware.h)
- Forward declarations for circular dependencies
- S1G beacon API updates for kernel 6.6
- SPI GPIO warning removal

**Usage:**
If you have a Morse Micro wireless adapter, the driver should auto-load. Check with:

```bash
# Check if driver is loaded
lsmod | grep morse

# View driver logs
dmesg | grep morse
```

### prplMesh Support

prplMesh (v1.16.4) is included for Wi-Fi mesh networking capabilities.

**Patches Applied:**
- Missing `<cstdint>` includes for GCC 13.3 compatibility
- Virtual function hiding fixes
- Pessimizing move optimizations

**Configuration:**
Refer to the [prplMesh documentation](https://prplfoundation.org/) for mesh setup.

## Troubleshooting

### Can't Access Web Interface

1. Verify Ethernet connection is active (link lights on)
2. Check your computer's IP is in `192.168.1.x` range
3. Try pinging: `ping 192.168.1.1`
4. Disable Wi-Fi on your computer (use Ethernet only)
5. Clear browser cache or try incognito/private mode

### SSH Connection Refused

1. Wait 2-3 minutes after power-on for full boot
2. Verify network connectivity: `ping 192.168.1.1`
3. Check SSH service: `telnet 192.168.1.1 22`
4. If still failing, connect a monitor/keyboard to debug

### Raspberry Pi Won't Boot

1. **Check power supply** - Pi 5 needs 5V/3A minimum (5V/5A recommended)
2. **Re-flash SD card** - corruption may have occurred
3. **Try different SD card** - some cards are incompatible
4. **Check SD card is fully inserted**
5. **Connect HDMI monitor** to view boot messages
6. **Connect 5v power supply to RaspberryPi** - this forces it to boot. It also told me that I needed to reassemble my stack to make the pogo-pins have a tighter contact with my Pi.

### Network Not Working

```bash
# Check interface status
ifconfig

# Check network configuration
cat /etc/config/network

# Restart networking
/etc/init.d/network restart

# Check logs
logread | grep network
```

### Morse Driver Not Loading

```bash
# Check for driver files
ls -la /lib/modules/*/morse*

# Try manual load
modprobe morse

# Check kernel messages
dmesg | tail -50
```

## Upgrading

### To Upgrade from Existing OpenWrt Installation

1. Download the **sysupgrade** image
2. **Backup your configuration**:
   ```bash
   # Via LuCI: System → Backup/Flash Firmware → Generate Archive
   # Via SSH:
   sysupgrade -b /tmp/backup-$(date +%Y%m%d).tar.gz
   ```
3. Upload and flash:
   ```bash
   # Via LuCI: System → Backup/Flash Firmware → Flash Image
   # Via SSH:
   sysupgrade -v openwrt-*-sysupgrade.img.gz
   ```

### To Upgrade This Custom Build

Since this is a custom build, you'll need to rebuild from source or wait for updated images from the build maintainer.

## Building from Source

This image was built using custom patches for kernel 6.6 compatibility. If you need to rebuild:

### Build Environment Requirements
- Ubuntu 22.04+ (or similar Linux distribution)
- 50GB+ free disk space
- 16GB+ RAM recommended
- Fast internet connection

### Patches Required

**morse_driver patches:**
1. `006-fix-enum-mismatch.patch` - morse_log_is_enabled type fix
2. `007-fix-firmware-enum-mismatch.patch` - morse_firmware_init type fix
3. `008-forward-declare-test-mode-enum.patch` - Circular dependency fix
4. `009-fix-s1g-beacon-api.patch` - Kernel 6.6 S1G beacon structure changes
5. `010-remove-spi-gpiod-warning.patch` - SPI_CONTROLLER_ENABLE_CS_GPIOD warning

**prplmesh patches:**
1. `001-add-missing-cstdint-include.patch` - netlink_event_listener.h
2. `002-add-cstdint-to-key-value-parser.patch` - key_value_parser.h
3. `003-fix-overloaded-virtual.patch` - Virtual function hiding
4. `004-remove-pessimizing-move.patch` - backhaul_manager.cpp
5. `005-remove-pessimizing-move-platform-manager.patch` - platform_manager.cpp
6. `006-remove-pessimizing-move-ambiorix.patch` - ambiorix_variant.cpp

### Quick Build Instructions

```bash
# Clone OpenWrt repository
git clone https://git.openwrt.org/openwrt/openwrt.git
cd openwrt

# Add feeds (including Morse feed)
# [Feed configuration steps here]

# Update and install feeds
./scripts/feeds update -a
./scripts/feeds install -a

# Apply custom patches to feeds/morse/kernel/morse_driver/patches/
# Apply custom patches to feeds/prpl/prplmesh/patches/

# Configure build
make menuconfig
# Select: Target System = Broadcom BCM27xx
# Select: Subtarget = BCM2712 (Raspberry Pi 5)
# Select: Target Profile = Raspberry Pi 5

# Build
make -j$(nproc)
```

## Technical Details

### Kernel Version
- Linux 6.6.119

### Target Architecture
- `aarch64_cortex-a76`

### Toolchain
- GCC 13.3.0
- musl libc

### File System
- SquashFS (compressed root filesystem)
- Overlay for persistent storage

### Partition Layout
```
/dev/mmcblk0p1  - Boot partition (FAT32)
/dev/mmcblk0p2  - Root filesystem (SquashFS)
/dev/mmcblk0p3  - Overlay (ext4, for configuration/data)
```

## Security Considerations

### IMPORTANT Security Steps

1. **Set root password immediately** after first login
2. **Change default LAN IP** if 192.168.1.x conflicts with your network
3. **Enable firewall** (enabled by default, but verify)
4. **Disable SSH password auth** and use keys:
   ```bash
   # Generate key on your computer
   ssh-keygen -t ed25519
   
   # Copy to OpenWrt
   ssh-copy-id root@192.168.1.1
   
   # Disable password auth
   uci set dropbear.@dropbear[0].PasswordAuth='off'
   uci commit dropbear
   /etc/init.d/dropbear restart
   ```
5. **Keep system updated**:
   ```bash
   opkg update
   opkg list-upgradable
   opkg upgrade <package>
   ```

### Firewall Rules

Default firewall configuration:
- WAN → Reject all incoming
- LAN → Accept all
- Forward LAN → WAN allowed (NAT)

## Performance

### Expected Performance on Raspberry Pi 5

- **Boot Time**: 30-60 seconds (cold boot)
- **Memory Usage**: ~150-250 MB (base system)
- **Network Throughput**: 
  - Ethernet: Up to 1 Gbps
  - Wi-Fi: Depends on adapter (Morse Micro specific)

## Support & Resources

### Official Documentation
- [OpenWrt Wiki](https://openwrt.org/docs/start)
- [OpenWrt Forum](https://forum.openwrt.org/)
- [Raspberry Pi 5 OpenWrt Page](https://openwrt.org/toh/raspberry_pi_foundation/raspberry_pi)

### Package Management
```bash
# Update package lists
opkg update

# Search for packages
opkg list | grep <keyword>

# Install package
opkg install <package>

# Remove package
opkg remove <package>
```

### Useful Packages

```bash
# System monitoring
opkg install htop iotop

# Network tools
opkg install tcpdump iperf3 mtr

# File transfer
opkg install rsync samba4-server

# VPN
opkg install openvpn-openssl wireguard-tools

# Ad blocking
opkg install adblock luci-app-adblock
```

## Known Issues

1. **First boot is slow** - Allow 2-3 minutes for initial setup
2. **USB devices may require kernel modules** - Install as needed via opkg
3. **Morse Micro support requires compatible hardware** - Driver loads but needs appropriate adapter
4. **Limited GPU/hardware acceleration** - VideoCore VII support is basic

## Changelog

### Version: 24.10.5-morse-2.8.5 (2024-12-26)

#### Added
- Morse Micro driver v1.14.1 with kernel 6.6 patches
- prplMesh v1.16.4 with GCC 13.3 compatibility patches
- Custom patches for BCM2712 (Raspberry Pi 5)

#### Fixed
- Enum/integer mismatch errors in morse_driver
- Missing header includes in prplMesh
- Virtual function hiding warnings
- Pessimizing move warnings in prplMesh
- S1G beacon API compatibility for kernel 6.6
- SPI GPIO descriptor warnings

#### Technical Changes
- Kernel: 6.6.119
- Toolchain: GCC 13.3.0, musl libc
- Architecture: aarch64_cortex-a76
- 11 custom patches applied across 2 packages

## License

OpenWrt is licensed under GPL-2.0. Individual packages may have different licenses.

This custom build includes:
- **OpenWrt**: GPL-2.0
- **Morse Micro driver**: Check vendor license
- **prplMesh**: Apache-2.0 / BSD-3-Clause (components vary)

## Credits

- **OpenWrt Project**: Core operating system
- **Morse Micro**: Wireless driver
- **prpl Foundation**: prplMesh Wi-Fi mesh
- **Raspberry Pi Foundation**: Hardware platform
- **Build maintained by**: Drexel (Custom patches for kernel 6.6 compatibility)

## Disclaimer

This is a **custom, unofficial build**. Use at your own risk. The maintainer provides no warranty or guarantee of fitness for any purpose.

**NOT RECOMMENDED FOR PRODUCTION USE WITHOUT THOROUGH TESTING**

Always maintain backups and have a recovery plan before deploying to critical systems.

---

**Build Date**: December 26, 2024  
**Builder**: Drexel  
**Build Environment**: Amazon Linux on EC2 instance  
**OpenWrt Version**: 24.10.5  
**Morse Version**: 2.8.5  
**Kernel**: 6.6.119
