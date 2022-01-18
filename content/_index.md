---
title: "Index"
date: 2020-04-23T12:46:11+02:00
draft: false
description: "Make your own Little Printer!"
ogimage: "littleprinter_table.jpg"
---
<!-- TOC -->

- [What is this?](#what-is-this)
- [So you want to have a Little Printer](#so-you-want-to-have-a-little-printer)
- [What you'll need](#what-youll-need)
- [How does this all work?](#how-does-this-all-work)
- [How to get started](#how-to-get-started)
- [Let's make a (fake) printer](#lets-make-a-fake-printer)
- [Wiring it all up](#wiring-it-all-up)
- [Setting up python-paperang](#setting-up-python-paperang)
- [Setting up sirius-client](#setting-up-sirius-client)
- [Credits](#credits)
- [FAQ](#faq)
- [What we need help with](#what-we-need-help-with)
- [History / timeline](#history--timeline)
- [Links](#links)
  - [Code](#code)
  - [Discussion/community](#discussioncommunity)
  - [Code of Conduct](#code-of-conduct)
- [Other](#other)
  - [Misc notes ignore for now](#misc-notes-ignore-for-now)
  - [Addendum: how the whole fake printer thing works (by Josh)](#addendum-how-the-whole-fake-printer-thing-works-by-josh)

<!-- /TOC -->

![little printer](littleprinter_table.jpg)

# What is this?

The [Little Printer](https://vimeo.com/32796535) was a cute internet-connected thermal printer released in 2012 by BERG. The company shut down in late 2014, and so did the servers at the end of 2015. They open-sourced a [minimal reimplementation of the server](https://github.com/nordprojects/sirius), which was later [extended by Nordprojects](https://nordprojects.co/projects/littleprinters/), along with releasing a new [app](https://apps.apple.com/us/app/little-printers/id1393105914), in 2019. With that, people with Little Printers could use them again after [hacking](https://github.com/genmon/sirius/issues/26#issuecomment-378230535) their [BERG Cloud Bridge](http://web.archive.org/web/20130501144818/http://bergcloud.com/help/#the-bridge). You can't really buy a Little Printer anymore; this page is all about making your own.

# So you want to have a Little Printer

Good news! You came to the right place. This page (hopefully) contains all the information and tools you need for it. **But first: [join our Discord!](https://discord.gg/cKagPkjcCS)** It's where all the development happens, and where you can get help if you get stuck with something.

# What you'll need

* A thermal printer, like the Paperang P1. There are several places where you can buy one: Amazon ([US](https://www.amazon.com/PAPERANG-Portable-Language-Wireless-Bluetooth/dp/B07TCJ2TC6/), [UK](https://www.amazon.co.uk/PAPERANG-Wireless-Mobile-Instant-Printer/dp/B078HZK2NV), [DE](https://www.amazon.de/Paperang-Mini-Drucker-Android-Ger%C3%A4te-Druckpapier-gew%C3%B6hnlich/dp/B082PWV57H/)), [Thepaperang](https://thepaperang.com/products/paperang-p1), [Paperangprint](https://paperangprint.com/collections/paperang-printers/products/paperang-p1), [Aliexpress](https://www.aliexpress.com/w/wholesale-paperang-p1.html) and so on. Don't forget to buy a few rolls of paper for it; you can find them in many colors, and sticker paper is especially fun!
* As an alternative, you can use any thermal printer that uses the [ESC/POS](https://en.wikipedia.org/wiki/ESC/P#Variants) protocol. This guide was written with the Paperang in mind, but you can skip the irrelevant bits and use the `escpos` driver in [`sirius-client`](https://github.com/ktamas/sirius-client).
* Ideally, a Raspberry Pi with Bluetooth, but we've tested this with a Mac as well, and with some tweaks, it should work on any computer with Bluetooth running *nix.
* Node.js 10, Yarn, Python 3.7, Docker.
* An iPhone for the [app](https://itunes.apple.com/us/app/little-printers/id1393105914?ls=1&mt=8), but you can use the [web client](https://littleprinters.ink/) as well (with reduced functionality).

# How does this all work?

I'm going to use this useful graph from [Nordprojects](https://nordprojects.co/projects/littleprinters/):

![architecture](how-it-works.png)
<small>© Nordprojects, 2019</small>

The original Little Printer architecture required two devices: the printer itself, and the bridge. The printer connected to the bridge via [Zigbee](https://en.wikipedia.org/wiki/Zigbee), and the bridge connected to the BERG Cloud (which was replaced by the [`sirius`](https://github.com/nordprojects/sirus) project — see below). The BERG Cloud Bridge acted as a platform for an envisioned ecosystem of IoT devices, that never really materialized. The client ([`sirius-client`](https://github.com/ktamas/sirius-client) — we'll get to that soon) acts as the bridge and has various "drivers" for printers.

![new architecture](how-it-works-2.png)
<small>© Nordprojects, 2019</small>

# How to get started

(If you have an ESC/POS-compatible thermal printer or you don't have a printer yet: create a fake printer below then jump to the FAQ.)

We'll assume you have a Paperang P1. Great! You can already use it via its app on your smartphone, but that's not what we'll do, at least, not for the most of this. One thing you should do before we start: open up the app, click the three dots on the top right corner, select your Paperang and under "Automatic Shutdown Settings", set it to "Non-automatic close". This prevents it from going to sleep.

Connect your Paperang to a power source with a USB cable to prevent drain. You can communicate with the printer via USB, but for now, we'll just use Bluetooth.

Pair your Paperang with your computer over Bluetooth and make sure you have Node 10, Python 3.7 and Docker installed.

# Let's make a (fake) printer

This project is predicated on us pretending to be a Little Printer, so let's do that.

First, visit [this website](https://little-printer-claim-code.netlify.app/) that generates the required `*.printer` file for you. Download it and guard it with your life; this is your printer now.
<!--
 - Clone the [`nordprojects/sirius`](https://github.com/nordprojects/sirius) repository from github
 - `cp .env.example .env`
 - edit `.env` and add `DATABASE_URL=postgresql://postgres:plop@sirius-database/sirius`
 - `docker-compose -f docker-compose.yml -f docker-compose.db.yml -f docker-compose.development.yml up --build -d`
 - `docker-compose exec sirius bash`
 - `./manage.py fake printer`

If you run into any errors, take a look at the [troubleshooting steps](sirius-server-troubleshoot) for the `sirius` server. 
-->

The generated printer file will look something like this:

```
     address: db708b77ae2ee5b5
      secret: 1011795836314
  claim code: fojy-q4xv-7pe2-xt00
```

Next, we're going to claim the printer on [Nordprojects' `sirius` instance](https://littleprinter.nordprojects.co/). Sign in there with Twitter, and click "Claim a printer". Enter the claim code you have and give it a name, then hit "Claim Printer". **Note: the claimed printer will not show up on the UI until you connect it to the server (see the next section).**


# Wiring it all up

Next, we'll put it all together, connecting our Paperang to the network. For that, we use two projects: [sirius-client](https://github.com/ktamas/sirius-client) and [python-paperang](https://github.com/ktamas/python-paperang). They should really be merged into one. One day.

Anyways, clone both of them from Github, and we'll start testing things first with the latter project. We're going to assume you have at least a passing understanding of both Python and Typescript.

# Setting up python-paperang
If you're on Linux, and we'll assume Debian (or Raspbian), you'll need the following packages installed: `build-essential libbluetooth-dev libhidapi-dev libatlas-base-dev python3-llvmlite python3-numba python3-skimage python-llvmlite llvm-dev cython3 python3-skimage`.

Install the requirements: `pip3 install -r requirements.txt`.

Create a `config.py` based on `config.example.py`. If you don't have the MAC address of your paperang, that's fine; just use an empty string and the code will find it. 

Run `printer.py` to print a self-test. If you did everything well, you should have a bunch of infos printed on your Paperang! You can also edit this file to print any arbitrary image, processed with the famous [Atkinson dithering algorithm](https://en.wikipedia.org/wiki/Dither#Algorithms).

# Setting up sirius-client

Next, let's get `sirius-client` working.

You'll need npm and ts-node installed; to do that on Debian/Raspbian, run the following:
```sudo apt-get install npm
sudo npm install -g ts-node
```

You'll also need to install yarn if you don't already have it.
On Debian, you should follow [yarn's specific install instructions](https://classic.yarnpkg.com/en/docs/install#debian-stable). Come back here when you're done.

Now run `yarn install`.

Time to run sirius-client! add the path to the printer file you generated as a command-line argument:
```
yarn ts-node bin/client.ts run --uri wss://littleprinter.nordprojects.co/api/v1/connection -p ~/my-printer.printer -d filesystem
```

Now run python-paperang with `python3 littleprinter.py`, and you're ready to print something!

Go back to [Nordprojects' `sirius` instance](https://littleprinter.nordprojects.co/), and your fake little printer should show up. If not, claim it again, and then it will work. Save the [`device.li`](https://device.li) address, you'll need it.

Get the [app](https://itunes.apple.com/us/app/little-printers/id1393105914?ls=1&mt=8) (or use the [web client](https://littleprinters.netlify.app/)), add a printer via its `device.li` address, and print something! Or use the API. Go wild.

# Credits

First of all, follow us on twitter ([@tinyprinterclub](https://twitter.com/tinyprinterclub))!

Major contributors to this project include [Tamas Kadar (KTamas)](https://ktamas.com/) ([email](mailto:ktamas@ktamas.com), [twitter](https://twitter.com/ktamas)), [Joshua May (notjosh)](https://notjosh.com/) ([twitter](https://twitter.com/notjosh)) and Monica Farrell ([twitter](https://twitter.com/monica_farrell)). Watch the [!!Con 2020](http://bangbangcon.com/) talk ["Little Printing for Everyone!!1"](http://bangbangcon.com/speakers.html#tamas-kadar) by KTamas once it's published on YouTube. The "how does it work" image was created by [Nordprojects](https://nordprojects.co).

# FAQ

**Q: Why?**  
A: Because it's fun.

**Q: Do I need a Paperang P1, specifically? Can't I just use something else?**  
A: Of course you can! `sirius-client` already has a generic `escpos` driver, which lets you connect a wide variety of thermal printers that use the ESC/POS protocol. It also has a simple console driver, that simply shows the image your printer would print. There is nothing stopping you from making your own driver to your favorite printer of choice.

**Q: What about the other Paperang models (P2, P2S)?**  
A: We **think** they *should* work, but we have not tested them yet. Assuming they do, there is another caveat: they print at a higher resolution. The P1 has the exact same resolution as the original Little Printer, and the server sends pixel-perfect bitmaps — for now — to the client, hence this is not a problem with that. There are plans to modify the server so it supports more resolutions; you're very welcome to contribute.

**Q: Where can I get a real Little Printer?**  
A: I mean, eBay, if you're lucky. I've been looking for a while now, and it seems like noone wants to sell theirs.

**Q: What if I have a real Little Printer and I want it to connect to the new network?**
A: [This document](https://docs.google.com/document/d/1JT1f2ClVdAnjrnby92V9ONBnN05EFQGYpLG5ijl5KRI/edit) should have all the information you need. You'll have to flash your BERG Cloud Bridge. Unfortunately, it has a hardware bug: there is about a 1-in-100 chance you will brick it when you flash it. Not much can be done about that.

# What we need help with

* Reviewing PRs for `sirius`, merging headless chrome
* Porting `python-paperang` to Typescript and/or improving communication between that and `sirius-client`
* Building lots of cool services
* Checking out Paperang clones, see if they work with the library or not
* Building an Android version of the [app](https://itunes.apple.com/us/app/little-printers/id1393105914?ls=1&mt=8) ([source code](https://github.com/nordprojects/littleprinters-ios-app))
* ...and so much more

# History / timeline

* (November, 2011) [Hello Little Printer, available 2012](https://vimeo.com/32796535) (Product announcement link)
* (March, 2012) [Little Printer homepage](http://web.archive.org/web/20120301003906/http://bergcloud.com/littleprinter/)
* (May, 2014) [BERG Cloud shop](http://web.archive.org/web/20140529015642/http://uk-shop.bergcloud.com/) 
* (July, 2014) [Little Printer homepage](http://web.archive.org/web/20140722111616/http://littleprinter.com/)
* (September, 2014) [Week 483](http://web.archive.org/web/20151019041519/http://blog.bergcloud.com/2014/09/09/week-483/) (BERG is shutting down)
* (September, 2014) [The future of Little Printer](http://web.archive.org/web/20150207025053/http://littleprinterblog.tumblr.com/post/97047976103/the-future-of-little-printer)
* (March, 2015) [Trying something](http://web.archive.org/web/20151025093243/http://littleprinterblog.tumblr.com/post/112431535978/trying-something) (Announcing `sirius`, the new backend for the BERG Cloud Bridge)
* (March, 2015) [genmon/sirius](https://github.com/genmon/sirius)
* (March, 2015) [A little longer](http://web.archive.org/web/20151025164214/http://littleprinterblog.tumblr.com/post/115018012993/a-little-longer) (Keeping the servers running for a few more months)
* (December, 2015) [End of 2015](http://web.archive.org/web/20151025164214/http://littleprinterblog.tumblr.com/post/115018012993/a-little-longer) (People working on Sirius, Bridge can be updated to use it)
* (December, 2015) [Updating the Bridge](https://github.com/genmon/sirius/wiki/Updating-the-Bridge)
* (December, 2015-July, 2019) [Directing the BERG cloud bridge](https://github.com/genmon/sirius/issues/8) (long GH issue, important historical document about the "lost years")
* (May, 2019) [Little Printer returns as an open-source messaging device](https://www.theverge.com/2019/5/19/18628287/little-printer-berg-nord-projects-app-open-source-messaging)
* (May, 2019) [Little Printers, a friendly new messaging app and cloud platform.](https://nordprojects.co/projects/littleprinters/) (Nordprojects' announcement of the new app, working server, api, device.li etc.)

# Links

## Code

* [nordprojects/sirius](https://github.com/nordprojects/sirius) (the server's code)
* [nordprojects/littleprinters-ios-app](https://github.com/nordprojects/littleprinters-ios-app) (the iOS app's code)
* [andrewn/littleprinters-web](https://github.com/andrewn/littleprinters-web) (port of iOS app to the web)
* [notjosh/sirius-client](https://github.com/notjosh/sirius-client) (notjosh's universal client for `sirius` — WIP)
* [ktamas/sirius-client](https://github.com/ktamas/sirius-client) (KTamas' fork of the client — older, but works)
* [tinyprinter/python-paperang](https://github.com/tinyprinter/python-paperang) (the python library that connects to the Paperang P1, with the protocol reverse-engineered)
* [sirius-image-decoder](https://ls6.github.io/sirius-image-decoder/) (detailed exploration of decoding images coming from sirius-server, with working code)
* [tinyprinter/tinyprinter.club](https://github.com/tinyprinter/tinyprinter.club) This website! Feel free to add more information to it and submit a PR! It's built with [hugo](http://gohugo.io/).

## Discussion/community

* [Join our Discord!](https://discord.gg/cKagPkjcCS)
* Check out [this GitHub issue](https://github.com/genmon/sirius/issues/26) for past discussions.

## Code of Conduct

* You can find our CoC [here](code-of-conduct).

# Other

## Misc notes ignore for now

BERG Cloud Bridge sits by your broadband router and wirelessly connects Little Printer to the web, which makes it easy for you to place Little Printer where you can see it.

---

## Addendum: how the whole fake printer thing works (by Josh)

Josh: It takes the printer ID (mac address of printer iirc, that you can generate) and makes a random 16 digit code ("claim code"). From there, it posts that to the server as part of the normal handshake. Then the device just...waits. eventually the user logs onto the server, types in claim code, then you own that printer ID, and the server will start firing payloads at it. It's all very....simple. no like, fancy key exchange, or identification, or anything.

KTamas: right, so if i want to make a new printer, what do i do?...

Josh: so, the most reliable way to make a printer is to clone `https://github.com/nordprojects/sirius` then run `./manage.py fake printer` -> generates `*.printer` file. Looks like:

```
  address: abcdef0123456789
       DB id: 8
      secret: 4a8489a9b8
  claim code: 123a-456b-789c-012d
```

Sso that is "a printer", essentially. but what do we do with that? ... well! When we connect to the server (via websocket), we don't need to send any special "it's my first time" payload, we just send a regular payload every time, that looks like:

```
{
    type: 'BridgeEvent',
    json_payload: {
      ncp_version: '0x46C5',
      uptime: '45.71 23.94',
      firmware_version: 'v2.3.1-f3c7946',
      network_info: {
        extended_pan_id: '0xredacted',
        node_type: 'EMBER_COORDINATOR',
        radio_power_mode: 'EMBER_TX_POWER_MODE_BOOST',
        security_level: 5,
        network_status: 'EMBER_JOINED_NETWORK',
        channel: 11,
        security_profile: 'Custom',
        power: 8,
        node_eui64: '0xredacted',
        pan_id: '0xDF3A',
        node_id: '0x0000',
      },
      name: 'power_on',
      local_ip_address: '192.168.1.98',
      uboot_environment:
        'long text',
      mac_address: 'redacted',
      model: 'A',
    },
    bridge_address: 'redacted',
    timestamp: 1426256447.70695,
  }
```

(Much of that can actually be ignored I think - sirius isn't using it - but that's what it looks like.) So that gets the bridge online, not the printer. To get the printer online, it sends a packet afterward that looks like:

```
{
    timestamp: 1419107228.91187,
    type: 'BridgeEvent',
    json_payload: {
      name: 'encryption_key_required',
      device_address: 'redacted',
    },
    bridge_address: 'redacted',
  }
```
So at that point, the server knows about that bridge, and now it knows the printer on that bridge. That's all the client does. on to the server claim code! Tong story short, it mostly [lives here](https://github.com/nordprojects/sirius/blob/master/sirius/coding/claiming.py). So when you type a claim code into the website, [it gets unpacked to find the bridge ID and the device ID](https://github.com/nordprojects/sirius/blob/master/sirius/coding/claiming.py#L70-L89) and then it associates that device to your account. voila! THE GOOD NEWS IS that in searching for that, I found [an actually documented summary of what claim codes are](https://github.com/nordprojects/sirius#claim-codes).

KTamas: and after all this, it appears permanently on device.li?

Josh: Yurp, essentially. I guess technically "until someone else claims it", if the device is reset etc. but I don't know if that regenerates the ID, or if it's truly based on the mac address I guess it could be easy enough to make a lil microsite that generates new addresses + claim codes, to make this part easier.
