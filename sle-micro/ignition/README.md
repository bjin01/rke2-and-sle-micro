# Create qcow2 disk with ignition & combustion files for sle-micro

## Create a ignition yaml file
![sle-micro.fcc](./ignition/sle-micro.fcc)

## Create or convert the ignition yaml to ignition json file
I use podman and butane container to convert yaml to json for ignition.
```
sudo podman run --interactive --rm quay.io/coreos/butane:release --pretty --strict < ignition/slemicro.fcc > ignition/provision.ign
```

https://docs.fedoraproject.org/en-US/fedora-coreos/producing-ign/

## Create combustion script
Combustion is a dracut module that enables you to configure your system on its first boot. Combustion reads a provided file called script and executes commands in it and thus performs changes to the file system. You can use Combustion to change the default partitions, set users' passwords, create files, install packages, etc.

The script file must be stored in directory combustion and the file name must be "script".

## Create a disk (qcow2) with label "ignition" 
Once the SLE-Micro pre-built image boots for first-time and if it finds a disk/usb/DVD with label "ignition" then it will apply the configurations from the config.ign files but also if combustion script if combustion directory is found along side ignition.

```
sudo qemu-img create -f qcow2 ignition.qcow2 100M
```
Now we need to partition this disk image and copy our ignition and combustion files to it.
For that we use qemu-nbd which needs nbd kernel module to be loaded first:
```
sudo modprobe nbd max_part=10
```
Now we connect a nbd device with the qcow2 image file.
```
sudo qemu-nbd -c /dev/nbd1 ignition.qcow2
```
Then, use ```fdisk /dev/nbd1``` to create one linux partition using the entire device.

After fdisk we create a filesystem on this newly created linux partition, assuming the partition device is /dev/nbd0p1
```
sudo mkfs.ext4 /dev/nbd0p1
```

Now we label the disk with "ignition" as label name:
```
sudo mount /dev/nbd0p1 /mnt
```
And we copy the ignition and combustion files to the mounted directory:
```
sudo cp -r ./ignition /mnt/ 
sudo cp -r ./combustion /mnt/
```
Now umount the device and disconnect nbd device:
```
sudo umount /mnt 
sudo qemu-nbd -d /dev/nbd0p1
```

Now the ignition.qcow2 is ready to be used with SLE-Micro booting.