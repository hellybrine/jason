# Hardware Setup

Jason's hardware preparation follows a **security-first approach** where every component is validated and configured before network exposure. This section covers the physical preparation required to transform commodity single-board computers into a coordinated security fabric.

## Hardware Requirements

### Core Components
- **NanoPi R2S**: Segmentation plane (IPFire firewall)
- **NanoPi R1S**: Access plane (OpenWrt wireless AP)  
- **Raspberry Pi 4B**: Services plane (containerized security services)

### Essential Accessories
- **USB-to-UART adapter** (3.3V logic, FTDI-based recommended)
- **Jumper wires** for UART connections
- **High-quality USB-C/micro-USB power supplies** (2.5A minimum for each board)
- **Ethernet cables** (Cat6 recommended for reliable gigabit performance)
- **MicroSD cards**: Class 10 or UHS-I, **minimum 32GB capacity**

### Storage Requirements and Security Implications

**MicroSD Card Selection Criteria**:
- **Performance**: Class 10 minimum, UHS-I preferred for reduced boot times and improved I/O performance
- **Capacity**: 32GB minimum to accommodate base OS, security services, and log retention
- **Reliability**: Name-brand cards from reputable manufacturers to prevent corruption-induced security failures
- **Write endurance**: High-endurance cards preferred for nodes with extensive logging requirements

**Security consideration**: Cheap or counterfeit SD cards frequently fail under sustained write operations, which can compromise security logging and create undetected protection gaps.

## Pre-Installation Preparation

### SD Card Preparation and Verification

**Initial Card Validation**:
```
# Verify card capacity and health (Linux/macOS)
lsblk                          # Identify card device
sudo badblocks -wsv /dev/sdX   # Write/read test (destructive)

# Windows alternative: use H2testw for capacity/integrity verification
```

**Secure Formatting**:
```
# Complete data destruction (security requirement)
sudo dd if=/dev/zero of=/dev/sdX bs=1M status=progress
sudo fdisk /dev/sdX            # Create new partition table
```

This process ensures **no residual data** from previous installations that could interfere with security configurations.

### UART Console Setup

**Hardware Connection Requirements**:

**NanoPi R2S Debug Header** (3-pin, 2.54mm spacing):
```
[GND] [RX] [TX]
 1     2    3
```

**NanoPi R1S Debug Header** (3-pin, 2.54mm spacing):
```
[3.3V] [TX] [RX]
  1     2    3
```

**Connection Mapping**:
- UART Adapter GND → Board GND
- UART Adapter RX → Board TX
- UART Adapter TX → Board RX

**Critical timing consideration**: NanoPi boards use **115200 baud**, which requires UART adapter compatibility verification before deployment.

## Image Acquisition and Verification

### Official Image Sources

**IPFire for NanoPi R2S**:
- Source: `https://downloads.ipfire.org/releases/ipfire-2.x/2.29-core186/`
- File: `ipfire-2.29.armv6l-multi.img.gz`
- **SHA256 verification mandatory** for security integrity and general best practices

**OpenWrt for NanoPi R1S**:
- Source: `https://downloads.openwrt.org/releases/23.05.x/targets/sunxi/cortexa7/`
- File: `openwrt-23.05.x-sunxi-cortexa7-friendlyarm_nanopi-r1s-h3-squashfs-sdcard.img.gz`

**Raspberry Pi OS (64-bit Lite)**:
- Source: `https://downloads.raspberrypi.org/raspios_lite_arm64/images/`
- File: Latest stable release
- **Lite version required** for better resource utilisation

### Cryptographic Verification

**Best practice**: All downloaded images must be cryptographically verified before flashing.

```
# Download checksums and signatures
wget https://downloads.ipfire.org/releases/ipfire-2.x/2.29-core186/ipfire-2.29.armv6l-multi.img.gz.sha256
wget https://downloads.openwrt.org/releases/23.05.x/targets/sunxi/cortexa7/sha256sums

# Verify checksums
sha256sum -c ipfire-2.29.armv6l-multi.img.gz.sha256
sha256sum -c sha256sums

# GPG signature verification (when available)
gpg --verify sha256sums.asc sha256sums
```

**Security rationale**: Image tampering represents a supply chain attack vector that could compromise the entire security architecture. Cryptographic verification ensures image integrity and authenticity.

## Flashing Procedures

### Cross-Platform Flashing Tools

**Recommended Tools by Platform**:

**Windows**: 
- **Rufus 4.x** (rufus.ie) - Reliable, supports compression formats
- **Balena Etcher** - Cross-platform option with built-in verification

