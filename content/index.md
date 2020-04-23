---
title: "Welcome!"
date: 2020-04-23T12:46:11+02:00
draft: false
---
# Table of Contents

<!-- TOC -->

- [Table of Contents](#table-of-contents)
- [So you want to have a Little Printer](#so-you-want-to-have-a-little-printer)
- [What you'll need](#what-youll-need)
- [How does this all work?](#how-does-this-all-work)
- [How to get started](#how-to-get-started)
- [Let's make a (fake) printer](#lets-make-a-fake-printer)
- [Wiring it all up](#wiring-it-all-up)
- [Credits](#credits)
- [FAQ](#faq)
- [What we need help with](#what-we-need-help-with)
- [History / timeline](#history--timeline)
- [Links](#links)
  - [Code](#code)
- [Other](#other)
  - [Misc notes ignore for now](#misc-notes-ignore-for-now)
  - [Addendum: how the whole fake printer thing works (by Josh)](#addendum-how-the-whole-fake-printer-thing-works-by-josh)

<!-- /TOC -->

# So you want to have a Little Printer

Good news! You came to the right place. Just follow these ??? easy steps and you'll be set up in no time* (a fair amount of time, actually.)

# What you'll need

* A thermal printer, like the Paperang P1. There are a number of places where you can buy one: Amazon ([US](https://www.amazon.com/PAPERANG-Portable-Language-Wireless-Bluetooth/dp/B07TCJ2TC6/), [UK](https://www.amazon.co.uk/PAPERANG-Wireless-Mobile-Instant-Printer/dp/B078HZK2NV), [DE](https://www.amazon.de/Paperang-Mini-Drucker-Android-Ger%C3%A4te-Druckpapier-gew%C3%B6hnlich/dp/B082PWV57H/)), [Thepaperang](https://thepaperang.com/products/paperang-p1), [Paperangprint](https://paperangprint.com/collections/paperang-printers/products/paperang-p1), [Aliexpress](https://www.aliexpress.com/w/wholesale-paperang-p1.html) and so on. Don't forget to buy a few rolls of paper for it; you can find them in many colors, and sticker paper is especially fun!
* Ideally, a Raspberry Pi with Bluetooth, but we've tested this with a Mac as well, and with some tweaks it will work on any computer with Bluetooth running *nix
* Node.js 10, Yarn, Python 3.7, Docker
* Ideally an iPhone for the [app](https://itunes.apple.com/us/app/little-printers/id1393105914?ls=1&mt=8), but you can use the [web client](https://littleprinters.netlify.app/) as well (with reduced functionality)

# How does this all work?

I'm going to steal this useful graph from [Nordprojects](https://nordprojects.co/projects/littleprinters/):

![architecture](how-it-works.png)
<small>© Nordprojects, 2019</small>

The original Little Printer architecture required two devices: the printer itself, and the bridge. The printer connected to the bridge via [Zigbee](https://en.wikipedia.org/wiki/Zigbee), and the bridge connected to the Berg Cloud (which was replaced by the `sirius` project — see below). The Berg Cloud Bridge acted as a platform for an envisioned ecosystem of IoT devices, that never really materialized. The client (`sirius-client` — we'll get to that soon) acts as the bridge and has various "drivers" for printers.

![new architecture](how-it-works-2.png)
<small>© Nordprojects, 2019</small>

# How to get started

We'll assume you have your Paperang. Great! You can already use via its app on your smartphone, but that's not what we'll do.

Connect your Paperang to a power source with a USB cable to prevent drain. It should also keep it turned on. You can communicate with the printer via USB but for now, we'll just use bluetooth.

Pair your Paperang with your computer over Bluetooth and make sure you have Node 10, Python 3.7 and Docker installed.

# Let's make a (fake) printer

This project is predicated on us prentending to be a Little Printer, so let's do that.

The easy way is visiting [this website](https://little-printer-claim-code.netlify.app/) that generates the required `*.printer` file for you.

The hard way is cloning the [`nordprojects/sirius`](https://github.com/nordprojects/sirius) repository from github, running `docker-compose -f docker-compose.yml -f docker-compose.db.yml -f docker-compose.development.yml up --build -d`, waiting for it to build, sshing into it (`docker-compose exec sirius bash`) and running `./manage.py fake printer`, which gives you the same result.

I recommend the easy way.

In the end, you have a file that looks like this:

```
     address: db708b77ae2ee5b5
      secret: 1011795836314
  claim code: fojy-q4xv-7pe2-xt00
```

Save it and guard it with your life; this is your printer now.

Next, we're going to claim the printer on the nord-sirious instance. Go to [https://littleprinter.nordprojects.co/](https://littleprinter.nordprojects.co/), sign in with Twitter, and click "Claim a printer". Enter the claim code you have and give it a name, then hit "Claim Printer".

Congratulations, you now have a fake Little Printer! Save the [`device.li`](https://device.li) address, you'll need it later to add the printer to the (web)app.

# Wiring it all up

Next, we'll put it all together, connecting our Paperang to the network. For that, we use two projects: [sirius-client](https://github.com/ktamas/sirius-client) and [python-paperang](https://github.com/ktamas/python-paperang). They should really be merged into one. One day.

Anyways, clone both of them from Github, and we'll start with some testing with the latter project. We're going to assume you have at least a passing understanding of both Python and Typescript.

If you're on Linux, and we'll assume Debian, you'll need the following packages installed: `libbluetooth-dev libhidapi-dev libatlas-base-dev python3-llvmlite python3-numba python-llvmlite llvm-dev`.

Create a `config.py` based on `config.example.py`. If you don't have the MAC address of your paperang, that's fine; just use an empty string and the code will find it. Set a temp directory that's writeable by both projects.

Run `printer.py` to print a self-test. If you did everything well, you should have a bunch of infos printed on your Paperang! You can also edit this file to print any arbitrary image, processed with the famous [Atkinson dithering algorithm](https://en.wikipedia.org/wiki/Dither#Algorithms).

Next, let's get `sirius-client` working. Install ts-node globally (`npm install -g ts-node`), then do a `yarn install`. Put your printer file into the `fixtures` folder, then edit `bin/client.ts` and point `printerDataPath` to it. Edit `src/device/printer/filesystem-printer.ts` and update it with the temp directory we use for communication that you created two paragraphs before.

Now run both projects (`python3 littlepriter.py` and `ts-node bin/client.ts`), and you're ready to print something!

Get the [app](https://itunes.apple.com/us/app/little-printers/id1393105914?ls=1&mt=8) (or use the [web client](https://littleprinters.netlify.app/)), add a printer via its `device.li` address, and print something! Or use the API. Go wild.

# Credits

[Tamas Kadar (KTamas)](https://ktamas.com/) ([email](mailto:ktamas@ktamas.com), [twitter](https://twitter.com/ktamas)), [Joshua May (notjosh)](https://notjosh.com/) ([twitter](https://twitter.com/notjosh)), Monica Farrell ([twitter](https://twitter.com/monica_farrell)). Watch the [!!Con 2020](http://bangbangcon.com/) talk ["Little Printing for Everyone!!1"](http://bangbangcon.com/speakers.html) by KTamas once it's published on YouTube. The "how does it work" image was created by [Nordprojects](https://nordprojects.co).

# FAQ

**Q: Why?**  
A: Because it's fun.

**Q: Do I need a Paperang P1, specifically? Can't I just use something else?**  
A: Of course you can! `sirius-client` already has a generic `escpos` driver, which lets you connect a wide variety of thermal printers that use the ESC/POS protocol. It also has a simple console driver, that simply show the image your printer would print. There is nothing stopping you from making your own driver to your favorite printer of choice.

**Q: What about the other Paperang models (P2, P2S)?**  
A: They will work... sort of. The thing about them is they print at a higher resolution. The P1 has the exact same resolution as the original Little Printer, and the server sends pixel-perfect bitmaps to the client, hence this is not a problem with that. With the newer models, you either have to add support to the server to make higher-resolution images or resize the images on the client-side.

**Q: Where can I get a real Little Printer?**  
A: I mean, eBay, if you're lucky. I've been looking for a while now, and it seems like noone wants to sell theirs.

# What we need help with

* Reviewing PRs for `sirius`, merging headless chrome
* Porting `python-paperang` to Typescript and/or improving communcation between that and `sirius-client`
* Building lots of cool services
* Checking out Paperang clones, see if they work with the library or not

# History / timeline

* (November, 2011) [Hello Little Printer, available 2012](https://vimeo.com/32796535) (Product announcement link)
* (March, 2012) [Little Printer homepage](http://web.archive.org/web/20120301003906/http://bergcloud.com/littleprinter/)
* (May, 2014) [Berg Cloud shop](http://web.archive.org/web/20140529015642/http://uk-shop.bergcloud.com/) 
* (July, 2014) [Little Printer homepage](http://web.archive.org/web/20140722111616/http://littleprinter.com/)
* (September, 2014) [Week 483](http://web.archive.org/web/20151019041519/http://blog.bergcloud.com/2014/09/09/week-483/) (Berg is shutting down)
* (September, 2014) [The future of Little Printer](http://web.archive.org/web/20150207025053/http://littleprinterblog.tumblr.com/post/97047976103/the-future-of-little-printer)
* (March, 2015) [Trying something](http://web.archive.org/web/20151025093243/http://littleprinterblog.tumblr.com/post/112431535978/trying-something) (Announcing `sirius`, the new backend for the Berg Cloud Bridge)
* (March, 2015) [genmon/sirius](https://github.com/genmon/sirius)
* (March, 2015) [A little longer](http://web.archive.org/web/20151025164214/http://littleprinterblog.tumblr.com/post/115018012993/a-little-longer) (Keeping the servers running for a few more months)
* (December, 2015) [End of 2015](http://web.archive.org/web/20151025164214/http://littleprinterblog.tumblr.com/post/115018012993/a-little-longer) (People working on Sirius, Bridge can be updated to use it)
* (December, 2015) [Updating the Bridge](https://github.com/genmon/sirius/wiki/Updating-the-Bridge)
* (December, 2015-July, 2019) [Directing the Berg cloud bridge](https://github.com/genmon/sirius/issues/8) (long GH issue, important historical document about the "lost years")
* (May, 2019) [Little Printer returns as an open-source messaging device](https://www.theverge.com/2019/5/19/18628287/little-printer-berg-nord-projects-app-open-source-messaging)
* (May, 2019) [Little Printers, a friendly new messaging app and cloud platform.](https://nordprojects.co/projects/littleprinters/) (Nord Projects' announcement of the new app, working server, api, device.li etc.)

# Links

## Code

* [nordprojects/sirius](https://github.com/nordprojects/sirius) (the server's code)
* [nordprojects/littleprinters-ios-app](https://github.com/nordprojects/littleprinters-ios-app) (the iOS app's code)
* [notjosh/sirius-client](https://github.com/notjosh/sirius-client) (notjosh's universal client for `sirius` — WIP)
* [ktamas/sirius-client](https://github.com/ktamas/sirius-client) (KTamas' fork of the client — older, but works)
* [ktamas/python-paperang](https://github.com/ktamas/python-paperang) (the python library that connects to the Paperang P1, with the protocol reverse-engineered)

# Other

## Misc notes ignore for now

BERG Cloud Bridge sits by your broadband router and wirelessly connects Little Printer to the web, which makes it easy for you to place Little Printer where you can see it.

British Experimental Rocket Group


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
So at that point, the server knows about that bridge, and now it knows the printer on that bridge. That's all the client does. on to the server claim code! Tong story short, it mostly lives here:
https://github.com/nordprojects/sirius/blob/master/sirius/coding/claiming.py. So when you type a claim code into the website, it gets unpacked to find the bridge ID and the device ID: https://github.com/nordprojects/sirius/blob/master/sirius/coding/claiming.py#L70-L89 and then it associates that device to your account. voila! THE GOOD NEWS IS that in searching for that, I found an actually documented summary of what claim codes are: https://github.com/nordprojects/sirius#claim-codes.

KTamas: and after all this, it appears permanently on device.li?

Josh: Yurp, essentially. I guess technically "until someone else claims it", if the device is reset etc. but I don't know if that regenerates the ID, or if it's truly based on the mac address I guess it could be easy enough to make a lil microsite that generates new addresses + claim codes, to make this part easier.