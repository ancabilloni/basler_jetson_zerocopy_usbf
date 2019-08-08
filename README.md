# Patching Jetson TX2's linux kernel with zerocopy usbf for Basler Camera

### Purpose
- To get direct data transfer without much CPU's overhead. 
- Try to get copy of image frame faster on high speed camera.
- Read more [here](https://github.com/basler/linux-usb-zerocopy/wiki) and Basler Pylon's README at Performance Optimization section.

### Things I used on this experiment
- Jetpack version 3.2.1
- Grab [buildJetsonTX2Kernel](https://github.com/jetsonhacks/buildJetsonTX2Kernel)from Jetsonhacks to help with the build kernel process 

### Get a fresh kernel source for Jetson
- Get a Jetson kernel source by running script `buildJetsonTX2Kernel/scripts/getKernelSourcesNoGUI.sh`
- The kernel source is copied in `/usr/src/kernel/kernel-4.4` after finished downloading and uncompressed

### The manual patching process
- Get a copy of original `devio.c` in `kernel-4.4/drivers/usb/core/devio.c` and save it somewhere.
- Get a copy of Basler's `devio.c` file with zerocopy feature [here](https://github.com/basler/linux-usb-zerocopy/blob/linux-4.2.y-usb-zerocopy/drivers/usb/core/devio.c), and this is the only file they changed. Check this for [changes](https://github.com/basler/linux-usb-zerocopy/commit/4106b5b45d075d2d2d06c7ba6fa59e7999fdfd5d)
- Manually check every change from the Basler's `devio.c` version and add the changes to the orignal `devio.c`

### Save you sometimes
- You can use the `devio_zc.c` that I already applied the changes into in this repo.
- You can rename this file back to `devio.c` and replace the original `devio.c` with this file.

### Build the kernel
```
apt-get install qt5-default -y
cd /usr/src/kernel/kernel-4.4
make xconfig
```
A gui pop up, double-click on General Setup among the list on the left hand side. On the right hand side, find the row mentioned about Local version, then double-click and add the kernel's tag name for your new kernel. For example: `-usb-zerocopy`. Then **Save** & exit GUI.

Run `makeKernel.sh` script in `buildJetsonTX2Kernle/scripts` with sudo privilege and wait to finish (about 20 mins)

Copy over newly compiled Kernel to boot by running `copyImage.sh` in `buildJetsonTX2Kernle/scripts`

Reboot

### After
If system boots up normal, congrats! We did not mess this up. You can check your new kernel name with `uname -r`

To check whether zerocopy is enabled: `cat /sys/module/usbcore/parameters/usbfs_disable_zerocopy`
To disable zerocopy: `sudo sh -c 'echo 1 > /sys/module/usbcore/parameters/usbfs_disable_zerocopy'` 
To enable:  `sudo sh -c 'echo 0 > /sys/module/usbcore/parameters/usbfs_disable_zerocopy'`


