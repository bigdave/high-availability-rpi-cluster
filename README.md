# high-availability-rpi-cluster
Instructions and tools for setting up a high-availability, scalable Raspberry Pi cluster to act as a web server

![Image of my Raspberry Pi Cluster](http://res.cloudinary.com/ddrnjraio/image/upload/c_scale,h_412,w_289/v1458351469/rpi-stack.jpg)

## Overview

This project demonstrates how to setup a high-availability, load-balanced web server using Raspberry Pi's. I use this cluster in my home to run personal projects.

This documentation covers the procedure for a 4-unit stack, however the same instructions can easily be applied to any size stack from 3-unit to as many as you'd like. I chose 4-units because it demonstrates how to construct a complex infrastructure without overcomplicating the instructions.

Primary features of this project:

* Raspberry Pi's! Who doesn't love them?
* Raspbian Jessie (Lite)
* Distributed Postgres database
* Load balancing with [HAProxy](http://www.haproxy.org/)

## Caveats

This project is indented for hobbyists, and not for deployment of any critical applications, as single points of failure do exist:

* Power interruption
* Network interruption
* Load Balancer failure

It is possible to mitigate or overcome these issues, but they are complex enough that they lie outside the scope of this project. If you need a system which tolerates these failures, you probably shouldn't be running on a cobbled-together Raspberry Pi cluster. I recommend [Heroku](http://www.heroku.com) for deployment of critical applications.

## Hardware

* [Raspberry Pi 2 Model B](http://smile.amazon.com/Raspberry-Pi-Model-Project-Board/dp/B00T2U7R7I) x 4 (Any model of Raspberry Pi should work - they shouldn't even all need to be the same model - but I have only tested these instructions with this model)
* [Samsung 32GB MicroSD Card](http://smile.amazon.com/Samsung-Class-Adapter-MB-MP32DA-AM/dp/B00IVPU786) x 4 (I choose 32GB because it seems to be the sweet-spot for pricing, but 16GB cards will work just as well as long as your project isn't too large.
* [GeauxRobot Raspberry Pi Stackable Dogbone Case](http://smile.amazon.com/GeauxRobot-Clear-Raspberry-Enclosure-Shape/dp/B00BR1IJUO) x 1
* [GeauxRobot Raspberry Pi Stackable Addon Case](http://smile.amazon.com/GeauxRobot-Raspberry-Stackable-Case-Enclosure/dp/B00MRLM9QS) x 3
* [ORICO 4-Port USB Hub](http://smile.amazon.com/ORICO-Aluminum-12V2-5A-Adapter-3-3Ft/dp/B00CBEKK1W) x 1 (If you intend to have more than 4 RPi's in your stack, [7](http://smile.amazon.com/ORICO-Aluminum-12V2-5A-Adapter-USB3-0/dp/B00C93DDKA), [10](http://smile.amazon.com/ORICO-Aluminum-Adapter-3-3Ft-USB3-0/dp/B00CBEVTL2) and [13](http://smile.amazon.com/ORICO-Port-USB3-0-Charging-Indicator/dp/B00NAMKDDY) port models are also available)
* [1ft MicroUSB Cable](http://smile.amazon.com/gp/product/B019PZPYK6) x 4 (I chose to use a different color cable for the load balancer, so you may want 3 of the same color, and one of another)
* [1ft CAT6a Shielded Ethernet Cables x5](http://smile.amazon.com/Cable-Matters-Snagless-Shielded-Ethernet/dp/B00BIPSHQK) x 1 (The product I link to is a 5-pack, so you only need one)
* [TP-LINK 8-Port Gigabit Ethernet Switch](http://smile.amazon.com/gp/product/B001EVGIYG) x 1 (This is just my preferred switch, which allows for expansion to 8 units. Any network switch should work. Do not use a network _hub_.)

# Getting Rasbian onto your SD Card

To get started, you’ll need to download the latest version of [Raspbian](https://www.raspberrypi.org/downloads/raspbian/). That link always points to the latest version, and should prompt you to begin the download. Once the download is complete, unzip the file, and you should have a file named something similar to `2015-02-16-raspbian-wheezy.img`.

First thing we need to do is get that image onto our SD card, so using an adapter of some kind, insert your SD card into your Mac, and let’s find out which drive it is so that we can wipe it and apply the image:

    $ df -h
    Filesystem      Size   Used  Avail Capacity  iused    ifree %iused  Mounted on
    /dev/disk1     232Gi   80Gi  152Gi    35% 20972379 39933507   34%   /
    devfs          195Ki  195Ki    0Bi   100%      676        0  100%   /dev
    map -hosts       0Bi    0Bi    0Bi   100%        0        0  100%   /net
    map auto_home    0Bi    0Bi    0Bi   100%        0        0  100%   /home
    map -fstab       0Bi    0Bi    0Bi   100%        0        0  100%   /Network/Servers
    /dev/disk2s1    56Mi  9.7Mi   46Mi    18%      512        0  100%   /Volumes/boot

The giveaway here is Location being `/Volumes/...`. If you have multiple entries showing that, you probably have an external hard drive, or usb drive connected. You can use Size to further narrow down which volume is your SD card, however as you can see in my case, with disk2s1, it’s not always going to be as clear as you might like (I’m not using a 56Mb card…). The important part of this is the 2.

Now that we know which volume is our SD card, let’s copy the image to it. First we have to unmount it so that we can actually write. Replace the 2 in disk2s1 with the number of your device, since it might be different:

    $ sudo diskutil unmount /dev/disk2s1

Now use dd to copy the image onto the drive. if and of indicate the input file, and output file. Remember to replace the 2 in of with the number you discovered above:

    $ sudo dd bs=1m if=~/Downloads/2015-02-16-raspbian-wheezy.img of=/dev/rdisk2
    rdisk2
    3125+0 records in
    3125+0 records out
    3276800000 bytes transferred in 220.861308 secs (14836460 bytes/sec)

This command may take several minutes to run, depending on the speed of your card, so go grab a drink.

Once that completes, we can eject the card, and it’s ready to insert into the Pi:

    $ sudo diskutil eject /dev/rdisk2
    Disk /dev/rdisk2 ejected

> **Cloning**: So this might be a good time to clone your SD card, rather than follow the same procedure 3 more times. If you can figure out how to do that, go ahead. You might also wait until you've completed more steps below so that you don't have to repeat those. This guide does not cover cloning SD cards. I expect you to have discipline, and type every command exactly - as many times as you need to.

# Connecting to the Pi

Insert your SD card into the Pi, connect an ethernet cable to it, and then connect power. Wait about a minute while your Pi boots and grabs an IP address.

There are a few ways to figure out what the IP address of your PI is, but I find the easiest is to go into your router, look at the DHCP leases, and find the newest. Assuming you are on your home network, it should be something like 192.168.0.32. Armed with this knowledge, you should now be able to ssh into the device:

    $ ssh pi@192.168.0.32
If you have the right address, you’ll receive a warning about not knowing if the host is authentic, and asked if you want to proceed. You want to, so type yes. If you have the wrong address, you’ll see a message like this:

    ssh: connect to host 192.168.0.32 port 22: Connection refused

You will be prompted for a password, which will be raspberry. Type that (you won’t see the characters appear on the screen, for security reasons).

Since this is your first time logging in, you should see a message like this:

    NOTICE: the software on this Raspberry Pi has not been fully configured. Please run 'sudo raspi-config'

Go ahead and do that now, and you’ll be presented with a menu and several options. For now, all we want to do is select the first option, ‘Expand Filesystem’, so that the entire SD card is available for use. Select that, and once it completes and you are returned to the menu, select ‘Finish’ using the arrow keys.

Let’s make sure we have the latest versions of all of the pre-installed software before we proceed:

    $ sudo apt-get update
    $ sudo apt-get upgrade
And if you want to install any other software as a convenience, now would be an acceptable time to do that:

    $ sudo apt-get install tree

# Configuring Servers

In order to make things easier to manage, we're going to change the hostnames of each Raspberry Pi. I'm using the following, but your names can be whatever you like:

    rpi-master
    rpi-slave1
    rpi-slave2
    rpi-slave3

Do this on each Pi by editing `/etc/hosts` and replacing `raspberrypi` on the final line with whatever hostname you choose.
Next, edit `/etc/hostname` and replace `raspberrypi` with the same hostname again.
Run the following commands to update the hostname and reboot:

    $ sudo /etc/init.d/hostname.sh
    $ sudo reboot

## Master

This is the server which will do our load balancing, distributing requests to the webservers. We will do this using HAProxy. There are a lot of tools available to do this, but I enjoy HAProxy because it's extremely easy to setup, and can be configured to work for a wide variety of applications.

    $ sudo apt-get install haproxy
    $ sudo service haproxy status
    
You should now be up and running with HAProxy. Of course now we have to configure it and tell it what to do. First, let's make a backup of the config for reference later:

    $ cp /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg.original

Now let's enable logging. Edit `/etc/rsyslog.conf` and look for the following section, which will be commented out. Uncomment it and it should look pretty much like the following:

    # provides UDP syslog reception
    $ModLoad imudp
    $UDPServerRun 514

Restart the syslog service and logging should be working:

    $ sudo service syslog restart

## Slaves

    $ sudo apt-get install git ruby ruby-dev

Now install Rails. This may take some time (On a Raspberry Pi 2 Model B it took me an hour and a half)

    $ sudo gem install rails
    
