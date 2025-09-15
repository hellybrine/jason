# OpenWrt Configuration: Access Plane Security

The NanoPi R1S access plane configuration transforms a basic OpenWrt installation into a **cryptographically-enforced wireless gateway** that implements zero-trust device onboarding. This configuration eliminates the traditional wireless security model where authentication success automatically grants network privileges.

## Initial System Access and Preparation

### LAN-Based Configuration Access
**Security rationale**: Initial configuration occurs through wired connection to prevent wireless eavesdropping during credential setup and security policy establishment.

**Physical connection setup**:
1. Connect NanoPi R1S Ethernet port to laptop/workstation
2. Configure laptop network interface to static IP: `192.168.1.100/24`
3. Access OpenWrt web interface: `http://192.168.1.1`
4. Default credentials: `root` (no password initially)

**Immediate security hardening**:
```
# SSH to device for command-line configuration
ssh root@192.168.1.1

# Set root password immediately
passwd

# Update package lists and install essential packages
opkg update
opkg install wpad-wolfssl hostapd-utils
```

### Package Requirements for WPA3 and RADIUS
**Critical security note**: Default OpenWrt installations often include `wpad-basic` which lacks WPA3 support. Enterprise-grade authentication requires specific package combinations.

```
# Remove basic wireless daemon
opkg remove wpad-basic wpad-basic-wolfssl

# Install full-featured wireless daemon with WPA3 support
opkg install wpad-wolfssl

# Install RADIUS client functionality
opkg install freeradius3-utils

# Verify WPA3 capability
hostapd -v | grep -E "(WPA3|SAE)"
```

## Wireless Interface Configuration

### Radio Hardware Configuration
**Security principle**: Minimize attack surface by disabling unnecessary wireless features and optimizing for security over performance.

```
# Configure radio parameters via UCI
uci set wireless.radio0.disabled='0'
uci set wireless.radio0.country='US'          # Set appropriate country code
uci set wireless.radio0.channel='6'           # Fixed channel prevents frequency agility attacks
uci set wireless.radio0.htmode='HT20'         # Disable channel bonding for better range/security
uci set wireless.radio0.txpower='20'          # Reduce transmission power to limit signal range
uci set wireless.radio0.legacy_rates='0'      # Disable legacy rates that lack security features
```

**Channel selection security considerations**:
- **Fixed channels** prevent frequency-hopping attacks that could bypass monitoring
- **Non-overlapping channels** (1, 6, 11 in 2.4GHz) reduce interference and improve detection capabilities
- **Lower power settings** limit physical attack range while maintaining coverage for legitimate devices

### Access Point Interface Creation
**Architecture principle**: Create isolated wireless interface that enforces authentication before network integration.

```
# Create secure AP interface
uci set wireless.@wifi-iface=wifi-iface
uci set wireless.@wifi-iface.device='radio0'
uci set wireless.@wifi-iface.mode='ap'
uci set wireless.@wifi-iface.network='iot_zone'      # Separate from LAN network
uci set wireless.@wifi-iface.ssid='JASON-IoT'
uci set wireless.@wifi-iface.hidden='1'              # Hide SSID for operational security
uci set wireless.@wifi-iface.isolate='1'             # Enable client isolation
uci set wireless.@wifi-iface.disabled='0'
```

**Network isolation rationale**: The `iot_zone` network prevents authenticated devices from directly accessing the configuration network, enforcing traffic routing through the segmentation plane.

## WPA3-SAE Implementation

### Cryptographic Security Configuration
**Security objective**: Implement WPA3-SAE (Simultaneous Authentication of Equals) to eliminate offline dictionary attacks and provide forward secrecy.

```
# Configure WPA3-SAE encryption
uci set wireless.@wifi-iface.encryption='sae'
uci set wireless.@wifi-iface.key='ComplexPassphrase2024!'
uci set wireless.@wifi-iface.ieee80211w='2'           # Mandatory Management Frame Protection
uci set wireless.@wifi-iface.sae_require_mfp='1'     # Require MFP for SAE
uci set wireless.@wifi-iface.wpa_disable_eapol_key_retries='1'  # KRACK mitigation
```

**Management Frame Protection (MFP) significance**: 
- **Prevents deauthentication attacks** that could force devices to reconnect and leak credentials
- **Protects association/disassociation frames** from spoofing attacks
- **Required by WPA3 specification** for security compliance

