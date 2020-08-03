# MiniMagnus Build Guide
You can build your own MiniMagnus using this guide. 

## Parts List
(Note: This list assumes you will use 9x Pis. You can start with as few as 2x Pis.)
* 9x Raspberry Pis
We recommend going with 3B or 3B+ - note that the Pi 4B and up are NOT supported. This is because the graphics stack in the Pi 4B is different and not yet supported by the demos.
* 9x microSD cards (at least 16GB in size), preferably from SanDisk or Samsung.
* 9x USB to microUSB cables (preferably glowing)
During testing, you may want to use cables with an in-line power toggle switch.
* 2x GeeekPi Raspberry Pi Cluster Case Raspberry Pi Case with Cooling Fan
This cluster case houses 4x Pis, for a total of 8, and includes heatsinks and fans to keep them cool.
* Flirc Raspberry Pi Case Gen2 (New Model)
This case acts as a heatsink, and is used to keep the Prime node cool. You may not be able to find this exact case - either a decent metal case or a case with active cooling will do.
* Standard HDMI cable
* USB keyboard and mouse for initial setup + debugging
* Sabrent 10-Port 60W USB 3.0 Hub (or equivalent) to power the nodes.
* TP-Link TL-SG116 16-port Gigabit Ethernet Switch (or equivalent) to provide networking to the nodes.
* Ubiquiti EdgeRouter X (or equivalent) to provide DHCP and Internet services.
* 11x ethernet cables (one for each Pi, one to connect the switch and the router, and one to connect the router to a network)
* Mayflash MAGIC-NS adapter for connecting an appropriate Bluetooth gamepad.
* Sony PlayStation 4 DualShock 4 Controller (recommended)

## Initial Physical Build
* Label each of the nodes with a number (the side of the Ethernet port is a good place to put the label).
* Take the first node (labelled "1") and install it into the Flirc case. This will be the Prime node.
* Install 4x nodes into each of the cluster cases. This will take a while - expect about 2-3 hours of building.
* Connect the Ethernet cables from the nodes to the switch. Port 1 should go to the Prime node (node 1), port 2 to node 2, etc.
* Plug in the EdgeRouter X, and configure it appropriately. (This is beyond the scope of ths document at this stage, but feel free to ask us for help if you run into trouble).
* Connect the EdgeRouter X to port 16 on the switch.
* (Optionally) connect your management laptop to port 15 on the switch.

## Getting Started
* Download and extract the latest Raspbian 32-bit image ("Raspberry Pi OS (32-bit) with desktop")
* Prepare the microSD cards by flashing that image to them, using either balenaEtcher or Raspberry Pi Imager
* Insert the microSD card into each Pi

## First boot and configuration
We'll set up the nodes one-by-one to keep things simple. You'll need to follow these steps for each of your Pis.

* Attach a keyboard and mouse, and connect the node to a suitable TV or monitor using an HDMI cable
* Turn on the node and wait for it to boot
* Run through the Raspberry Pi setup wizard, skipping the WiFi and update steps. Click Restart when prompted.
* Once the Pi reboots, click the Pi icon in the top left, then go to Preferences -> Raspberry Pi Configuration.
* Change the Hostname from "raspberrypi" to "piX" where X is the node number for the node. (for example, the prime node will be called pi1)
* Click the Interfaces tab and enable SSH
* Click OK, then reboot when prompted.

Now it's time to set up the networking. 

