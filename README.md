## Notes

### Set static ip in VM
Since "private_network" configuration is not supported by Vagrant and Vmware Fusion, we need find workaround.

We can leverage on "base_mac" directive and dhcp mechanism of Vmware fusion.

In vagrant file, assign a static MAC address to target vm.

<pre>
<code>
config.vm.define "control01" do |control01|
  control01.vm.box = settings["software"]["box"]
  control01.vm.hostname = 'control01'
  ...
  control01.vm.provider "vmware_desktop" do |v|
    ...
    v.base_mac = "00:50:56:31:20:90"
  end
  ...
end
</code>
</pre>

Change Vmware fusion vmnet8 (NAT device automßatically created) dhcpd conf file.
<pre>
<code>
sudo vi /Library/Preferences/VMware\ Fusion/vmnet8/dhcpd.conf
</code>
</pre>

Add following configuration to reserve a static ip for vm
<pre>
<code>
# please match the hostname and mac address configured in vagrant file
host control01 {
        hardware ethernet 00:50:56:31:20:90;
        fixed-address 192.168.75.12;
}
</code>
</pre>