### Mixed-Mode Compatibility (Optional)
**Use case**: When legacy device support is required while maintaining maximum security for capable devices.

```
# Configure WPA2/WPA3 mixed mode (use with caution)
uci set wireless.@wifi-iface.encryption='sae-mixed'
uci set wireless.@wifi-iface.ieee80211w='1'           # Optional MFP for backward compatibility
```

**Security trade-off**: Mixed mode reduces security to the lowest common denominator. Pure WPA3-SAE is preferred for Jason's security model.

## RADIUS Integration for Identity-Based Access Control

### RADIUS Client Configuration
**Security architecture**: Bind network access to centralized identity verification rather than shared credentials.

```
# Configure RADIUS authentication
uci set wireless.@wifi-iface.encryption='wpa2'        # WPA2-Enterprise base
uci set wireless.@wifi-iface.server='192.168.2.100'   # RADIUS server IP (Services plane)
uci set wireless.@wifi-iface.port='1812'              # Standard RADIUS authentication port
uci set wireless.@wifi-iface.key='SharedRADIUSSecret' # RADIUS shared secret

# Configure additional RADIUS parameters
uci set wireless.@wifi-iface.auth_algs='1'            # Open system authentication only
uci set wireless.@wifi-iface.wpa='2'                  # WPA2 protocol
uci set wireless.@wifi-iface.rsn_pairwise='CCMP'      # AES-CCMP encryption
```

### MAC Address-Based Pre-Authentication
**Security layer**: Combine 802.1X authentication with MAC address validation for defense-in-depth.

```
# Enable MAC address filtering with RADIUS integration
uci set wireless.@wifi-iface.macfilter='2'            # Allow only listed MACs
uci set wireless.@wifi-iface.maclist='aa:bb:cc:dd:ee:ff'  # Example trusted device

# Configure dynamic VLAN assignment (advanced)
uci set wireless.@wifi-iface.dynamic_vlan='1'
uci set wireless.@wifi-iface.vlan_file='/etc/hostapd.vlan'
```

**RADIUS attribute mapping**: The RADIUS server can return VLAN assignments and access policies based on device identity, enabling granular per-device authorization.

## Advanced Security Configuration

### Client Isolation and Traffic Control
**Security principle**: Prevent lateral movement between authenticated devices at Layer 2.

```
# Configure comprehensive client isolation
uci set wireless.@wifi-iface.isolate='1'              # Basic client isolation
uci set wireless.@wifi-iface.ap_isolate='1'           # AP-level isolation
uci set wireless.@wifi-iface.multicast_to_unicast='1' # Convert multicast to unicast

# Rate limiting for DoS prevention
uci set wireless.@wifi-iface.maxassoc='50'            # Limit concurrent associations
uci set wireless.@wifi-iface.max_inactivity='300'     # 5-minute inactivity timeout
```

### SSID Visibility and Operational Security
**Security consideration**: Hidden SSIDs provide operational security benefits while creating minimal security overhead.

```
# Configure SSID hiding with security optimizations
uci set wireless.@wifi-iface.hidden='1'               # Hide SSID in beacons
uci set wireless.@wifi-iface.ignore_broadcast_ssid='1' # Ignore broadcast probe requests
uci set wireless.@wifi-iface.disable_dgaf='1'         # Disable group-addressed forwarding

# Beacon and probe response optimization
uci set wireless.@wifi-iface.beacon_int='100'         # Standard beacon interval
uci set wireless.@wifi-iface.dtim_period='2'          # DTIM period for power management
```

**Hidden SSID operational note**: While not a security control against determined attackers, SSID hiding reduces automated attack tool effectiveness and limits reconnaissance information leakage[11].

### Wireless Security Hardening
**Objective**: Eliminate protocol-level attack vectors and reduce attack surface.

```
# Disable vulnerable legacy features
uci set wireless.radio0.legacy_rates='0'                 # No legacy 802.11b rates
uci set wireless.radio0.require_mode='n'                 # Require 802.11n minimum
uci set wireless.@wifi-iface.wps_pushbutton='0'       # Disable WPS completely
uci set wireless.@wifi-iface.wds='0'                  # Disable WDS bridging

# Configure robust security parameters
uci set wireless.@wifi-iface.wpa_group_rekey='3600'   # Hourly group key rotation
uci set wireless.@wifi-iface.wpa_pairwise_rekey='3600' # Hourly pairwise key rotation
uci set wireless.@wifi-iface.wpa_gmk_rekey='86400'    # Daily group master key rotation
```

