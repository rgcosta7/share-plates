# USB Devices as VMFS Datastore in vSphere ESXi 7.0

## Step 1 - Disable usbarbitrator service

- Connect to the ESXi host with SSH

- Stop the USB arbitrator service.

~~~bash
~ # /etc/init.d/usbarbitrator stop
~~~

- Permanently disable the USB arbitrator service after reboot.

~~~bash
~ # chkconfig usbarbitrator off
~~~

No reboot is required


## Step 2 - Create VMFS Datastore

- Plug in the USB device to your ESXi host.

- While connecting, check /var/log/vmkernel.log for errors. It should been registered using the following message:

~~~bash
ScsiDevice: 5745: Successfully registered device "mpx.vmhba34:C0:T0:L0" from plugin "NMP" of type 0
~~~

# Create VMFS Datastore on USB drives

- (Assuming that the Device ID is naa.5000000000000001)

- Connect to the ESXi host with SSH

- Plug in the USB device to your ESXi host. While connecting the USB device you can either watch /var/log/vmkernel.log to identify the device name or identify it within /vmfs/devices/disks.

~~~bash
~ # ls /vmfs/devices/disks
~~~

- Write a GPT label to the device:

~~~bash
~ # partedUtil mklabel /vmfs/devices/disksnaa.5000000000000001 gpt
~~~
  
- To create a partition you need to know the start sector, end sector, which depends on the device size and the GUID.
  The start sector is always 2048
  The GUID for VMFS is AA31E02A400F11DB9590000C2911D1B8
  The end sector can be calculated with the following formula (Use the numbers from getptbl):

~~~bash
~ # partedUtil getptbl /vmfs/devices/disks/naa.5000000000000001
gpt
15566 255 63 250069680
~~~

- You can also calculate the end sector with the following command:

~~~bash
~ # eval expr $(partedUtil getptbl /vmfs/devices/disks/naa.5000000000000001 | tail -1 | awk '{print $1 " \\* " $2 " \\* " $3}') - 1
250067789
~~~

- Create the VMFS partition (Replace with your end sector)

~~~bash
~ # partedUtil setptbl /vmfs/devices/disks/naa.5000000000000001 gpt "1 2048 250067789 AA31E02A400F11DB9590000C2911D1B8 0"
~~~

