---
title: "Recent projects of note"
date: 2023-01-07T20:45:03+08:00
draft: false
---

I haven't really been finding [my job](https://www.linkedin.com/in/edward-beech-48442a74/) that interesting of late which normally results
in me having a lot of energy for my out-of-work pursuits.

Unfortunately for reasons I can't quite work out yet, I also haven't been motivated
to [make any music](https://soundcloud.com/initialed85/sets/the-loop) and so all that's really left is personal projects.

Anyway- the next few headings cover the things I've been up to.

## Minecraft

I had a bit of time off over Christmas and we all went to stay with my wife's family in Darwin where we enjoyed monsoonal weather and lots
of pool time.

One of my nephews is wild about Minecraft (likes to watch streamers and their mods etc) but only has the Bedrock edition for his Xbox, so I
bought him a copy of Java edition for Christmas.

I figured it was as good an excuse as any to try to run a Minecraft server yet again after numerous failed attempts (usually I just get
bored); naturally I had to run it on the [Ghettowulf Cluster](/posts/an-update-on-the-ghettowulf-cluster/) to keep it
interesting for myself.

Here's where we ended up:

- A [Fabric](https://fabricmc.net/) Minecaft server using the incredibly
  flexible [itzg/minecraft-server](https://github.com/itzg/docker-minecraft-server) Docker image
- Game sessions come through the Nginx load balancer running on a [Vultr](https://www.vultr.com/) VPS
- Game data stored in my [Rook Ceph](https://rook.io/) storage cluster
- Nightly backups to [Backblaze B2](https://www.backblaze.com/b2/cloud-storage.html) using
  a [Kubernetes CronJob](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/)
- Bedrock clients supported using [Geyser](https://github.com/GeyserMC/Geyser)
    - Not well tested, but I was able to join my Java server from my Bedrock on my Android phone- my plan is to support players joining from
      an Xbox (so my little neice can play)

![Image 1](/posts/recent-projects-of-note/image-1.png)

![Image 2](/posts/recent-projects-of-note/image-2.png)

## New Kubernetes node

Well, not strictly new- I had an old gaming PC that I'd taken to work to use as a build machine months ago.

It just stopped working one day so I figured I'd take it home and try to work out what was wrong with it.

The symptoms were that it would turn on, sometimes show some display and try to boot and then eventually turn off; I figure it was probably
dried out thermal paste and decided to sort that out.

Practically, the process went like this (with my toddler son "helping" me throughout):

- Remove CPU and cooler
- Leave PC open on floor of my home office
- (Some days later)
- Sit down to apply thermal paste and hope for the best

However unbeknownst to me, my son had decided to do a bit of autonomous fixing on his own while I wasn't around:

![Image 3](/posts/recent-projects-of-note/image-3.png)

My fault for leaving it open really!

I entertained ideas of straightening out the pins but I've never had a steady hand and my eyes are terrible these days so I figured no time
like the present for an upgrade.

![Image 4](/posts/recent-projects-of-note/image-4.png)

So we've gone from
an [Intel i5 9400f](https://www.intel.com.au/content/www/au/en/products/sku/190883/intel-core-i59400f-processor-9m-cache-up-to-4-10-ghz/specifications.html)
to an [AMD Ryzen 5600G](https://www.amd.com/en/products/apu/amd-ryzen-5-5600g) and I'm pretty pleased with it- AMD make good stuff lately.

I can't get my [Nvidia GeForce RTX2060](https://www.msi.com/Graphics-Card/GeForce-RTX-2060-VENTUS-XS-6G/support) to work in there though- no
idea if it's a new motherboard thing, an integrated Radeon GPU thing or what, but I don't desperately need it working so it's on the shelf
for now; if I get a 4th CCTV camera I'll be forced to either work out the Nvidia GPU or
refactor [cameranator](https://github.com/initialed85/cameranator) to support Radeon GPUs (maybe- maybe it'll still be within capacity for
my 3 GPUs?).

It's quickly overtaken my old HP server as "largest ability to do some work" and Kubernetes has adapted accordingly (as permitted
by [`nodeSelector`](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/) of course!)

![Image 5](/posts/recent-projects-of-note/image-5.png)

And similarly Rook Ceph set about balancing the storage in my cluster (it's got a 1TB spinning disk in there that I gave to Rook Ceph):

![Image 6](/posts/recent-projects-of-note/image-6.png)

While I was at it, I dug out my [Dremel](https://www.dremel.com/au/en/p/dremel-4000-f0134000nd) and hacked up
the [document stands I bought from Officeworks](https://www.officeworks.com.au/shop/officeworks/p/wire-vertical-file-organiser-large-black-ja02852)
and was able to tidy up the [ap-maylands-1](https://en.wikipedia.org/wiki/Maylands,_Western_Australia) datacenter shelf a bit:

![Image 7](/posts/recent-projects-of-note/image-7.png)

The large case is the new node- I sure am putting some faith in those [Bunnings](https://www.bunnings.com.au/) shelf brackets!

## A worse and more complicated RC car

A very long time ago I picked up [one of these](https://www.hosim.com/products/9155-blue) with an intention to make some sort of project of
it- I've done this [once before](https://github.com/initialed85/pi-rc-car) with [Pygame](https://www.pygame.org/news) for
a [different RC car](https://www.tamiya.com/english/products/58347lunchbox/index.htm) but not very well.

I stumbled across some challenges when I opened the Hosim car up:

- Radio and speed controller are integrated, making it hard to inject my own signals and pretend to be the radio
    - On opening this up, I discovered the speed controller was fully potted, so I couldn't even solder my way around this
- Steering servo was the 5-wire type (meaning the integrated radio / speed controller was also part of a steering servo)

This time I wanted to use [Rust](https://www.rust-lang.org/) and [Bevy](https://bevyengine.org/) because Rust is awesome and Bevy is also
awesome (and handles all the hard parts of input and scheduling for me).

Here's where it ended up:

- Rust + Bevy native client running on MacOS (and probably anything)
    - Incoming input events consumed from a PS4 controller
    - Outgoing input events produced as UDP datagrams at 30 Hz
        - [MessagePack](https://github.com/3Hren/msgpack-rust) used as serialization
- Rust + [Tokio](https://github.com/tokio-rs/tokio) native client running on a Raspberry Pi 3
    - Incoming UDP datagrams consumed containing input events
    - [PWM](https://en.wikipedia.org/wiki/Pulse-width_modulation) signals produced for steering servo and speed controller

The [code can be found here](https://github.com/initialed85/rc-things); interesting observation, it's really not much code! Certainly seems
to be less than my prior Python attempt. Rust is just so good.

Here are the parts of note:

- A [HobbyKing KL15148B](https://hobbyking.com/en_us/hobbykingtm-hk15148b-mg-digital-servo-1-7kg-0-18sec-21g.html) aircraft servo for a
  steering servo
    - I inherited this with a bunch of other RC aircraft stuff from [a friend](http://blog.84ace.com/) years and years ago so, so this
      seemed like a good way to [reuse some old stuff](/posts/permacomputing/) and work around my 5-wire steering servo issue
- A [Hobbywin Quickrun 1060 speed controller](https://hobbytechtoys.com.au/products/hobbywing-quicrun-60a-brushed-sbec-wp-esc)
    - I wrestled with the [old speed controller from the Tamiya](https://www.rcjaz.com.au/tamiya-90549-rc-esc-teu105bk-bulk-p-90055353.html)
      but I think it's translation from input PWM to output PWM wasn't right for the beefier motor from the Hosim (the lowest possible duty
      cycle was way too fast); this new speed controller has been exceptional, I can go flat out and I can crawl along barely moving.

I also had a challenge around the woeful performance of the Raspberry Pi 3's WiFi chipset / antenna combo and I worked around this by
separating the concern of WiFi connectivity using a [GL.iNet GL-MiFi](https://www.gl-inet.com/products/gl-mifi/) that I had lying around as
a wireless client with an [iptables](https://linux.die.net/man/8/iptables) rule passing the received UDP on to the Raspberry Pi (which is
connected via Ethernet).

Naturally I used [OpenWrt](https://openwrt.org/) for this piece because it's great and it makes doing router stuff really simple.

Here's the car in motion:

{{< video "/posts/recent-projects-of-note/video-1.mp4" "video-1" >}}

Here's a nice glamour shot:

![Image 8](/posts/recent-projects-of-note/image-8.png)

The batteries are on top because there's no longer any room inside but it does make it nice and easy to remove them for charging- however it
does also make it handle hilariously poorly with such a high center of gravity.

Here's how the innards look with the body as far off as the WiFi antenna will allow without being unplugged:

![Image 9](/posts/recent-projects-of-note/image-9.png)

I love it, the range is way worse (I can basically only drive it in my house and on the street outside my house), the handling is worse, the
technology is way more complicated but it's my mess dammit and over-complicating things is what I'm paid to do so really I'm just plying my
trade here.

Some TODO items:

- A camera for remote control
- 4G / LTE for even remoter control ([Tailscale](https://tailscale.com/) naturally to VPN in)
- GPS for localisation

Hell, could I use an old Android and cover off on camera and localisation and do other cool stuff with the onboard sensors? It could even be
the 4G / LTE backhaul! I shouldn't have mailed all my old phones to my dad...

## So what's next?

Well I've got a mountain of unfinished projects, the next two that I wanna make some progress on are:

- PS4 controller + Rust + Bevy to control a [DJI Tello](https://www.ryzerobotics.com/tello) that I've had forever
    - The only way I have to control it right now is the official Android app and it's garbage
- Build a robot on [this base kit](https://www.altronics.com.au/p/k1090-2wd-motorised-robot-building-base-kit/) that I bought and assembled
  with my son the other day
    - I've had a [Husarion Core2-ROS](https://husarion.com/manuals/core2/) gathering dust for literally years; I think you're supposed to
      use it with [ROS](https://www.ros.org/) but I'm on the fence about that- it's been superseded
      by [ROS2](https://docs.ros.org/en/foxy/index.html) which in my experience is even worse to develop for on MacOS and I dunno what
      Raspberry Pi support is like so who knows
