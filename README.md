# IoTEdge-SoC_FPGA
[Azure IoT Edge](https://docs.microsoft.com/en-us/azure/iot-edge/?wt.mc_id=IoTEdge-SoC_FPGA-github-pdecarlo) Module for controlling an [Intel® Cyclone® V SoC FPGA](https://www.intel.com/content/www/us/en/products/programmable/fpga/cyclone-v.html)

![Diagram](https://dmtyylqvwgyxw.cloudfront.net/instances/132/uploads/images/custom_image/image/40663/normal_blob?v=1559928208)

## Purpose
To enable cloud deployment of FPGA configurations via Raw Binary Files (.rbf) to remote devices.

## Video Demonstration

[![IoTEdge SoC FPGA Demonstration](https://img.youtube.com/vi/puXh0V_DZ4Y/0.jpg)](https://www.youtube.com/watch?v=puXh0V_DZ4Y)

## Supported Devices
* [Terasic De10-Nano](http://www.terasic.com.tw/cgi-bin/page/archive.pl?Language=English&No=1046)

(Note: porting to other Cyclone V enabled hardware should be straightforwad but is not guaranteed)

## How it works
[Azure IoT Edge](https://docs.microsoft.com/en-us/azure/iot-edge/?wt.mc_id=IoTEdge-SoC_FPGA-github-pdecarlo) enables developers to deploy containerized modules to internet connected devices which allows for maintaining a desired state of running services through cloud-configured deployment configurations.  This mechanism also offers the ability to securely update running modules on devices remotely via changes to this configuration.  

 This IoT Edge module leverages work from [@nhasbun](https://github.com/nhasbun/de10nano_fpga_linux_config) to configure the FPGA portion of the Cyclone V SoC from Linux within an Iot Edge module, allowing for a robust deployment mechanism for shipping FPGA configurations to remote devices at scale.

 This is accomplished using direct register access to take control of the FPGA Manager device which is used by the HPS to configure the FPGA. A multi-stage Dockerfile builds this component using SoCEDS-18.1.0.625 and provides it to an IoT Edge module which maps /dev/mem from the host to allow for interaction with the FPGA from the containerized module.

## Kernel / OS Requirements
 An IoT Edge Compatible kernel / OS needs to be installed onto the target device.  Microsoft officially provides .deb packages for IoTEdge for ARM, as such your OS choice needs to be Debian compatible.

You will want to ensure that your device is connected to internet via the ethernet port and obtain a serial connection by following these [instructions](https://software.intel.com/en-us/node/731116).

Once connected to your device, ensure that is up to date and has curl installed with:

```
sudo apt update
sudo apt upgrade
sudo apt install curl
```

IoT Edge itself depends on an installation of Moby, to verify that your FPGA device's kernel can support this, run and view the output of:

`curl -sSL https://raw.githubusercontent.com/moby/moby/master/contrib/check-config.sh | bash`

This will produce an output similar to the following:

```
info: reading kernel config from /proc/config.gz ...

Generally Necessary:
- cgroup hierarchy: properly mounted [/sys/fs/cgroup]
- CONFIG_NAMESPACES: enabled
- CONFIG_NET_NS: enabled
- CONFIG_PID_NS: enabled
- CONFIG_IPC_NS: enabled
- CONFIG_UTS_NS: enabled
- CONFIG_CGROUPS: enabled
- CONFIG_CGROUP_CPUACCT: enabled
- CONFIG_CGROUP_DEVICE: enabled
- CONFIG_CGROUP_FREEZER: enabled
- CONFIG_CGROUP_SCHED: enabled
- CONFIG_CPUSETS: enabled
- CONFIG_MEMCG: enabled
- CONFIG_KEYS: enabled
- CONFIG_VETH: enabled (as module)
- CONFIG_BRIDGE: enabled (as module)
- CONFIG_BRIDGE_NETFILTER: enabled (as module)
- CONFIG_NF_NAT_IPV4: enabled (as module)
- CONFIG_IP_NF_FILTER: enabled (as module)
- CONFIG_IP_NF_TARGET_MASQUERADE: enabled (as module)
- CONFIG_NETFILTER_XT_MATCH_ADDRTYPE: enabled (as module)
- CONFIG_NETFILTER_XT_MATCH_CONNTRACK: enabled (as module)
- CONFIG_NETFILTER_XT_MATCH_IPVS: enabled (as module)
- CONFIG_IP_NF_NAT: enabled (as module)
- CONFIG_NF_NAT: enabled (as module)
- CONFIG_NF_NAT_NEEDED: enabled
- CONFIG_POSIX_MQUEUE: enabled

Optional Features:
- CONFIG_USER_NS: enabled
- CONFIG_SECCOMP: enabled
- CONFIG_CGROUP_PIDS: enabled
- CONFIG_MEMCG_SWAP: enabled
- CONFIG_MEMCG_SWAP_ENABLED: enabled
    (cgroup swap accounting is currently enabled)
- CONFIG_BLK_CGROUP: enabled
- CONFIG_BLK_DEV_THROTTLING: missing
- CONFIG_IOSCHED_CFQ: enabled
- CONFIG_CFQ_GROUP_IOSCHED: enabled
- CONFIG_CGROUP_PERF: enabled
- CONFIG_CGROUP_HUGETLB: missing
- CONFIG_NET_CLS_CGROUP: missing
- CONFIG_CGROUP_NET_PRIO: enabled
- CONFIG_CFS_BANDWIDTH: enabled
- CONFIG_FAIR_GROUP_SCHED: enabled
- CONFIG_RT_GROUP_SCHED: enabled
- CONFIG_IP_NF_TARGET_REDIRECT: enabled (as module)
- CONFIG_IP_VS: enabled (as module)
- CONFIG_IP_VS_NFCT: enabled
- CONFIG_IP_VS_PROTO_TCP: enabled
- CONFIG_IP_VS_PROTO_UDP: enabled
- CONFIG_IP_VS_RR: enabled (as module)
- CONFIG_EXT4_FS: enabled
- CONFIG_EXT4_FS_POSIX_ACL: enabled
- CONFIG_EXT4_FS_SECURITY: enabled
- Network Drivers:
  - "overlay":
    - CONFIG_VXLAN: enabled (as module)
      Optional (for encrypted networks):
      - CONFIG_CRYPTO: enabled
      - CONFIG_CRYPTO_AEAD: enabled
      - CONFIG_CRYPTO_GCM: enabled
      - CONFIG_CRYPTO_SEQIV: enabled
      - CONFIG_CRYPTO_GHASH: enabled
      - CONFIG_XFRM: enabled
      - CONFIG_XFRM_USER: enabled (as module)
      - CONFIG_XFRM_ALGO: enabled (as module)
      - CONFIG_INET_ESP: enabled (as module)
      - CONFIG_INET_XFRM_MODE_TRANSPORT: enabled (as module)
  - "ipvlan":
    - CONFIG_IPVLAN: missing
  - "macvlan":
    - CONFIG_MACVLAN: missing
    - CONFIG_DUMMY: missing
  - "ftp,tftp client in container":
    - CONFIG_NF_NAT_FTP: enabled (as module)
    - CONFIG_NF_CONNTRACK_FTP: enabled (as module)
    - CONFIG_NF_NAT_TFTP: enabled (as module)
    - CONFIG_NF_CONNTRACK_TFTP: enabled (as module)
- Storage Drivers:
  - "aufs":
    - CONFIG_AUFS_FS: missing
  - "btrfs":
    - CONFIG_BTRFS_FS: enabled
    - CONFIG_BTRFS_FS_POSIX_ACL: enabled
  - "devicemapper":
    - CONFIG_BLK_DEV_DM: missing
    - CONFIG_DM_THIN_PROVISIONING: missing
  - "overlay":
    - CONFIG_OVERLAY_FS: enabled
  - "zfs":
    - /dev/zfs: missing
    - zfs command: missing
    - zpool command: missing

Limits:
- /proc/sys/kernel/keys/root_maxkeys: 1000000

```

You will want to ensure that you have all of the items under `Generally Necessary` and `Network Drivers: - "overlay"` marked as enabled.

If these are not enabled, then you will need to refer to the source code of your kernel to enable these features.

If you need to compile and setup a new kernel / OS from source, there are instructions avaialable @ https://www.digikey.com/eewiki/display/linuxonarm/DE10-Nano+Kit.  Please note that following these steps without modification will produce an image that lacks CONFIG_BRIDGE, CONFIG_VETH, CONFIG_VXLAN modules which are required by Moby.  

The missing modules can be enabled during the execution of `./build_kernel.sh` in the associated steps for building the Linux Kernel.  This will produce a menuconfig prompt that allows for enabling / disabling key kernel configuration options

[CONFIG_BRIDGE](https://cateee.net/lkddb/web-lkddb/BRIDGE.html) can be enabled under `Networking support > Networking options - 802.1d Ethernet Bridging`

[CONFIG_VETH](https://cateee.net/lkddb/web-lkddb/VETH.html) can be enabled under `Device Drivers > Network device support - Virtual ethernet pair device`

[CONFIG_VXLAN](https://cateee.net/lkddb/web-lkddb/VXLAN.html) can be enabled under `Device Drivers > Network device support - Virtual eXtensible Local Area Network (VXLAN)`

When you have confirmed that you have satisfied the requirements for Moby, install it onto your device with the following:

```
# You can copy the entire text from this code block and 
# paste in terminal. The comment lines will be ignored.

# Download and install the moby-engine
curl -L https://aka.ms/moby-engine-armhf-latest -o moby_engine.deb && sudo dpkg -i ./moby_engine.deb

# Download and install the moby-cli
curl -L https://aka.ms/moby-cli-armhf-latest -o moby_cli.deb && sudo dpkg -i ./moby_cli.deb

# Run apt-get fix
sudo apt-get install -f
```

After a few minutes you should be able to verify that Moby has installed properly by running:

``` 
systemctl status docker
```

Next we will install the IoT Edge security daemon and IoT Edge runtime with:

```
# You can copy the entire text from this code block and 
# paste in terminal. The comment lines will be ignored.

# Download and install the standard libiothsm implementation
curl -L https://aka.ms/libiothsm-std-linux-armhf-latest -o libiothsm-std.deb && sudo dpkg -i ./libiothsm-std.deb

# Download and install the IoT Edge Security Daemon
curl -L https://aka.ms/iotedged-linux-armhf-latest -o iotedge.deb && sudo dpkg -i ./iotedge.deb

# Run apt-get fix
sudo apt-get install -f

# Symlink crypto and ssl libraries
sudo ln -s /usr/lib/arm-linux-gnueabihf/libcrypto.so.1.0.0 /lib/arm-linux-gnueabihf/libcrypto.so.1.0.2 

sudo ln -s /usr/lib/arm-linux-gnueabihf/libssl.so.1.0.0 /lib/arm-linux-gnueabihf/libssl.so.1.0.2
```
Next, follow these [steps](https://docs.microsoft.com/en-us/azure/iot-edge/how-to-install-iot-edge-linux-arm#option-1-manual-provisioning) to manually configure IoT Edge to connect to your Azure IoT Hub.

Once you have completed these steps, verify that the iotedge service has started successfully with:

```
sudo systemctl status iotedge
```

## Deploy the DE10Nano_RBF_Loader Module

Requirements:
* [Visual Studio Code](https://code.visualstudio.com/)
* [Azure IoT Edge Extension for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-edge?wt.mc_id=RetroArchAIwithIoTEdge-github-pdecarlo)


Create a deployment for the IoT Edge device by right-clicking `deployment.template.json` and select `Generate IoT Edge Deployment Manifest`.  This will create a file under the config folder named `deployment.arm32v7.json`, right-click that file and select `Create Deployment for Single Device` then select the registered device in your IoT Hub which represents the DE10-Nano device.

The project ships with an .rbf file (fpga_config_file.rbf) that acts as a basic XOR gate using the physical switches on the DE10-Nano.  When SW0 or SW1 are exclusively switched on, an onboard LED will light up.  If you are curious how this .rbf was created, you may refer to this [tutorial](http://hamblen.ece.gatech.edu/DE2/DE2_tutorials/tut_quartus_intro_vhdl.pdf).

To deploy your own .rbf, edit the `DE10Nano_RBF_Loader/module.json` to point to a docker repostory that you control, next overwrite the existing fpga_config_file.rbf with your intended .rbf file, then right-click `deployment.template.json` and select `Build and Push IoT Edge Solution`. This will update the deployment file under the config folder named `deployment.arm32v7.json`, right-click that file and select `Create Deployment for Single Device` then select the registered device in your IoT Hub which represents the DE10-Nano device.  Once the IoT Edge deployment completes, your .rbf will load into the FPGA on the target device.

You can confirm that the FPGA has been configured by running:

`sudo iotedge logs DE10Nano_RBF_Loader`

This should produce output similar to the following: 
```
******************************************************
MSEL Pin Config..... 0xa
FPGA State.......... Powered Off
cfgwdth Register.... 0x1
cdratio Register.... 0x0
axicfgen Register... 0x0
Nconfig pull reg.... 0x0
CONF DONE........... 0x0
Ctrl.en?............ 0x0
******************************************************
Turning FPGA Off.
Setting cdratio with 0x3.
Turning FPGA On.
Loading rbf file.
EOF reached.
******************************************************
MSEL Pin Config..... 0xa
FPGA State.......... User Phase
cfgwdth Register.... 0x1
cdratio Register.... 0x3
axicfgen Register... 0x0
Nconfig pull reg.... 0x0
CONF DONE........... 0x0
Ctrl.en?............ 0x0
******************************************************
```
