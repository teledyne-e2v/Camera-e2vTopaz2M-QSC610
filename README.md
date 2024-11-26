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

## Copy the patch and apply it
FROM OUTSIDE DOCKER:

```
cd /home/teledyne/turbox-sdkmanager-ws/sourcecode/turbox-c610-le2.0-dev.release.Post-CS1.r002002/apps_proc/src/vendor/qcom/proprietary
```

```
cp -r  ~/DRIVER_QUALCOMM/2024.06.28-QualcommDriversDeliverables/QCS610_release/Patch .
 ```

FROM DOCKER sdkmanager:
```
cd /home/turbox/workspace/sourcecode/turbox-c610-le2.0-dev.release.Post-CS1.r002002/apps_proc/src/vendor/qcom/proprietary
```

```
git apply Patch/0001-Camera-Bring-up-e2vTopaz2M.patch --whitespace=nowarn
```

## Build the image
```
cd /home/turbox/workspace/sourcecode/turbox-c610-le2.0-dev.release.Post-CS1.r002002
```

```
./turbox_build.sh -alv debug
```

... AFTER COUPLE OF HOURS ...

## correction patch to solve this issue
FROM OUTSIDE DOCKER:
```
cd /home/teledyne/turbox-sdkmanager-ws/sourcecode/turbox-c610-le2.0-dev.release.Post-CS1.r002002/apps_proc/poky/meta-qti-bsp
```

```
cp /home/teledyne/DRIVER_QUALCOMM/C610-install_patch/patch_ptool-native_git.txt .
```

FROM DOCKER sdkmanager:
```
cd /home/turbox/workspace/sourcecode/turbox-c610-le2.0-dev.release.Post-CS1.r002002/apps_proc/poky/meta-qti-bsp
```

```
patch -p1 < patch_ptool-native_git.txt
```
RESTART THE BUILD
```
cd /home/turbox/workspace/sourcecode/turbox-c610-le2.0-dev.release.Post-CS1.r002002
```

```
./turbox_build.sh -alv debug
```


##Create the image

```
./turbox_build.sh -uv debug
```

```
./turbox_build.sh --zip_flat_build -l -v debug
```

## Flash the image
FROM OUTSIDE DOCKER:

Identify the docker ID:
```
sudo docker ps
```
Terminal render looks like this:
```
CONTAINER ID   IMAGE                                                                COMMAND       CREATED        STATUS        PORTS     NAMES
ea61bf6aa551   public.ecr.aws/k5o4b3u5/thundercomm/turbox-sdkmanager-18.04:v3.2.3   "/bin/bash"   4 months ago   Up 17 hours             turbox-sdkmanager-18.04_v3.2.3_1000
```
The container ID here is ea61bf6aa551, please replace it in all the next commands by your own ID.

Get the image from the docker
```
sudo docker cp ea61bf6aa551:/home/turbox/workspace/sourcecode/turbox-c610-le2.0-dev.release.Post-CS1.r002002/turbox/output/FlatBuild_Turbox_C610_xx.xx_LE2.0.l.debug.Post-CS1.r002002.zip .
```

Copy the ZIP file in Windows PC and extract it.

Flashing procedure:
-	Unplug all cables
-	Hold down the “FORCE_USB_BOOT” button on Kit while plugging a USB cable (without power cable!) to your windows PC. Here you should the kit appears as mode 9008 in Device Manager. 
-	When this happens, plug in the power cable and open QFIL
-	In QFIL, make sure memory type selected is eMMC for C610, then choose correct port, correct flat image page (in “Select Programmer” browse) and correct XML
-	Before you click download, right-click “Status” window and choose “clear log”, then click the Download button
-	If it still fails, right-click again the “Status” button and choose “Save log” and send us the log
-	Try to repeat again if it fails, after closing SW programs and unplugging all cables



##  Save Images
- connect usb-C
- power on
- push power on button

```
adb devices
```
List of devices attached

1a6c8f68	device


```
adb root
```


```
adb remount
```


```
adb disable-verity
```


```
adb reboot
```

Wait for the reboot is completed

```
adb root
```


```
adb remount
```


```
adb shell
```


```
gst-launch-1.0 -e qtiqmmfsrc name=qmmf camera=0 ! 'video/x-bayer,format=(string)gbrg,bpp=(string)10,width=1920,height=1080,framerate=60/1' ! multifilesink enable-last-sample=false location="/data/misc/camera/MipiRaw10_%d.raw" max-files=5
```

##  Live preview on screen
- connect usb-C
- connect hdmi to a screen
- power on
- push power on button


```
adb devices
```

List of devices attached

1a6c8f68	device

```
adb root
```

```
adb remount
```

```
adb disable-verity
```

```
adb reboot
```

Wait for the reboot is completed

```
adb root && adb remount && adb shell mount -o remount,rw /
```
Start the remote shell:
```
adb shell
```
Activate Weston environment and HDMI display
```
export WESTON_DISABLE_ATOMIC=1
export XDG_RUNTIME_DIR=/run/user/root
weston --tty=1 --idle-time=123456 &
```
Start gstreamer pipeline
```
gst-launch-1.0 qtiqmmfsrc name=camsrc ! video/x-raw\(memory:GBM\), format=NV12,width=1920,height=1080, framerate=60/1 ! waylandsink fullscreen=true async=true sync=false
```
