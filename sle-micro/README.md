# SLE-Micro on libvirt 

Here are steps and commands I used to create KVM VM using SLE-Micro v5.3 pre-built Image from SUSE.

## Convert original SLE-Micro raw image to qcow2 format
```
sudo qemu-img convert -f raw -O qcow2 SLE-Micro.x86_64-5.3.0-Default-GM.raw SLE-Micro.qcow2
```

## Resize the qcow2 and make it to a size you need. 
Here I add 40GB on top. Once the image will be booted SLE-Micro will take the entire size and partition it accordingly by using btrfs filesystem. Of course you can also use ignition to make your favorite partitioning.
```
sudo qemu-img resize my-slemicro-usb.qcow2 +40G
```

## Create a new disk image that uses the original disk image as a backing store
This trick helps me to start over again if something went wrong but the backing store qcow2 file has already the correct size etc.
```
sudo qemu-img create -f qcow2 -F qcow2 -b SLE-Micro.qcow2 my-slemicro.qcow2
```
## Create the VM and modify the libvirt xml
```
sudo virt-install --connect qemu:///system \
             --import \
             --name testvm \
             --ram 4096 --vcpus 4 \
             --os-type=linux \
             --os-variant=opensuse15.3 \
             --disk path=/data/sm1/my-slemicro.qcow2,format=qcow2,bus=virtio \
             --disk path=/data/sm1/ignition-disk.qcow2,format=qcow2,bus=virtio \
             --vnc --noautoconsole \
             --print-xml > testvm.xml
```
__The ```path=/data/sm1/ignition-disk.qcow2``` is the disk with label ignition to be used for first boot configurations.

The testvm.xml is the output that can be used to create the VM using virsh command-line.
```
sudo virsh define testvm.xml
```
>Note: You might need to delete the mac_address in the xml file if you reuse the xml file to create other VMs to avoid same mac address used in other VMs.

Start the VM either using virt-manager UI or commandline.
```
sudo virsh start testvm
```

**Now we continue to setup RKE2 on the first sle-micro node.**

