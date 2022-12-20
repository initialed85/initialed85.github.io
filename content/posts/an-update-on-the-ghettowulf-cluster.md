---
title: "An Update on the Ghettowulf Cluster"
date: 2022-12-20T15:49:42+08:00
draft: false
---

So it's been about 2 months since I last wrote about this thing and it had been running for a couple of weeks before I wrote about it, which
means it's been operating for maybe 3 months all up.

Despite a few initial teething issues, I have to say it's been pretty good (and definitely less maintenance than my old setup was).

Here's how it looks at the moment:

![Image 1](/posts/an-update-on-the-ghettowulf-cluster/image-1.png)

As you can see, cable management is my passion.

I've even gone as far as setting up some monitoring:

![Image 2](/posts/an-update-on-the-ghettowulf-cluster/image-2.png)

![Image 3](/posts/an-update-on-the-ghettowulf-cluster/image-3.png)

![Image 4](/posts/an-update-on-the-ghettowulf-cluster/image-4.png)

![Image 5](/posts/an-update-on-the-ghettowulf-cluster/image-5.png)

[Promscale](https://github.com/timescale/promscale) is dead, long
live [VictoriaMetrics](https://github.com/VictoriaMetrics/VictoriaMetrics).

## Learnings

It hasn't all been sunshine though; here are some of the things that went wrong...

### User error

While trying to fix some Ceph-related issues, I thought I could safely just blitz the whole `rook-ceph` namespace and redeploy it, expecting
everything to just pick up where it left off because the identity of the storage cluster is on the physical disks, right?!

That was an incorrect assumption- probably my files were still there, locked away in the numerous layers of abstraction and spread across my
nodes, but all of the context around how to access them was long gone when the Persistent Volume Claim was deleted.

So I had no choice but to make peace with an enforced spring clean of my old garbage files; fortunately nothing important, a week or so of
CCTV footage and some scanned documents that only needed to live long enough to email them out).

### Swap configuration

If you're a Kubernetes OG you may be of the mindset that
a [Kube node should not have swap enabled](https://github.com/kubernetes/kubernetes/issues/53533#issuecomment-334947625) thus
empowering [node-pressure eviction based on `MemoryPressure`](https://kubernetes.io/docs/concepts/scheduling-eviction/node-pressure-eviction/#node-conditions)
and then all of your [ridiculously vendor-locked automation](https://docs.aws.amazon.com/cdk/index.html) can spin you
up [a new Kube node](https://docs.aws.amazon.com/eks/index.html) faster than you can say "credit card".

Unfortunately if your cluster is managed by yourself and not by [Uncle Jeff](https://www.instagram.com/p/CYMawuhF-bM), losing a node isn't
something that automatically resolves itself for a fee, it's quite an inconvenient problem because now you need to physically access your
nodes in order to restart them (a significant inconvenience requiring a ladder if, like me, you thought it was a good idea to put your
cluster on a shelf above your fridge).

My experience was that for _reasons_ I would find that one or more of Kube nodes would completely lock up (after exhausting all their
memory) and because there was no swap, the Linux kernel had quite literally nowhere to put the unused stuff in order to try and recover the
situation with a healthy bit of [OOM Killing](https://utcc.utoronto.ca/~cks/space/blog/linux/OOMKillerWhen) and so everything fell in a
heap.

Fortunately for me, I discovered
that [recent versions of Kubernetes actually have support for nodes with swap memory enabled](https://kubernetes.io/blog/2021/08/09/run-nodes-with-swap-alpha/);
after re-enabling swap on all my nodes, suddenly my problems just disappeared- so I guess the lesson for me here is to second-guess what
seems like conventional wisdom when it's telling me to do things that I wouldn't otherwise do (because in all non-Kubernetes cases I would
always have swap, I just underestimated my own actual experiences in that department and overestimated the dated Kubernetes guidance).

### Watch all the dogs

I don't really actively develop [cameranator](https://github.com/initialed85/cameranator) any more (short of
the [recent UI refactor](https://github.com/initialed85/cameranator/commit/68f0b5ba7ac1cbf352ec2d5877dd740dc3dede69)); since forever though
it's had this odd bug where parts just hang up randomly- I'm pretty sure it's not my code, because I see it in
the [Segment Generator](https://github.com/initialed85/cameranator/blob/master/pkg/segments/segment_generator/segment_generator.go) (which
is just some orchestration I've written
around [ffmpeg](https://ffmpeg.org/)) as well as
in [Motion](https://motion-project.github.io/).

The common theme? They both touch the GPU... and they both use ffmpeg underneath- so probably the issue is
ffmpeg. [Side note](https://www.youtube.com/watch?v=9kaIXkImCAM).

I tried to upgrade everything to be using later versions of ffmpeg, I failed miserably and decided it wasn't worth my time; so what else can
I do?

[Well a watchdog of course](https://en.wikipedia.org/wiki/Watchdog_timer), what else!
I [slapped together a way to expose liveness](https://github.com/initialed85/cameranator/commit/2d8489f34d3fac9d1eba2607e48a3f061856244a#diff-ba3e0ae0f7bebeb946e335c0c2107b27b9f623a2a783f1b059cff1cde6f8dfe1)
and then made use of that with
a [`livenessProbe`](https://github.com/initialed85/cameranator/commit/2d8489f34d3fac9d1eba2607e48a3f061856244a#diff-324aeb1b9f75967f6ac0a2b0e1d8bc7b1854b9f94c85ba55219937d27214c510R102) (
not unlike a [Docker Healthcheck](https://docs.docker.com/engine/reference/builder/#healthcheck)) so now this too is something that has
ceased to be a source of failures for me.

### So what's left?

I'm not really sure what to do next- for sure there are outstanding things, like:

- Get the Prometheus metrics from the Kube API and the Node Exporter to be using a common label so that I can relate them together
- Change from Traefik to Nginx because it's been nice using Traefik (the default with k3s) but frankly I'm an Nginx guy and I only have so
  many brain cells to dedicate to knowledge about reverse proxies
- Migrate Home Assistant from Docker to Kube
    - I can't even remember why I haven't done this yet, I got stuck on something mundane and just sort of stopped putting time into it-
      might have been reverse proxy `Host` header related?
- Physically tidy up the shelf
    - I think I can use
      a [document stand like this](https://www.officeworks.com.au/shop/officeworks/p/wire-vertical-file-organiser-large-black-ja02852) to
      get the laptops to take up less footprint
- Give Ceph the disks currently owned by the the ZFS pool on the HP Microserver
    - The Home Assistant migration is blocking this; also I should _probably_ buy
      some [external caddies](https://www.amazon.com.au/ORICO-Toolfree-External-Enclosure-Support/dp/B00GAML7OK) so that I can distribute
      those drives across the cluster (rather than having a huge chunk of storage on a single node)
- Migrate [dinosaur](https://github.com/initialed85/dinosaur) from Docker to Kubernetes
    - This is non-trivial because its whole mechanism is enabled through orchestrating the creation of Docker containers
- Deploy [my entry](https://github.com/initialed85/eds-game-for-ftp-game-jam-2022) for
  the [FTP Game Jame 2022](https://itch.io/jam/ftp-gamejam) on the cluster
    - More on this thing in the next post

Honestly though, I dunno how much of that I'll do- I've achieved the secondary goal of migrating (most of) my stuff to Kubernetes and in
doing so I've achieved my primary goal of learning about Kubernetes; for sure I don't know everything about it but I feel like I've been
exposed to a reasonable chunk of the capability it provides and got a feel for some of the patterns, enough to BS my way through an
interview for a job with it anyway!

![Image 6](/posts/an-update-on-the-ghettowulf-cluster/image-6.png)
