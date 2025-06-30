# Physical Virtualization Server Setup

## **Foreword**
This guide sets up a physical Debian 12 system with KDE Plasma desktop environment and KVM/QEMU virtualization platform for running the lab topology, feel free to use another distro (some commands can be different).


## **Requirements**
- **CPU**: x86_64 with virtualization support
- **RAM**: 16GB minimum (32GB+ recommended)
- **Storage**: 500GB SSD minimum
- **Network**: Ethernet adapter with VLAN Support / more is better!
- **BIOS/UEFI**: VT-x/AMD-V and VT-d/IOMMU must be enabled


## **System Setup**
Get the latest Debian netinst ISO file and install on Virtualization Server
[Download Debian ISO](https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/)

```bash
# switch to root user
su -

# Set hostname to "pve"
hostnamectl set-hostname pve

# Update /etc/hosts
sed --in-place "s/127.0.1.1.*/127.0.1.1\tpve/" /etc/hosts

# Set timezone
timedatectl set-timezone Europe/Berlin

# Change interface naming
sed --in-place 's/^GRUB_CMDLINE_LINUX=.*/GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0"/' /etc/default/grub
grub-mkconfig --output /boot/grub/grub.cfg

# System update
apt update && apt upgrade --yes

# Install essential packages
apt install --yes nmap tcpdump wireshark traceroute dnsutils whois telnet curl wget net-tools bridge-utils vlan ethtool htop python3 python3-venv python3-pip nano build-essential git make cmake gcc g++ nano iproute2 kde-plasma-desktop plasma-nm plasma-pa konsole dolphin kate kde-spectacle plasma-discover ovmf qemu-system-x86 qemu-utils qemu-block-extra iptables netfilter-persistent

# reboot PC so interface name changes apply
reboot
```
Check for virtualization support - Should return a number > 0
```bash
grep --extended-regexp --count '(vmx|svm)' /proc/cpuinfo
```

### Directory structure
Create directories for VM storage
```bash
mkdir --parents /vm
mkdir --parents /vm/iso
mkdir --parents /vm/adminpc

mkdir --parents /vm/fw01
mkdir --parents /vm/fw02
mkdir --parents /vm/xp01
mkdir --parents /vm/win7
mkdir --parents /vm/dc01
mkdir --parents /vm/dc02
mkdir --parents /vm/ex01
mkdir --parents /vm/ex02
mkdir --parents /vm/sw01
mkdir --parents /vm/sw02
mkdir --parents /vm/fs01
mkdir --parents /vm/fs02
mkdir --parents /vm/ls01
mkdir --parents /vm/ls02
mkdir --parents /vm/ws01
mkdir --parents /vm/ws02
mkdir --parents /vm/web01
mkdir --parents /vm/web02
mkdir --parents /vm/proxy01
mkdir --parents /vm/proxy02
mkdir --parents /vm/db01
mkdir --parents /vm/db02
mkdir --parents /vm/log01
mkdir --parents /vm/victim01
```

### Set permissions
Assuming main user is `admin`
```bash
chown --recursive admin:kvm /vm
usermod --append --groups wireshark admin
usermod --append --groups kvm admin
```
## **Network Setup**

```bash
# Backup current network configuration
cp /etc/network/interfaces /etc/network/interfaces.backup

# Assuming main gateway interface name is eth0
tee /etc/network/interfaces << 'EOF'
auto lo
iface lo inet loopback

# using eth0 for bridge - this interface name can be different in your setup
auto br0
iface br0 inet dhcp
bridge_ports eth0
bridge_stp off
bridge_fd 0
bridge_maxwait 0
EOF
```

### Enable persistent Forwarding
```bash
tee --append /etc/sysctl.conf << 'EOF'
# Enable IP forwarding for VMs
net.ipv4.ip_forward = 1
net.ipv6.conf.all.forwarding = 1
EOF

# Apply sysctl changes
sysctl --load
```

## **Next Steps**

After completing this setup, you'll have a fully functional virtualization host ready for implementing the lab topology defined in [Topology](topology.md).
