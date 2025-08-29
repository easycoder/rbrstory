# The system controller #

## Hardware ##

The system controller is a small Linux computer. For the time being I'll describe just one model, but the same applies to pretty well anything that runs Linux and has a display. The only cable that connects to the device is for power; everything else is done with wifi or Bluetooth.

The model currently used is a Chinese-made industrial computer called the IXHUB. The CPU is from the ARM family and it normally arrives programmed with an ancient version of Android.

Whatever computer is used, the software assumes it will be oriented in potrait mode, not landscape. The IXHUB comes without a case, so something suitable needs to be constructed. It may be that a design will become available for use by a 3D printer.

## Software ##

The first thing to do is to flash the computer with Linux. The manufacturer of the IXHUB provides instructions on how to do this; the instructions and the relevant files are all in our repository. There are a number of other steps to take; here is a complete set of instructions.

The IXHUB is an industrial computer with a touchscreen, that comes with a rather elderly version of Android installed. For many users the first job is to convert this to Linux, which is well suited to this machine. The manufacturer supplies instructions at [https://docs.google.com/document/d/16iSiGHH2VvuXWdOkp5VizXUO-V9MsP6H/view](https://docs.google.com/document/d/16iSiGHH2VvuXWdOkp5VizXUO-V9MsP6H/view) and requires a Windows computer. Study the instructions carefully and follow them exactly; they may seem incomplete but in fact everything you need is there.

Once this job is done, the computer will boot up in Linux. However, the problem for an English or American user is that the system is all in Chinese. So here is what to do.

First connect a keyboard and a mouse to the computer. Use one or both of the ports marked USB.
Click the internet icon on the right of the task bar at the top of the screen. Select your router from the list and supply the password. Now go to the bottom of the screen and look for the console application, which has an icon showing a black window. Click to open it, then type
```
ip a
```
which will give you all the network settings. Look for the IP address of the machine. You can log into this address from another computer on the network, using SSH and giving the user name 'linaro' and the password also 'linaro'. Or alternatively you can continue on the IXHUB itself.

Now update the software on the computer, using these commands:
```
sudo apt update
sudo apt upgrade
```
To change the system language you need to edit a couple of files, and for this you will use the nano editor, which isn't installed. So fetch it:
```
sudo apt install nano
```
Let's change the system language from Chinese to UK or US English. Open the following file for editing:
```
sudo nano /etc/default/locale
```
As supplied, this specifies the Chinese language, so change it by replacing all the content with the following. (American customers should replace GB with US; for other languages please consult the Linux documentation regarding locales.)
```
LANG=en_GB.UTF-8
LANGUAGE=en_GB:en
```
The next time you reboot the computer it will come up in English.

### Setting portrait mode ###

Some applications require the computer to be in portrait mode, but the default is landscape. If this applies to you, the next job is to rotate the screen. We do this by adding an `xrandr` command to the startup applications:

 - Open Startup Applications:
 - Go to the XFCE menu and navigate to Settings > Session and Startup.
 - Click on the Application Autostart tab.
 - Add a New Startup Program:
 - Click on the "Add" button.
 - In the "Name" field, enter a descriptive name like "Rotate Screen".
 - In the "Command" field, enter the xrandr command for the desired rotation:
 - `xrandr --output DSI-1 --rotate left`
 - Click "OK" to save the new startup program. The next time you restart, the screen will be rotated 90 degrees to the left.

## Setting up the RBR software ##

For this, we strongly recommend you connect to the controller using SSH from your own computer. Log in as linaro/linaro, as previously instructed.

After flashing with Linux, the IXHUB has a rather curious partitioning scheme. Here I'm assuming you have a model with 16GB of SSD. The root partition, which takes up well under half of this, also includes the `/home` directory, but very little is left over for user files. There is however a `userdata` partition with about 8GB of space, easily more than enough for our needs here. So we'll fix the system to use this instead of the default.

First, log in as the superuser:
```
sudo sh
```
No password is needed. Now we'll move all our user data to the `/userdata` partition:
```
cd /home
cp -r linaro/* /userdata
cp -r linaro/.* /userdata
rm -rf linaro
```
Then we'll make `/userdata` our home partition and exit from superuser mode:
```
ln -s /userdata linaro
exit
```
You can test everything is OK by rebooting the IXHUB. Now we can set up RBR. All the relevant files go into an `rbr` folder, leaving the top level home directory free, which makes things a lot simpler when updates are performed. We need to download our RBR setup script from the website, then run it to do the system installation:
```
mkdir rbr
cd rbr
wget https://rbrheating.com/ui/setup
sh setup
```
When this runs it first asks for a new password, to replace 'linaro'. Then it performs a complete system update (it will ask for the new password again) and fetches/installs everything it will need. This can all take some time so be patient. At the end, the system restarts itself and you can then log in again as 'linaro' but with your new password.

At this point the RBR system will be running its main script. The script is called `rbr.ecs` and it runs every 60 seconds as a `cron` task.

## Changing the router ##

If you change your router you will need to reconfigure the system, and this can't be done on the controller GUI as the machine most likely doesn't have a keyboard. Instructions on how to proceed can be found ~stid:software/router:here~

~stid:home/pagelist:List of Pages~
