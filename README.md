# high-availability-rpi-cluster
Instructions and tools for setting up a high-availability, scalable Raspberry Pi cluster to act as a web server

## Overview

This project demonstrates how to setup a high-availability, load-balanced web server using Raspberry Pi's. This documentation covers the procedure for a 4-unit stack, however the same instructions can easily be applied to any size stack from 3-unit to as many as you'd like. I chose 4-units because it demonstrates how to construct a complex infrastructure without overcomplicating the instructions.

Primary features of this project:

* Raspberry Pi's! Who doesn't love them?
* Distributed Postgres database
* Load balancing with [HAProxy](http://www.haproxy.org/)

## Caveats

This project is indented for hobbyists, and not for deployment of any critical applications, as single points of failure do exist:

* Power interruption
* Network interruption
* Load Balancer failure

It is possible to mitigate or overcome these issues, but they are complex enough that they lie outside the scope of this project. If you need a system which tolerates these failures, you probably shouldn't be running on a cobbled-together Raspberry Pi cluster. I recommend [Heroku](http://www.heroku.com) for deployment of critical applications.

## Hardware

* [Raspberry Pi 2 Model B](http://smile.amazon.com/Raspberry-Pi-Model-Project-Board/dp/B00T2U7R7I) x 4
* [Samsung 32GB MicroSD Card](http://smile.amazon.com/Samsung-Class-Adapter-MB-MP32DA-AM/dp/B00IVPU786) x 4 (I choose 32GB because it seems to be the sweet-spot for pricing, but 16GB cards will work just as well as long as your project isn't too large.
* [GeauxRobot Raspberry Pi Stackable Dogbone Case](http://smile.amazon.com/GeauxRobot-Clear-Raspberry-Enclosure-Shape/dp/B00BR1IJUO) x 1
* [GeauxRobot Raspberry Pi Stackable Addon Case](http://smile.amazon.com/GeauxRobot-Raspberry-Stackable-Case-Enclosure/dp/B00MRLM9QS) x 3
* [ORICO 4-Port USB Hub](http://smile.amazon.com/ORICO-Aluminum-12V2-5A-Adapter-3-3Ft/dp/B00CBEKK1W) x 1

If you intend to have more than 4 RPi's in your stack, [7](http://smile.amazon.com/ORICO-Aluminum-12V2-5A-Adapter-USB3-0/dp/B00C93DDKA), [10](http://smile.amazon.com/ORICO-Aluminum-Adapter-3-3Ft-USB3-0/dp/B00CBEVTL2) and [13](http://smile.amazon.com/ORICO-Port-USB3-0-Charging-Indicator/dp/B00NAMKDDY) port models are also available

* [1ft MicroUSB Cable](http://smile.amazon.com/gp/product/B019PZPYK6) x 4 (I chose to use a different color cable for the load balancer, so you may want 3 of the same color, and one of another)
* [1ft CAT6a Shielded Ethernet Cables x5](http://smile.amazon.com/Cable-Matters-Snagless-Shielded-Ethernet/dp/B00BIPSHQK) x 1 (The product I link to is a 5-pack, so you only need one)
* [TP-LINK 8-Port Gigabit Ethernet Switch](http://smile.amazon.com/gp/product/B001EVGIYG) x 1
