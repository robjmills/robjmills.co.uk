---
layout: post
title:  "Setting server time in Vagrant"
date:   2016-04-25 10:36:55 +0000
categories: dev vagrant
---

![vagrant](/assets/images/vagrant.png){:class="img-responsive"}

Within Liquidshop, one of the most complicated features is the calculation of correct delivery and despatch dates for an order. Add in to the mix things like timezones and bank holidays then this can be a real pain. To test this effectively it's important to be able to test for different scenarios. Testing for a specific time and date can be achieved, of course, by settting date within your scripts directly but when you're looking at state within a number of different locations then this isn't always an effective approach.

We work locally on Vagrant VM's which run an almost identical setup to our CentOS servers. Testing for the correct date "state" can most obviously be done by forcing a specific date/time/timezone on the VM directly. This can be achieved quite simply like:

`sudo date --set="17 April 2015 12:34:56"`

With Vagrant though, there is a pretty big gotcha here. I discovered, much to my frustration, that the time and date kept on auto-correcting itself. I wasted a lot of time looking into how various Linux servers keep their time in sync without realising that the issue here is actually related to Vagrant, and specifically Virtualbox.

In the case of Virtualbox it manages the system time using Guest Additions. So, if you want your time and date changes to be stored (until reboot) then you need to first disable Guest Additions. This can be achieved pretty easily with the following:

`sudo service vboxadd-service stop`

You are now free to update the server date and time without it syncing back to the correct local time.

If you want to re-enable the time syncing before reboot you can simply run:

`sudo service vboxadd-service start`

I lost a lot of time to this, I hope this helps someone else. I know i'll certainly forget this myself in the future!