* Open up the EdgeRouter X control panel (https://10.99.86.1) and log into it.
* Go to the Services tab
* Next to LAN, click the Actions dropdown, then click View Leases.
* You should see your Pi in the lease list. Click the "Map Static IP" button.
* Give the Pi an appropriate IP - we suggest 10.99.86.10X, where (again) X is the node number. Ensure its name is correct, then hit Save.
* Reboot the Pi
* In a terminal, check that the Pi now has the correct IP ("ip a" or similar)

The node is now ready to be configured.

## Initial Software build
Now it's time to install, configure and compile everything needed for MiniMagnus to work.

### Ansible
There's an Ansible playbook that does most of the initial software setup for you.

Go to [the MiniMagnus repository](https://github.com/PawseySC/minimagnus) and follow the instructions, then return to this document when you're done there.

### Setting up and pairing the controller
* (Optional) Plug the MAGIC-NS into a Windows PC and use the update software to ensure it's running the latest firmware. You can probably skip this step unless you are having issues.
* Plug the MAGIC-NS into the Prime node. It will light up.
* Press and hold the little button on the side for 3-5 seconds. The LED will change and cycle through the different input modes. Keep changing modes until the LED turns GREEN.
* Take your PlayStation 4 controller, and hold the SHARE button and the PlayStation logo button at the same time. The light bar on the top of the controller should start flashing rapidly.
* Press the button on the side of the MAGIC-NS. It should briefly flash rapidly then turn solid green once the pairing is done.
* Reboot the Prime node.
* Once the Prime node is back up and at the desktop, turn the controller on and move the left thumbstick. It should now control the mouse.

### Getting the SPH demo working
The Ansible playbook will have placed most of the resources needed for the SPH demo in the appropriate places.

You will need to build the SPH demo and push it to the compute nodes.

* On your prime node, change to the /home/pi/SPH directory
`cd /home/pi/SPH`

* Edit the makefile and ensure all of your compute nodes are listed in the "copy" section
`nano makefile`

* Run `make` to compile the demo
`make`

* Now run `make run` to push it out and try it.
`make run`

Check that everything seems to work. To exit the demo, hit the "SHARE" button,
then move the "leaf" cursor over the Terminal image and press the "X" button.

Now that the SPH demo has been built and pushed, it can be run by the Kiosk script later.

### Getting the PiBrot (Mandelbrot) demo working
As with the SPH demo, the Ansible run will have already placed most resources needed for PiBrot.

You will need to build the PiBrot demo and push it to the compute nodes.

* On your prime node, change to the /home/pi/PiBrot directory
`cd /home/pi/PiBrot`

* Edit the makefile and ensure all of your compute nodes are listed in the "PIS=" variable
`nano Makefile`

* Run `make` to compile the demo
`make`

* Run `make run` to push it out and try it.
`make run`

To start the demo, move the left thumbstick on your controller. A leaf "cursor" will appear. Hover it over the Start button and press X.

Let it run for a couple of seconds, then exit.

To exit the demo, hit the "SHARE" button,
then move the cursor over the Terminal image and press the "X" button.

Now that the PiBrot demo has been built and pushed, it can be run by the Kiosk script later.

### Putting it all together - the Kiosk script
We have included a "kiosk" script which currently lives at /home/pi/startsph. It's a simple bash script that uses exit codes to switch between the two included TinyTitan demos.

Also included is a custom rc.local file which sets up keymapping from your gamepad's buttons and controls to the appropriate keyboard and mouse inputs. 

If you press the "PlayStation" button (the button in the middle of the controller), the Kiosk script is automatically started.

By default, the Kiosk script will boot into the SPH demo, but you can use the menu triggered by
the "SHARE" button to switch between demos. Just move the cursor as before over what you want, then hit X.

Selecting the "Terminal" icon will simply exit the Kiosk script and leave you on the Raspbian desktop.

## Testing it out
Perform one last reboot, then turn the controller on and hit the PlayStation button. The demo should roar to life. Hit the SHARE button and select Mandelbrot, and the SPH demo should close and the PiBrot demo should open.

## Troubleshooting
* If you can't connect to a node with its hostname (eg, ssh pi@pi1.local), try installing the "netatalk" package. It'll be installed by the first Ansible run.
* If you're having trouble trying to control the simulation, you may need to edit which event listeners are in use. These can be found in SPH/src/egl_utils.c (lines 178 and 179) and PiBrot/egl_utils.c (lines 173 and 174). You'll need to recompile the software if you do this.
* If you want to test out the demos using mouse and keyboard (no controller), edit the files above so that they use event0 and event1.
* If a simulation freezes or you need to kill the simulation for any other reason, SSH into the Prime node and run ~/killdemos.sh
