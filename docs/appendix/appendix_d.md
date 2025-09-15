### Configuration File Templates

**D.1 OpenWrt Wireless Configuration**
```
config wifi-device 'radio0'
    option type 'mac80211'
    option path 'platform/soc/1c30000.mmc/mmc_host/mmc1/mmc1:0001/mmc1:0001:1'
    option channel '6'
    option htmode 'HT20'
    option disabled '0'

config wifi-iface 'default_radio0'
    option device 'radio0'
    option network 'iot_zone'
    option mode 'ap'
    option ssid 'JASON-IoT'
    option encryption 'sae'
    option key 'SecurePassphrase2024!'
    option ieee80211w '2'
    option isolate '1'
```

**D.2 IPFire Zone Configuration**
```
# Network zone definitions
GREEN_DEV=eth1
GREEN_ADDRESS=192.168.1.1
GREEN_NETMASK=255.255.255.0

ORANGE_DEV=eth1.100
ORANGE_ADDRESS=192.168.100.1
ORANGE_NETMASK=255.255.255.0

RED_DEV=eth0
RED_TYPE=DHCP
```

**D.3 Docker Compose Services Definition**
```yaml
version: '3.8'
services:
  unbound:
    image: mvance/unbound:latest
    container_name: jason-unbound
    ports:
      - "192.168.200.100:53:53/udp"
    volumes:
      - ./config/unbound.conf:/opt/unbound/etc/unbound/unbound.conf:ro
    restart: unless-stopped
    mem_limit: 256m
```