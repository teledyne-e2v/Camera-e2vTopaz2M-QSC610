# Camera-e2vTopaz2M-QSC610
Camera driver for Qualcomm QCS610

* Bin: 
The sensor driver binary file, customer can push those files in folder bin to the device's
/usr/lib/cmaera to verify.

* CodeFile: 
All source files that need to be modified during the development.

* Patch: 
Patch containing all development modifications.

* RawImage: 
Dump raw image from device.

* Function: 
1920x1080 raw10 60fps bring up on QCS610 LE2.0

## Install and Run sdkmanager
Install docker

    sudo apt  install docker.io

get the sdk manager:


    sudo apt-get install wget
    
    sudo wget https://sdkgit.thundercomm.com/api/v4/projects/4649/repository/files/turbox-sdkmanager-setup.sh/raw?ref=main -O /usr/bin/turbox-sdkmanager-setup.sh

make it executable:

    sudo chmod +x /usr/bin/turbox-sdkmanager-setup.sh


## Start SDK manager
    turbox-sdkmanager-setup.sh --tag v3.2.3

=> the docker creates a folder ~/turbox-sdkmanager-ws and everything is saved and loaded inside

Create the ssh Key (inside the docker)

    ssh-keygen -t rsa -C "romain.guiguet@teledyne.com"
    
copy tkey in the gitlab: https://sdkgit.thundercomm.com/-/profile/keys

    cat ~/.ssh/id_rsa.pub

Create a token in GitLab account for the next step

    sdkmanager --init

Here is what is needed to configure:
```
[turbox@sdkmanager-18.04-3.2.3 ~ ]$ sdkmanager --init
[INFO]	The current network is available.
[INFO]	Accessing the server...
[INFO]	Checking TOKEN ...
TOKEN INVALID.
Please refer to https://docs.thundercomm.com/turbox_doc/tools/turbox-sdk-git-user-guide to generate the token.
If you do not have a ThunderComm account, please contact service@thundercomm.com.
Please Input the token :glpat-G8FDb9noQ_jSCUPi-wPD
[INFO]	TOKEN       OK.

[INFO]	Checking SSH KEY ...
[INFO]	SSH KEY     OK.

[INFO]	Checking Git ...
[INFO]	Current git user.name = .
[INFO]	Current git user.email = .
Do you want to initialize your git account?(Yes/No): Yes
Please input the user.name: romain.guiguet@teledyne.com
Please input the user.email: romain.guiguet@teledyne.com
[INFO]	Current git user.name = romain.guiguet@teledyne.com.
[INFO]	Current git user.email = romain.guiguet@teledyne.com.
[INFO]	GIT         OK.

Select your region, as your SDK download speed is closely related to your choice of region.
    1. Chinese mainland
    2. Others
Select your region [Enter the number]:2
[INFO]	Setting region to Others ...
[INFO]	Region      OK.
```

## Download the sources (in the sdkmanager docker)

    sdkmanager --sdk_download

Here is what is needed to configure:
```
[turbox@sdkmanager-18.04-3.2.3 ~ ]$ sdkmanager --sdk_download
[INFO]	The current network is available.
[INFO]	Accessing the server...
[INFO]	Getting your information, please wait ...

Product options are as follows:
....1. TurboX_CT6490_C6490
....2. TurboX_C610_C410
Which product would you like?    [Enter the number]:
2

SDK branch options are as follows:
....1. SDK.Turbox-QCS610.LA.2.0
....2. SDK.Turbox-QCS610.LE.2.0
....3. SDK.Turbox_C610.LA1.0
....4. SDK.turbox-c610-le1.0-dev
Which SDK branch would you like? [Enter the number]:
2

Version options are as follows:
....1. turbox-c610-le2.0-dev.release.CS.r002001
....2. turbox-c610-le2.0-dev.release.Post-CS1.r002002
Which version would you like?    [Enter the number]:
2
Input a relative path(/home/turbox/workspace/<relative path>) for sdk code or input 'Enter' to select the default directory:/home/turbox/workspace/sourcecode/turbox-c610-le2.0-dev.release.Post-CS1.r002002):
Are you sure to download the source code to /home/turbox/workspace/sourcecode/turbox-c610-le2.0-dev.release.Post-CS1.r002002 ?(Yes/No): Yes
```
... AFTER COUPLE OF HOURS ...

