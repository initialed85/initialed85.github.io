---
title: "How to write code like a Dinosaur"
date: 2022-09-18T12:43:02+08:00
draft: false
---

I've been working on a personal project called [Dinosaur](https://github.com/initialed85/dinosaur); it's a single-page application that
presents the user with two horizontal panes- a code editor in the language of your choice and a live feed to your code being executed.

Languages presently supported are:

- C
- Go
- Java
- Lua
- Python
- Rust
- TypeScript

Initially this sounds a bit like any of the various coding games / exercises out there but I've chosen to add what I think is a slight
variation in that all user sessions have network stack access to the same network as all other user sessions (in time I'll probably permit
this to be broken up into groups, permitting isolated group sessions).

You can access a running instance of it at [dinosaur.initialed85.cc](https://dinosaur.initialed85.cc/); have a play, see if you can break
it, feel free to exploit any glaring security holes I've missed.

It may sound kinda cool, but as you'll find out if you read on it's mostly just an exercise of plumbing together excellent software that
other folks have already built.

## Rationale

The idea for this came about as part of an ongoing friendly argument at work that sees our team of developers divided into two camps:

- The new, sharp developers that are masters of the clever features built into their IDEs
- The old, decaying "dinosaur" developers that are scared by clever features and use their IDEs more like an editor

Those of us in the new camp may often steer clear of things like the shell, favouring instead a rich UI as we churn out solutions at a high
velocity while those of us in the old camp waste time reading and understanding code and spend far too long typing
long-winded [Bash](https://en.wikipedia.org/wiki/Bash_(Unix_shell)) commands to achieve what the UI can do in mere seconds.

As with a lot of these arguments, neither party is absolutely correct, each simply gravitates towards one end of the spectrum as a
preference because it most gels with the way they work.

Probably if we were to be objective and pragmatic, a good developer would have a functional degree of mastery over both approaches thus
maximising the tools available to them.

I'll leave the reader to speculate as to where on the spectrum I would place myself!

## Architecture

The system looks something like this as an overview:

![Image 1](/posts/how-to-write-code-like-a-dinosaur/image-1.png)

And if we make the interesting parts a bit more transparent:

![Image 2](/posts/how-to-write-code-like-a-dinosaur/image-2.png)

Here's a quick summary of the different technologies in there:

- [Nginx](https://www.nginx.com/)
    - Web server / reverse proxy
    - Selected because it works well and I already know how to use it
    - Used to host the static content (TypeScript / React) and proxy the backend service
- [Go](https://go.dev/)
    - Language for compiling native executables
    - Selected because it has good built-ins for HTTP backend services and a great concurrent performance
    - Used as the language to write the backend service
- [Docker](https://www.docker.com/)
    - Orchestration system for Linux containers
    - Selected because it works well and I already know how to use it
    - Used both functionally to provide isolation for a user session and non-functionally as a deployment mechanism
- [GoTTY](https://github.com/sorenisanerd/gotty)
    - Batteries-included utility for exposing a shell command via HTTP
    - Selected because it made an otherwise detailed problem extremely easy
    - Used to provide the live "build and run" feed for a user session
- [entr](https://github.com/eradman/entr)
    - Executes a given command on changes to a watched filesystem path
    - Selected because it made an otherwise detailed problem extremely easy
    - Used to manage the "filesystem change-driven build and run" loop for a user session

## Lowering risk of exploitation

Given I haven't (as yet) hidden this thing behind any sort of social sign-in and have no other smarts to limit access, I wanted to try and
reduce the amount the system could be exploited (e.g. as part of a botnet or who knows what).

Docker makes this easy for me thanks to its usage of [Linux namespaces](https://en.wikipedia.org/wiki/Linux_namespaces) and various other
bits of sugar; here's what I've done:

- Each user session is in its own Docker container
- Each user session Docker container has [CPU](https://docs.docker.com/config/containers/resource_constraints/#cpu)
  and [RAM](https://docs.docker.com/config/containers/resource_constraints/#memory) resource constraints applied
- All user session Docker containers are in an internal Docker network with no internet access

I've not done a great deal of testing (this is a personal project and tests are too much like work!) but I'm hoping that this will help if
somebody tries to cripple the system with some really CPU or RAM hungry code.

## Approach for supported languages

I struggle with motivation for all of my personal projects because they often don't really serve any useful purpose; this is well evidenced
by my piecemeal home-automation-centric projects:

- [cameranator](https://github.com/initialed85/cameranator)
    - Home-grown CCTV motion detection and segment recording system
- [mqtt_things](https://github.com/initialed85/mqtt_things)
    - Some hacking to expose a few things via MQTT
- [zmote](https://github.com/initialed85/zmote)
    - A Python library for somebody else's low-volume-produced WiFi IR blaster

They all kinda work, they're all kinda used and none of them are reliable or complete or well tested.

This project has managed to keep me on the hook for a while because I get to play with a bunch of languages that I almost never used / had
previously never used; this has come about because (at least for now) easy supported language comes with a minimal template that does the
following:

- Open a UDP socket
- Send a 1 Hz (ish) heartbeat as a broadcast announcing itself in the main thread
- Receive broadcast heartbeats from other user session using a background thread

This sounds pretty minimal, but it's touching a few key concepts for each language that take you beyond simple things like using variables
and functions and printing output.

### C

I'm not at all experienced with C although it was part of my training long ago; I've written nearly nothing using it personally (maybe some
Arduino stuff here and there) and absolutely nothing using it professionally- C is OG, so it has to be a language that Dinosaur has support
for.

Because I don't know what I'm doing, my template may for sure have issues; for example no mutex on the socket interactions.

### Go

I'm pretty comfortable with Go both personally and professionally, but my approach
to [polyglotism](https://en.wikipedia.org/wiki/Polyglot_(computing)) is quite naive and reflects my lack of detail in that I tend to
write one language much like I write another with sort of a "lowest common denominator" approach (and as such I'm rarely maximising the full
features of a language).

For example a serious Go coder would probably have channels in front of the socket.

### Java

Java is like C for me; I've done very little of it personally and professionally, I was once taught it and deemed competent but I've
forgotten basically all of that.

Interestingly, it seems to be the heaviest when running the template code- could be a byproduct of my lack of understanding on the correct
way to write Java.

Practically it's quite verbose and while not really apparent with the small bit of template code for this project, it seems like it can be
quite an art to juggle JRE and various other dependencies.

### Lua

I added Lua because my Dad (a C coder from way back) constantly bangs on about it; we've had many light arguments about wherein I suggest
Lua is garbage (based on a brochure review and no practical experience) and so I needed to get some real experience with it.

As far as I can tell:

- Integrates well with C
- Has next to no standard library
- Has no built-in support for sockets (need external `luasocket` library)
- Has no built-in support for threads (just coroutines and you need to write your own dispatcher)
- The C-to-Lua interface (stack-based) needs care when achieving function interfaces because you just push values onto a stack (making
  it real easy to build an inconsistent returned interface for your varying code paths within a function); `luasocket` is an example of this
  causing bad interfaces

### Python

I'm pretty comfortable with Python, it's the language that I've done most of my professional coding with; however as with Go, the way I
write it is probably not as Pythonic as a serious Python developer.

### Rust

I have written almost zero Rust and I would absolutely love to write more (I just don't have an excuse, or didn't, until now).

It looks like it has an incredibly steep learning curve but despite how low-level it is (it feels like it's trying to solve the same problem
as C) it can be impressively terse.

This is the language I will be trying to learn the most about in the near future.

Simply getting the template code for this project to share a socket safely (at least as far as the borrow-checker is concerned) between the
main thread and the background thread took me days; the takeaways ended up being this:

```rust
// wrap a socket in an atomic reference counter; note the cool error handling
let socket = sync::Arc::new(net::UdpSocket::bind(local_addr).expect("failed to bind socket"));

// take a clone of the socket for safely "moving" into the thread from an ownership
let cloned_socket = socket.clone();

// spawn a thread and move the cloned socket into it
thread::spawn( move | | receive_loop(cloned_socket, local_ip));

// meanwhile in our main thread we can comfortably do this (don't even need to unwrap the atomic reference counter)
socket.send_to(
format!("Hello world from Rust @ {}", hostname).as_bytes(),
format!("{}:{}", broadcast_ip, PORT),
).expect("failed to send to socket");
```

Apparently with this we can have faith that when everyone is finished using despite rust not having a garbage collector per se- exceptional!

### TypeScript

We write a bit of TypeScript at work but it's all on the frontend; NodeJS gets a pretty good review for server-side stuff so I figured I'd
best include it.

Without question, it is super terse and that probably helps you get services up and running from nothing quite quickly.

I don't really like it- I had to hack more environment / dependency stuff in to get the TypeScript template working than I did for the other
languages; maybe it's okay if you're comfortable with it, but it feels like there's almost as much "fluff" around the edges of actually
achieving anything as Java (environment-wise).