## Network Interface Binding and VLAN Configuration

### IoT Zone Network Creation
**Architecture requirement**: Create dedicated network segment for IoT devices that routes through segmentation plane.

```
# Create IoT zone network interface
uci set network.iot_zone=interface
uci set network.iot_zone.proto='static'
uci set network.iot_zone.ipaddr='192.168.100.1'
uci set network.iot_zone.netmask='255.255.255.0'
uci set network.iot_zone.delegate='0'                    # Disable IPv6 delegation

# Configure DHCP for IoT zone
uci set dhcp.iot_zone=dhcp
uci set dhcp.iot_zone.interface='iot_zone'
uci set dhcp.iot_zone.start='100'
uci set dhcp.iot_zone.limit='150'
uci set dhcp.iot_zone.leasetime='12h'
uci set dhcp.iot_zone.dhcp_option='6,192.168.2.100'      # Point to services plane DNS
```

### VLAN Tagging for Segmentation Integration
**Traffic flow control**: Tag all IoT traffic for proper routing through segmentation plane.

```
# Configure VLAN tagging for segmentation plane integration
uci set network.iot_zone.type='8021q'
uci set network.iot_zone.ifname='eth0.100'               # VLAN 100 for IoT traffic
uci set network.iot_zone.vid='100'

# Configure bridge for wireless-to-VLAN mapping
uci set network.br_iot=device
uci set network.br_iot.name='br-iot'
uci set network.br_iot.type='bridge'
uci set network.br_iot.ports='eth0.100'
```

## Configuration Validation and Testing

### Wireless Interface Verification
```
# Apply all configuration changes
uci commit wireless
uci commit network
uci commit dhcp

# Restart wireless services
wifi down
wifi up

# Verify wireless interface status
iwconfig
iwlist wlan0 scan | grep -E "(ESSID|Encryption)"
hostapd_cli -i wlan0 status
```

### Security Configuration Audit
```
# Verify WPA3-SAE configuration
hostapd_cli -i wlan0 get_config | grep -E "(wpa|sae|ieee80211w)"

# Check client isolation
bridge fdb show br-iot

# Verify RADIUS connectivity (if configured)
echo "Message-Authenticator = 0x00, User-Name = \"test\"" | \
radclient -x 192.168.2.100:1812 auth SharedRADIUSSecret
```

### Operational Security Validation
```
# Monitor authentication attempts
logread -f | grep -E "(hostapd|wpa)"

# Verify traffic isolation
tcpdump -i br-iot -n icmp

# Check for security violations
dmesg | grep -E "(wifi|hostapd|authentication)"
```

## Web Interface Configuration (LuCI Alternative)

For administrators preferring graphical configuration:

1. **Navigate to Network → Wireless**
2. **Click "Edit" on the radio interface**
3. **General Setup**:
   - ESSID: `JASON-IoT`
   - Mode: `Access Point`
   - Network: Select `iot_zone`
4. **Wireless Security**:
   - Encryption: `WPA3-SAE` or `WPA2-EAP` (for RADIUS)
   - Key/Password: Enter strong passphrase
   - 802.11w Management Frame Protection: `Required`[13]
5. **Advanced Settings**:
   - Hide SSID: ✓ Enabled
   - Client Isolation: ✓ Enabled
   - Maximum Associations: `50`

## Security Testing and Validation

### Penetration Testing Checkpoints
```
# Test WPA3-SAE resistance to offline attacks
# (Requires specialized tools - cannot be performed with standard dictionaries)

# Verify Management Frame Protection
# (MFP-enabled networks should resist deauth attacks)

# Test client isolation effectiveness
# (Clients should not be able to reach each other)

# Validate RADIUS integration
# (Authentication should fail for unauthorized devices)
```

The OpenWrt access plane configuration creates a **cryptographically-enforced trust boundary** where wireless proximity does not imply network access privileges. Each device must prove its authorization through multiple authentication factors before receiving any network connectivity, establishing the foundation for Jason's zero-trust architecture.