**Linux**:
- **dd command** - Native, scriptable, reliable
- **Balena Etcher** - GUI option with verification
- **Raspberry Pi Imager** - Simplified for Pi-specific images

### Board-Specific Flashing Procedures

**NanoPi R2S (IPFire)**:
```
# Extract compressed image
gunzip ipfire-2.29.armv6l-multi.img.gz

# Flash to SD card (Linux/macOS)
sudo dd if=ipfire-2.29.armv6l-multi.img of=/dev/sdX bs=4M status=progress oflag=sync

# Verify write completion
sudo sync && sudo eject /dev/sdX
```

**Windows (Rufus procedure)**:
1. Launch Rufus as Administrator
2. Select target SD card device
3. Select IPFire image file
4. **Ensure "DD Image" mode** is selected (not ISO mode)
5. Verify write completion before removal

**NanoPi R1S (OpenWrt)**:
```
# Extract and flash OpenWrt image
gunzip openwrt-23.05.x-sunxi-cortexa7-friendlyarm_nanopi-r1s-h3-squashfs-sdcard.img.gz
sudo dd if=openwrt-*.img of=/dev/sdX bs=4M status=progress oflag=sync
```

**Raspberry Pi 4B**:
- **Raspberry Pi Imager recommended** for automatic SSH enablement
- Select "Raspberry Pi OS Lite (64-bit)"
- **Enable SSH in advanced options** before flashing
- Configure initial user credentials for security

## Post-Flash Hardware Validation

### Initial Boot Verification

**NanoPi Boards (UART Console Required)**:
```
# Connect UART console at 115200 baud
screen /dev/ttyUSB0 115200

# Expected boot sequence indicators:
# - U-Boot banner and version
# - Kernel loading messages  
# - Systemd service initialization
# - Login prompt appearance
```

**Critical boot milestones**:
- **U-Boot initialization**: Confirms bootloader functionality
- **Kernel mounting**: Verifies filesystem integrity
- **Network interface enumeration**: Confirms hardware detection
- **Service startup**: Indicates successful OS initialization

**Raspberry Pi (SSH or Direct Console)**:
```
# SSH connection
ssh username@raspberrypi.local

# Direct console connection (if HDMI temporarily connected)
# Default credentials: pi/raspberry (change immediately)
```

### Hardware Interface Verification

**Network Interface Confirmation**:
```
# Verify expected network interfaces
ip link show

# Expected interfaces per board:
# R2S: eth0, eth1 (dual Gigabit)
# R1S: eth0, wlan0 (Ethernet + Wi-Fi)  
# Pi 4B: eth0, wlan0 (if Wi-Fi enabled)
```

**GPIO and Hardware Feature Testing**:
```
# Check for hardware-specific features
lscpu                    # Verify CPU architecture
free -h                  # Confirm RAM allocation
lsblk                    # Verify storage detection
dmesg | grep -i error    # Check for hardware errors
```

## Security-Focused Pre-Configuration

### Immediate Security Hardening

**Default Credential Changes** (Critical Priority):
```
# Change default passwords immediately upon first boot
passwd                   # Root password change
passwd pi               # User password change (Raspberry Pi)

# Disable password authentication (SSH key only)
sudo nano /etc/ssh/sshd_config
# Set: PasswordAuthentication no
sudo systemctl restart sshd
```

**Firmware Update Verification**:
```
# Check for immediate security updates
sudo apt update && sudo apt list --upgradable
sudo apt upgrade                # Apply security patches

# NanoPi-specific firmware updates
sudo armbian-config            # Armbian boards only
```

## Common Preparation Issues

### SD Card Compatibility Problems

**Symptom**: Boot failure or filesystem corruption

**Solution**: 
- Verify SD card compatibility with specific SBC
- Test with different card brands/models
- Use high-endurance cards for production deployments

### UART Troubleshooting

**Symptom**: No console output or garbled text

**Common causes**:
- Incorrect baud rate (must be 1,500,000 for NanoPi boards)
- Reversed TX/RX connections
- Ground connection missing
- 5V logic levels (boards require 3.3V)

### Power Inadequacy

**Symptom**: Random reboots, USB device failures, network instability

**Solution**:
- Use official spec power supplies
- Verify power supply current capacity (2.5A minimum)
- Check cable quality (poor USB cables cause voltage drop)

The hardware preparation process here is supposed to establishe a **secure foundation** upon which the security architecture depends.