---
title: "Quake on WASM With WebSocket multiplayer"
date: 2024-05-12T00:53:44+08:00
draft: false
---

## Genesis

When I was a kid, my dad taught computing / programming at [Alanvale TAFE](https://www.tastafe.tas.edu.au/campuses/alanvale-campus); he would often bring home old computers from work (I think when they upgraded their machines) and network them together, so we always had computers around the house and I was interested in them from an early age.

The first vivid nightmare I remember was when my older sister let me play a bit of [Wolfenstein 3D](https://en.wikipedia.org/wiki/Wolfenstein_3D) and I guess it was on her save game where you confront Hitler.

I just remember walking through one of the doors and there he was with these big machine guns blazing, I nearly leapt backwards through the seat; he was so real, so lifelike- the graphics were incredible:

![image-1](/posts/quake-on-wasm-with-websocket-multiplayer/image-1.png)

Well, they sure seemed incredible at the time- clearly the graphics weren't that important, the game was plenty gripping as it was.

I remember the first network multiplayer game I ever played; me and my sister were at work with dad, I think it was the start of the year and he and another teacher were just getting stuff set up, so he brought us in and the other teacher brought his kids in (also a boy and a girl).

The other kids took us to one of the computer labs and got us all set up with a game of [Doom](<https://en.wikipedia.org/wiki/Doom_(1993_video_game)>). I remember being confused as to where the enemies were, the level was just vast an empty- but briefly out of nowhere an enemy appeared moving so fast and just gunned me down where I stood.

It happened a few more times and I remarked "wow they're so fast, how do you even kill them?" which prompted one of the other kids to explain that it was them that I saw and that we're all playing together in the same game- it absolutely blew my mind.

I was hooked after that, multiplayer was the future and I was lucky enough to live in it.

Doom came out in 1993, so I guess I would have been about 8 at the time.

![image-2](/posts/quake-on-wasm-with-websocket-multiplayer/image-2.png)

Outside of that experience, I never really played a lot of Doom- I'm not sure if my parents shut it down or what, but I mostly played racing games and side-scrollers.

Around 1995 / 1996 I started to get properly into first-person shooters; I remember playing [Rise of the Triad](https://en.wikipedia.org/wiki/Rise_of_the_Triad), [Hexen](<https://en.wikipedia.org/wiki/Heretic_(video_game)>), [Strife](<https://en.wikipedia.org/wiki/Strife_(1996_video_game)>) and [Duke Nukem 3D](https://en.wikipedia.org/wiki/Duke_Nukem_3D), but the game I remember most of all...

![image-3](/posts/quake-on-wasm-with-websocket-multiplayer/image-3.png)

The original [Quake](<https://en.wikipedia.org/wiki/Quake_(video_game)>).

It was groundbreaking- I think it was the only first person shooter at the time with real 3D graphics. It was so dark and gloomy, the enemies were scary, if you used the explosive weapons just right you could cause the enemy to explode into a shower of bodyparts: "[gibbed](https://quakewiki.org/wiki/Gib)".

But above all, it had multiplayer- both cooperative and deathmatch and boy did I play a lot of it; I'd rally my sister, my nephews and the kid from across the road to occupy some of the computers we had laying around so that we could play Quake, they didn't mind it, but they didn't enjoy it nearly as much as me.

For years I only had the shareware version; I remember having a magazine that had a review of the full version and I would read it over and over wondering how awesome it would be.

I can't remember how or when, but eventually I was able to buy a copy of the full version and it was as good as I hoped it would be, if not better. Me and the gang all started getting serious about deathmatch, now that we had all these dedicated deathmatch levels; mods started getting popular and we installed some bots which made the games even more frantic, it was so much fun.

Anyway, years went by, other games came and went- I played a lot of Counter Strike 1.6, I played a lot of Quake 3 but I'd regularly come back and play some plain old Quake. I dunno if it truly was that good or if it was just that nostalgic feeling of having my mind blown by how incredible a game was coming back to me.

I've certainly never experienced it since, and there are some very cool games out there these days.

## Paths cross

So, fast forward to 2024, some 28 years later and I don't really play games that much these days; I'm pretty busy writing code for work and I'm pretty busy being a dad- but Quake has just started finding its way back into my life again.

I noticed it in the Xbox game store, so I installed it and it was terrible- there was nothing wrong with it, it was exactly Quake, but I think I just really hate playing first person shooters with a controller. My son was pretty interested though, but at 4 years old he's probably best not to play that sort of game (my wife is already a bit annoyed at how he's growing a fascination for guns, despite neither of us introducing them to him).

As tends to happen I find a way to weave the aspects of my life together and in this case, while poking around I discovered that [some nice guy](https://www.m-h.org.uk/) had [ported Quake to WASM](https://github.com/GMH-Code/Quake-WASM).

I spun it up, it all worked nicely in the browser and of course my immediate question was "does multiplayer work?"- it turns out the answer was "probably", but unfortunately in practice, not really.

The porter had used [Emscripten](https://emscripten.org/) which is a great choice for this sort of thing and Emscripten promises to provide [a degree of transparent BSD-socket-to-WebSocket](https://emscripten.org/docs/porting/networking.html?highlight=network#emulated-posix-tcp-sockets-over-websockets) translation out of the box; such that if the code that previously didn't know about WASM tries to do stuff with BSD sockets, Emscripten will open a WebSocket to a URL that you can inject and write bytes to it / read bytes from it.

This didn't seem to work for Quake and I couldn't figure out why- surely it was just a UDP client socket connecting to a UDP server socket? That's exactly what this sort of thing is built for!

Unfortunately not. I did a bunch of research and it seems the original Quake protocol (often referred to as "NetQuake") has all sorts of issues (and this is why [QuakeWorld](<https://en.wikipedia.org/wiki/Quake_(video_game)#QuakeWorld>) sprung up as a more favoured deathmatch-only Quake multiplayer client and server).

The practical issue was that instead of doing the sensible, NAT-friendly thing described above (i.e. a client opens a socket to a server using `socket.h::connect()`, that socket stays open for bidirectional messaging), NetQuake uses `socket.h::bind()`, so the socket is listening on a given port and anybody can send it datagrams.

Which is exactly what happens; you send datagrams to the Quake server on the control port of `26000` and it will respond to you on that port, but after the initial handshake, the communications will move to another completely random port (advised in the handshake) meaning the server has to have a lot of ports open and exposed to the internet and so does the client!

This caused some problems for the Emscripten piece; typically you'd run a simple proxy like [websockify](https://github.com/novnc/websockify) and call it a day- but that won't work here, because the Quake server will send post-handshake messages back to the proxy from a new port (that the proxy doesn't recognize).

So I had no choice but to write a [custom Quake WebSocket proxy](https://github.com/initialed85/quake-websocket-proxy) in order to keep trying- unfortunately I still didn't get it working, I think more of the "faked out BSD sockets" approach was upsetting the system; things like address comparison checks (inside the Quake client) to know if a connection is good etc were failing resulting in the connection not progressing.

I thought long and hard about approaches that wouldn't have me writing any C (language of original Quake source code) but I couldn't come up with anything; this left me no choice other than implementing a new `net_landriver_t` for WebSockets, for Quake to use; here's the interface I needed to implement:

![image-4](/posts/quake-on-wasm-with-websocket-multiplayer/image-4.png)

Doesn't look too bad! And I won't need a fair bit of it as well- fortunately Emscripten has a nice [C abstraction built around the JavaScript WebSocket client](https://github.com/emscripten-core/emscripten/blob/main/system/include/emscripten/websocket.h).

So, I [went and wrote the implementation](https://github.com/initialed85/Quake-WASM/blob/master/WinQuake/net_websocket.c)- it was pretty fun to be honest; I can see there's a certain beauty and power to C, but I don't think I'd want to be writing it as part of a team that had juniors; too much to go wrong.

## An interesting challenge

C doesn't include a lot of batteries and it looks like [id Software](https://www.idsoftware.com/) at least had built a lot of their own things that you might find in the standard library of other languages- I don't know enough about C to know if this is typical.

I found myself wanting a FIFO queue in order to couple the asynchronous callbacks for WebSocket `onmessage` to the synchronous (but non-blocking) `Read` function I had to implement in `net_landriver_t` but I couldn't find anything immediately standing out as available, so I wrote a simple and gross one:

![image-5](/posts/quake-on-wasm-with-websocket-multiplayer/image-5.png)

Writing to it is pretty straightforward (and should probably be in a function):

![image-6](/posts/quake-on-wasm-with-websocket-multiplayer/image-6.png)

And so is reading from it:

![image-7](/posts/quake-on-wasm-with-websocket-multiplayer/image-7.png)

Is this idiomatic C? I dunno, probably not if I had to guess- but it does work!

Also you might notice the message queue item index as an char- I thought this was a bit clever because now I don't have to think about pruning the buffer or moving the index backward or anything; the read cursor and write cursor can just chase each other all day and it should all be fine... unless the write cursor laps the read cursor, then I dunno what'll happen.

Let's just hope we don't fall behind!

## Wrapping up

The main takeaway of all this is that you can go here and play [Quake multiplayer in your browser](https://quake.initialed85.cc/) (if my Kubernetes cluster hasn't crashed); there might be some bots in there, there might also be some other humans in there, who knows.

If you need some more bots, drop down the console and type `impulse 100`; courtesy of [Frikbot](https://www.insideqc.com/frikbot/fbx/).

If you want to change the map, click the RCON link and do something like `changelevel e3m4`.

Here are the repos:

-   [My fork of GMH-Code/Quake-WASM](https://github.com/initialed85/Quake-WASM)
-   [My fork of wyatt8740/Quake-LinuxUpdate](https://github.com/initialed85/Quake-LinuxUpdate)
-   [The proxy and general packaging](https://github.com/initialed85/quake-websocket-proxy)
