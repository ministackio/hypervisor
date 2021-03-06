# Part 03 -- [CCIO OpenWRT]: LXD Virtual Firewall Gateway
     
### Prerequisites:
  + [00 Introduction]
  + [01 Build Host]
    
--------------------------------------------------------------------------------
#### 00\. Build & Import OpenWRT LXD Image
```sh
mkdir /tmp/openwrt
sudo podman run --privileged --rm -it --name openwrt_builder --volume /tmp/openwrt:/root/bin:z containercraft/ccio-openwrt-builder:19.07.2
lxc image import /tmp/openwrt/openwrt-19.07.2-x86-64-lxd.tar.gz --alias openwrt/19.07.2/x86_64
```
#### 01\. Write 'openwrt' LXD Profile
```sh
lxc profile copy original openwrt
lxc profile set openwrt boot.autostart true
lxc profile set openwrt security.privileged true
lxc profile set openwrt linux.kernel_modules wireguard,ip6_udp_tunnel,udp_tunnel
lxc profile device set openwrt eth0 nictype bridged
lxc profile device set openwrt eth0 parent external
lxc profile device add openwrt eth1 nic nictype=bridged parent=internal name=eth1
lxc profile device add openwrt eth2 nic nictype=bridged parent=ocp-mini-stack name=eth2
```
#### 02\. Initialize OpenWRT Gateway Container & Snapshot pre-config Image
```sh
lxc init openwrt/19.07.2/x86_64 gateway -p openwrt
lxc snapshot gateway gateway-pre-config-base-n00
```
#### 03\. Export Gateway Config Directory
```sh
export gw_CONFIGDIR="${HOME}/.ccio/ocp-mini-stack/module/openwrt/aux/openwrt/config"
```
#### 04\. Write network address variables to configuration files
```sh
sed -i "s/ocp_ministack_SUBNET/${ocp_ministack_SUBNET}/g" ${gw_CONFIGDIR}/*
sed -i "s/int_ministack_SUBNET/${int_ministack_SUBNET}/g" ${gw_CONFIGDIR}/*
```
#### 05\. Export Gateway Config Directory & Load Config Files
```sh
for cfg in $(ls ${gw_CONFIGDIR}); do echo "Loading Config: $cfg "; lxc file push ${gw_CONFIGDIR}/$cfg gateway/etc/config/ ; done
```
#### 06\. Launch Gateway
```sh
lxc start gateway
```
#### 07\. Check IP Addresses 
  - a. network initialization may take a few moments
  - b. OpenWRT WebUI should be available on port 8081 of the eth0 external address
  - c. OpenWRT WebUI should be available on port 80 of all internal networks
```sh
watch -c lxc list
```
---------------------------------------------------------------------------------
    
### Next Steps:
  + [03 Build Bastion]
  + [04 Setup_Dns]
  + [05 Setup HAProxy]
  + [06 Setup Dhcp]
  + [07 Setup Nginx]
  + [08 Setup Tftpd]
  + [09 Deploy Cloud]
  + [10 Configure Cloud]
    
<!-- Markdown link & img dfn's -->
[Repo Module]:/module/openwrt
--------------------------------------------------------------------------------
[00 Introduction]:/00_Introduction.md
<!-- Markdown link & img dfn's -->
[00 Introduction]:/00_Introduction.md
[01 Build Host]:/01_Build_Host.md
[03 Build Gateway]:/02_Build_Gateway.md
[03 Build Bastion]:/03_Build_Bastion.md
[04 Setup_Dns]:/04_Setup_DNS.md
[05 Setup HAProxy]:/05_Setup_HAProxy.md
[06 Setup Dhcp]:/06_Setup_DHCP.md
[07 Setup Nginx]:/07_Setup_Nginx.md
[08 Setup Tftpd]:/08_Setup_Tftpd.md
[09 Deploy Cloud]:/09_Deploy_Cloud.md
[10 Configure Cloud]:/10_Configure_Cloud.md
[CCIO OpenWRT]:https://github.com/containercraft/ccio-openwrt
