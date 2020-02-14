# Overview
The purpose of this project is to put together a Raspberry Pi for personal use at the office. The Pi is to physically separate work activities from personal activities (personal notes, email, shopping, social media, etc). Both Raspberry Pi and corporate computer will connect to a KVM switch. For my purposes the primary function will be for note taking with [Joplin](https://joplinapp.org/). Joplin has a CLI mode, however, we will require the Pi have a desktop environment for additional Joplin features and web browsing activities.

# Bill of Materials
The materials used to make this project successful are as follows:

| Item | Cost | 
| ---- | ---  |
| raspberry pi 4 w 4GB RAM | $55 |
| officail pi 4 power supply | $10 |
| [Flirc Raspi Case](https://smile.amazon.com/gp/product/B07WG4DW52/ref=ppx_yo_dt_b_asin_title_o00_s00?ie=UTF8&psc=1) | $16 |
| [64GB u3 Mirco SD Card](https://smile.amazon.com/gp/product/B06XX29S9Q/ref=ppx_yo_dt_b_asin_title_o00_s00?ie=UTF8&psc=1) | $12 |
| [Male to Female:Micro HDMI to HDMI](https://smile.amazon.com/gp/product/B07K21HSQX/ref=ppx_yo_dt_b_asin_title_o00_s00?ie=UTF8&psc=1) | $8 |
| [KVM Switch](https://smile.amazon.com/gp/product/B07QM6ND7R/ref=ppx_yo_dt_b_asin_title_o01_s00?ie=UTF8&psc=1) | $33 |
| HDMI cable (provided by your friend, the IT Guy. If they're not your friend; you're missing out.) | - |
| Total | $134 |

# Roadblocks
* There is no Jolpin AppImage for ARM64 machines
* Joplin runs on electron which only runs on 64bit systems
  * Raspbian, the official RPi distro is currently only 32-bit
* Pop!_OS is built for AMD64 machines. Pi is ARM.
  * mainly bc Pop uses gnome which does not run on ARM
* Ubuntu Server for RPi does not have a GUI (Joplin can be run on the command line only if that is the only requirement)

# Method
## Flash MicroSD Card with Ubuntu Server
1. [Download](https://ubuntu.com/download/raspberry-pi) Ubuntu Server for Raspberry Pi (64-bit)
2. [Download](https://www.balena.io/etcher/https://www.balena.io/etcher/) and install Etcher
3. Use Etcher to flash the ubuntu image onto the MicroSD card

## Setup Wifi on Ubuntu Server
Ubuntu Server for Raspberry pi was missing necessary utilities to enable WiFi and connect to a network. First we need to get these utilities installed. This is best done by connecting the Pi to the internet via ethernet. From here we will follow this helpful [guide](https://www.linuxbabe.com/ubuntu/connect-to-wi-fi-from-terminal-on-ubuntu-18-04-19-04-with-wpa-supplicant).

If the router is too far from your desk it may help to ssh into the Raspberry Pi after it is connected to the router via ethernet. To do so more easily try the following from a linux command line:
1. Create a snapshot of device IP's connected to the network before connecting the Pi via ethernet
```
nmap -sn $(ifconfig | grep -E -A1 '(wlan|wlp|enp|eth)' | grep inet | awk '{print $2}' | head -1)/24 | sort | grep report | tee netscan
```
2. Create a snapshot of device IP's after the Pi is connected to the router via ethernet
```
nmap -sn $(ifconfig | grep -E -A1 '(wlan|wlp|enp|eth)' | grep inet | awk '{print $2}' | head -1)/24 | sort | grep report | tee netscan1
```
3. Compare the snapshot files and identify new IP's on the network

```
vimdiff netscan netscan1
```
4. Connect to the RasPi via ssh to install rfkill, iwconfig, ifconfig, and wpa_supplicant as described in the above guide
* the default user on the RasPi will be `ubuntu`
```
ssh ubuntu@<raspberry Pi's IP>
```

## Install Lubuntu Desktop
Install the lubuntu desktop on the Raspberry Pi. This will allow for a graphical user interface when the Pi is connected to a monitor.

1. Install the desktop
```
sudo apt-get install lubuntu-desktop
```
2. Select the `gdm3` display manager
During the install process for the desktop a prompt to select gdm3 or sddm for the display manager will appear. We selected gdm3 as it is more easily configured and has same primary features as sddm.

Note: After the desktop is installed the Pi may show a black screen for a minute or so on boot. The Pi is simply loading the display manager. No need to fret. 

## Compile Joplin for ARM64
Note the build instructions on Joplin's github page seen [here](https://github.com/nodesource/distributions#debinstall). We will perform the build on the Pi for simplicity. Though it will not build as quickly as it would on a laptop this will be plenty speedy for a weekend project (and the perfect time to make a snack).

First, we will need to install node v10.x. It important that we do not install a newer version of node even though they are available. Version 10.x is required for the build to succeed. At the time of writing NodeSource supports ARM64.

1. Install node

To install Node.js v10.x: follow Debian and Ubuntu installation instructions [here](https://github.com/nodesource/distributions#deb). When node completes its install we see a message that invites us to install more build tools. The C compiler may help us avoid gotchas and it can't hurt. Seeing as Yarn is another requirement to build Joplin as well we may as well follow the node installer's wisdom.

2. Install more build tools and yarn
```
## You may also need development tools to build native addons:
     sudo apt-get install gcc g++ make
## To install the Yarn package manager, run:
     curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
     echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
     sudo apt-get update && sudo apt-get install yarn
```
Now we have dependencies to build Joplin in place. It's time to clone the Joplin git repo and build the application.

3. Pull down the source.
` git clone https://github.com/laurent22/joplin.git`

4. Follow instructions in the "Building the Electron application" > "Linux and macOS" section of Joplin's [BUILD readme](https://github.com/laurent22/joplin/blob/master/BUILD.md).

## Configure Joplin
Time to start up the app and sychronize it. To start the app, open the terminal and navigate to the git `joplin` directory that was created when the repo was cloned. From there run `ElectronClient/run-prod.sh` to start the application.

Once Joplin is open select `Tools > Options > Encryption` to configure end-to-end encryption.
If notes are synchronized before encryption Joplin will take some time to encrypt notes locally and in the cloud. It is much easier to do this step first if encryption is a requirement.

Next, navigate to `Tools > Options > Synchronization` to setup synchronization. Dropbox is the most straight forward way to synchronize notes between this instance of Joplin and others running on personal devices including mobile phones. Nextcloud is another way to host notes for those seeking a follow-up weekend project.

# Future Improvements
## Fully Prepared Image
We could elimiate much of the tedium of this install process if in the future we could simply flash an image that had the tools and configurations we needed in place before ever booting the pi. Primarily, WiFi tools and the Lubuntu desktop. Additionally, nordvpn and Joplin.

## Snap for Node and Yarn
We may be able to utilize snapd for the handling of the build tools install. We would need to be able to specify v10.x for the snap package, else we may have a build failure. This improvement may help keep the rest of the OS more tidy if we decide we want to build Joplin on the Raspberry Pi.

## Create GUI Shortcut to Joplin
With this method, Joplin must be started on the command line. An improvement that can be made is to create a script with an icon that can start up the application with a click.

##  Containerized(?) Joplin
* Could we compile Joplin for AMD64 and run the container on ARM64?
  * if not, since Joplin is currently delivered with an AppImage we could create a build pipeline that would create an ARM64 AppImage to contribute to the project

## Steps for installing nordvpn
Steps for setting up nordvpn with helpful aliases.

## Improve Boot Speed
Reduce the time the black screen appears while the display manager loads.

## Full Disk Encryption
For improved security and peace of mind. Without this, passers-by can remove the MicroSD card and mount it on their machine for easy viewing of potentially sensitive files.

# Thanks
Thanks to the following people that helped make this happen! This project would not exist (or would have a very steep learning curve) without you!
* [The Raspberry Pi Foundation ](https://www.raspberrypi.org/about/)
* [Ubuntu](https://ubuntu.com/) and all of it's contributors!
* laurent22 and Joplin contributors!
* https://www.linuxbabe.com/
* [Lubuntu](https://lubuntu.net/) contributors!
* N for the borrowed workspace, thanks buddy
