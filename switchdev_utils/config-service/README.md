# Switchdev Network Configuration Service 
## Description
This folder contains SR-IOV, OVS and network configuration service for switchdev and VF-LAG  

The configuration listed below shall be performed as part of a systemd script triggered during machine boot and IFCFG files for network configuration and each configuration step is optionally performed on specified interfaces via configuration file:  
- Create VFs  
- Move to switchdev  
- Set VF trust for all VFs  
- Create bond over PFs  
- Create OVS bridges  
- Enable hw-offload in OVS  
- Connect bond or sriov pf to bridge  
- Create sriov_config service
- Create sriov_bond service

## Prerequisites
The below is assumed to be installed/configured on the host
- NICs Firmware up to date (as bundled with MLNX_OFED)
- (MLNX_OFED 5.2 or above ) or (mft 4.16 or above)
- NICs NVCONFIG configures SR-IOV to support VFs per PF
  - Mlxconfig -d <device> set SRIOV_EN=true NUM_OF_VFS=<numOfVfs>
    - Note: reboot required to apply SRIOV firmware configuration
- openvswitch (installed, enabled and running)
- yaml  

## Config yaml file  
The switchdev configuration service config file is a yaml file consists of the following sections:  
- hwOffloadEnabled:  
Configure OVS to enable or disable hardware offload  
- pfs:  
List of SRIOV PF objects which need to be configured, and has the following attributes:  
    - name: The name of the sriov pf interface.  
    - numOfVfs: Number of vfs to be configured on the sriov pf.  
    - switchdev: Boolean, Enable switchdev mode.  
    - vfTrust: Boolean, Configure vf trust.  
    - mtu: Number, configure mtu for sriov_pf (it will be ignored if sriov_pf is a bond slave)
- linuxBonds:  
list of linux bond objects which need to be created and configured  
    - name: The name of the bond interface  
    - slaves: List of bond slaves interfaces (a subset of sriov pfs)  
    - bondingOptions: Bonding options of the bond interface  
    - mtu: Number, configure mtu for bond interface and its slaves
- ovsBridges:  
List of ovs bridges which need to be created and configured  
    - name: The name of ovs bridge  
    - ports: List of ovs bridge ports (a subset of sriov pfs or linux bonds)  

**Note:**  
  - Configure mtu for sriov_pf will be ignored if sriov_pf is a bond slave
  - Configure mtu for bond interface will configure it to its slaves
  - Configure mtu for sriov_pf will set it to the bridge if sriov_pf is ovs port

## Install  
Prepare your own config.yaml file with all the configuration you need and pass it to installation script or you can copy it directly to `/etc/switchdev-config/config.yaml` before execution  
Install will do the following
- create `/etc/switchdev-config directory`    
- copy switchdev_network_config and uninstall.sh to `/etc/switchdev-config` directory  
- optionally copy configuration file to `/etc/switchdev-config directory`    

Run:  
```$ bash ./install.sh [config.yaml]```  

## Execution
Make sure that you have your own config yaml file here `/etc/switchdev-config/config.yaml` then run the script:  
```$ /etc/switchdev-config/switchdev_network_config```  
**Note:**
- You can rerun the script on any configuration change
- Log file is created here `/var/log/switchdev_network_config.log`  


## Uninstall
Uninstall will do the following:
- remove `/etc/switchdev-config/switchdev_network_config` script
- disable and remove sriov_config service  
- disable and remove sriov_bind service  

Run:  
```$ bash /etc/switchdev-config/uninstall.sh```
**Note:**
- SR-IOV PF configuration will reset after reboot
- Network configuration (bonds and bridges) will stay after reboot unless you clean them manually:
  - Removing relevant network-scripts in `/etc/sysconfig/network-scripts/`  
  - Deleting ovs_bridges using ```ovs-vsctl del-br <br-name>```  