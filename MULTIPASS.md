### Networking - dedicated private subnet for VMs with static private IP addresses

```shell
# admin powershell!!!

# add dedicated Hyper-V virtual switch
New-VMSwitch -SwitchName "multipass" -SwitchType Internal
# get the interface index
Get-NetAdapter | Select-Object -Property Name, InterfaceDescription, ifIndex
# look for "vEthernet (multipass)"
Get-NetAdapter | ? Name -eq "vEthernet (multipass)" | % ifIndex
$multipassIfIndex = Get-NetAdapter | ? Name -eq "vEthernet (multipass)" | % ifIndex

# choosing address = e.g. 10.38.0.0/24 - instructions are built around this address
# you might have VPN connected - check if VPN contains your subnet

# hopefully free
get-netroute | ? { $_.DestinationPrefix.StartsWith("10.38") }
# might be used on VPN - see next hop IP
get-netroute | ? { $_.DestinationPrefix.StartsWith("192.168.0") }

# use free address for our new segment
New-NetIPAddress -IPAddress 10.38.0.0 -PrefixLength 24 -InterfaceIndex $multipassIfIndex

# check if it's there
Get-NetIPAddress -InterfaceAlias "vEthernet (multipass)" | Select-Object IPAddress, PrefixLength

# well done
```

#### How to remove `multipass` Hyper-V vSwitch

```shell
# admin powershell!!!
Remove-VMSwitch -SwitchName Multipass
```