```
[INFO]	Success: repo sync -c -d --no-tags -j3 --no-clone-bundle
[INFO]	turbox-c610-le2.0-dev.release.Post-CS1.r002002 download succeeded.
[CMD]	PATH: /home/turbox/workspace/sourcecode/turbox-c610-le2.0-dev.release.Post-CS1.r002002
```
To recreate a new installation from scratch, that is require to remove the following folders: `sourcecode` and `.repo`

## Copy the patches and apply them
Open two terminals:
- [DOCKER]: terminal where the sdk manager has been started in the previous step
- [DRIVER]: treminal where this repository has been clone, containing the divers files and patches
Please follow the next steps in the correct order to apply properly the different patches

That is also require to note the CONTAINER ID that will be used in next steps to copy the patche files to docker:
[DRIVER]:
```
sudo docker ps
```
terminal render:
```
CONTAINER ID   IMAGE                                                                COMMAND       CREATED        STATUS      PORTS     NAMES
ea61bf6aa551   public.ecr.aws/k5o4b3u5/thundercomm/turbox-sdkmanager-18.04:v3.2.3   "/bin/bash"   4 months ago   Up 3 days             turbox-sdkmanager-18.04_v3.2.3_1000
```
The container ID here is ea61bf6aa551, please replace it in all the next commands by your own ID.

### Recipe correction
This correction has to be applied manually by copying the following files:

[DRIVER]:
```
sudo docker cp recipe/patch_ptool-native_git.txt ea61bf6aa551:/home/turbox/workspace/sourcecode/turbox-c610-le2.0-dev.release.Post-CS1.r002002/apps_proc/poky/meta-qti-bsp/.
```
[DOCKER]:
```
cd /home/turbox/workspace/sourcecode/turbox-c610-le2.0-dev.release.Post-CS1.r002002/apps_proc/poky/meta-qti-bsp
```

```
patch -p1 < patch_ptool-native_git.txt
```

### Base driver :
[DRIVER]:
```
sudo docker cp Patch ea61bf6aa551:/home/turbox/workspace/sourcecode/turbox-c610-le2.0-dev.release.Post-CS1.r002002/apps_proc/src/vendor/qcom/proprietary/.
```
[DOCKER]:
```
cd /home/turbox/workspace/sourcecode/turbox-c610-le2.0-dev.release.Post-CS1.r002002/apps_proc/src/vendor/qcom/proprietary
```

```
git apply Patch/0001-Camera-Bring-up-e2vTopaz2M.patch --whitespace=nowarn
```

### [optional] Weston environment auto-start :
That is possible to start the Weston GUI environment at power up using a script.
To prepare this function, that is required to add the following option in the kernel before creating the image.

[DOCKER]:
```
vim apps_proc/poky/meta-qti-distro/conf/distro/qti-distro-fullstack-debug.conf
```
At the end of the file, add the following statement:
```
#Remove selinux distro feature
DISTRO_FEATURES_remove = "selinux"
```

## Build the image
```
cd /home/turbox/workspace/sourcecode/turbox-c610-le2.0-dev.release.Post-CS1.r002002
```

```
./turbox_build.sh -alv debug
```

... AFTER COUPLE OF HOURS ...

```


```


##Create the image

```
./turbox_build.sh -uv debug
```

```
./turbox_build.sh --zip_flat_build -l -v debug
```

## Flash the image
Get the image from the docker

[DRIVER]:
```
sudo docker cp ea61bf6aa551:/home/turbox/workspace/sourcecode/turbox-c610-le2.0-dev.release.Post-CS1.r002002/turbox/output/FlatBuild_Turbox_C610_xx.xx_LE2.0.l.debug.Post-CS1.r002002.zip .
```

Copy the ZIP file in Windows PC and extract it.

Flashing procedure:
-	Unplug all cables
-	Hold down the “FORCE_USB_BOOT” button on Kit while plugging a USB cable (without power cable!) to your windows PC. Here you should the kit appears as mode 9008 in Device Manager. 
-	When this happens, plug in the power cable and open QFIL
-	In QFIL, make sure memory type selected is **eMMC** for C610, then choose correct port, correct flat image page (in “Select Programmer” browse) and correct XML
-	Before you click download, right-click “Status” window and choose “clear log”, then click the Download button
-	If it still fails, right-click again the “Status” button and choose “Save log” and send us the log
-	Try to repeat again if it fails, after closing SW programs and unplugging all cables


## Testing

Start the board:
* connect keyboard/mouse/screen (display port)
* connect usb-C to a host PC for adb commands
* connect power supply
* power on
* push power on button

### [optional] Weston environment auto-start :
That is possible to start the Weston GUI environment at power up using a script.
It requires to have the SELINUX option disabled (see the patch section).

[DRIVER]:

Check the device is connected
```
adb devices
```
Expected render:
```
List of devices attached

1a6c8f68	device

```
Execute the following commands:
```
adb root
adb remount
adb disable-verity
adb reboot
adb wait-for-device
adb root
adb remount
adb shell "setenforce 0"
adb shell "mount -o remount,rw /"
adb push weston/weston_preview /sbin/
adb shell "chmod 755 /sbin/weston_preview"
adb push weston/weston.service /etc/systemd/system/
adb shell "chmod 644 /etc/systemd/system/weston.service"
adb shell "systemctl daemon-reload"
adb shell "systemctl enable weston"
adb shell "systemctl start weston"
```
The Weston GUI environment should start.

After a reboot, this environment automatically starts.

Here is a gstreamer pipeline that can be executed in a weston terminal to test the camera:
```
gst-launch-1.0 qtiqmmfsrc name=camsrc ! video/x-raw\(memory:GBM\), format=NV12,width=1920,height=1080, framerate=60/1 ! waylandsink fullscreen=false async=true sync=false
```

###  Save Images

[DRIVER]:
```
adb devices
```

Terminal render:
```
List of devices attached

1a6c8f68	device
```
Change the permission level
```
adb root
adb remount
adb disable-verity
adb reboot
adb wait-for-device
```
Wait for the reboot is completed

```
adb root
adb remount
adb shell
```
A terminal access running on the board is created and a gstreamer pipeline can be launched to save RAW images:
```
gst-launch-1.0 -e qtiqmmfsrc name=qmmf camera=0 ! 'video/x-bayer,format=(string)gbrg,bpp=(string)10,width=1920,height=1080,framerate=60/1' ! multifilesink enable-last-sample=false location="/data/misc/camera/MipiRaw10_%d.raw" max-files=5
```
To get the images from the platform:
```
adb pull /data/misc/camera .
```
The image format is packed in 10b words so that is required to unpacked them to 16b image before opening them in an image viewer.

###  Live preview on screen

[DRIVER]:
```
adb devices
```
Terminal render:
```
List of devices attached

1a6c8f68	device
```
Change the permission level

```
adb root
adb remount
adb disable-verity
adb reboot
adb wait-for-device
```
Wait for the reboot is completed

```
adb root && adb remount && adb shell mount -o remount,rw /
```
Start the remote shell:
```
adb shell
```
A terminal access running on the board is created.

Activate Weston environment and HDMI display:
```
export WESTON_DISABLE_ATOMIC=1
export XDG_RUNTIME_DIR=/run/user/root
weston --tty=1 --idle-time=123456 &
```
Start gstreamer pipeline
```
gst-launch-1.0 qtiqmmfsrc name=camsrc ! video/x-raw\(memory:GBM\), format=NV12,width=1920,height=1080, framerate=60/1 ! waylandsink fullscreen=true async=true sync=false
```

### Log creation for debug

In case of facing issue, that is possible to generate a log file.

[DRIVER]:
Enable log mask
```
adb push camxoverridesettings.txt /etc/camera
```
Reboot device and enable log settings
```
adb reboot
adb wait-for-device
adb root
adb remount
```
Start collect logs after the device powered up
```
adb logcat -v threadtime -b all -b crash > your_log.txt
```
Then start reproducing the problem，after reproducing the problem, use Ctrl+C to stop log scraping and provide the log file on this case


