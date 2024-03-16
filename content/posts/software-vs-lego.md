---
title: "Software vs Lego"
date: 2024-03-16T17:16:59+08:00
draft: false
---

My son August (Augie to us, he's almost 4) always loved Lego (well, Duplo).

For the longest time he'd build really long rectangular trucks, he seemed to like getting every block packed in tightly on the tray / trailer(s) of the truck.

Lately though he's started building complicated trucks with lots of different levels and pieces, explaining things like "this truck has 3 hotels and a crane and 2 guns".

Here's one such example:

![Image 1](/posts/software-vs-lego/image-1.png)

# The challenge

These structures are often pretty unstable and collapse easily (especially if you try to move them) and he finds that really frustrating, but eventually he comes around and rebuilds it again.

I usually try to help him out and explain that he needs to repurpose some of his blocks for bracing (sacrificing some of the complicated parts of his structure so he can reliably keep other complicated parts of his structure).

We've only got a finite number of blocks, so there's a practical limit to the amount of complicated structure you can have and it's heavily affected by the amount of reinforcement you use along the way.

If you don't reinforce things at all, you can use all of your blocks to build a cool structure- but probably they'll collapse, especially the moment you try to interact with them.

On the flipside, if you over-reinforce things then your structure may not be cool at all, despite being extremely solid- this doesn't help us either, because now they're no fun to interact with.

# The comparison

Let's propose some rough counterparts between this Lego situation and software that represents a Minimal Viable Product.

## The box of Legos

This represents the pool of time and resources available; e.g.:

-   How big is our team?
-   How experienced is our team?
-   What technologies will we choose?
-   How long do we have to build our product?

These concepts don't individually map to aspects of playing with Lego, rather they're the budget:

-   Team size
    -   A small team can't make a lot of concurrent progress (risking our deadline due to limited throughput)
    -   A large team can make a lot of concurrent progress but requires coordination (risking our deadline due to poor organisation)
-   Experience
    -   An inexperienced team will be cheaper but may not be skilled enough and requires further coordination (risking our quality due to inexperience and our deadline due to poor organisation)
    -   An experienced team will be expensive but may constrain team size (back to risking our deadline due to limited throughput)
-   Technologies
    -   Bleeding edge technologies are exciting but uncertain (experienced developers will be engaged and excited but the technology may be unreliable and community non-existent)
    -   Proven technologies are reliable but boring (experienced developers may be unengaged and junior developers may struggle even with a well-used technology)
-   Time
    -   A tight timeline will necessitate efficiency (so we'll need experience, coordination and sensible technology choices); there's not much hope for an economic outcome here
    -   A relaxed timeline doesn't demand efficiency (so we can allow an inexperienced team to learn while making mistakes, an experienced team to pick exciting technologies); the difference is in the bill

## The structure we build

This is of course our product.

-   How many features do we have (e.g. individual user-facing parts of our system)?
-   How deep / detailed are the features (e.g. complexities within one of these user-facing parts)?
-   How well have we built the features (e.g. pragmatic use of abstractions, sensible code re-use, valuable automated test coverage)?

These concepts map quite directly to aspects of playing with Lego:

-   The number of features is the number of play areas made up of blocks
-   The depth of features is the complexity of each play area
-   How well-built the features are is the amount of bracing we've built the play areas with

Within reason, none of the above points on their own are anything to be concerned about when they're at extremes; the compounding effects need to be considered:

-   We can manage lots of features, if they're shallow
-   We can only manage a few features if they're deep
-   We can manage more feature area overall (lots of shallow features vs few deep features) if the features are well-built

## The time spent playing with the structure

This represents our desired users using our product.

-   Did we get any actual users?
-   Is it a good user experience?
-   Is it reliable and trustworthy?
-   Does it integrate well with adjacent workflows?

These concepts map somewhat to aspects of playing with Lego:

-   If we didn't want to play with our structure after building it, then we didn't get any actual users (implying failure of the product)
-   If playing with our structure didn't interest us very much, then our user experience is poor (implying poor uptake of the product)
-   If our structure kept breaking while we were playing with it, then our quality is poor and our uptime is poor (implying a high support cost)
-   If our structure wasn't applicable in the context of our other toys then it didn't integrate well

# The approach

The right thing to do here will depend heavily on who is involved, so I can really only speak for myself.

As a coder, I'm a pretty capable generalist (more backend than frontend and plenty of infrastructure) and as a leader I'm average (under pressure I fall back to isolation and detail and have to work very hard at collaboration).

I'm mostly going to look at this through the lens of team size / team experience and collaboration, but we can assume the best case for things that affect quality (tooling, pragmatic use of abstraction, sensible code re-use, valuable automated tests etc)- and by best case I mean best case for the experience level (so for juniors, that's probably not great).

Additionally, I am going to completely gloss over the most important part (that fails the most often) in terms of understanding your target market and building exactly (and only) what's necessary to penetrate that market- let's just assume that part all goes well (as well as to be hoped given the different pressures you'll be under for each approach).

## Build a large team of juniors and handle coordination and technology choices myself

Even a small team of juniors (for myself, as small as 3-ish) comes with significant overhead; either you're doing the right thing with lots of collaborative planning, pairing and mentoring (something extremely valuable for the growth of the developers, and for yourself) or you're not (and you're doing too much of the detailed work yourself).

In both cases, velocity is poor; in the first, you'll probably miss your deadline (and potentially never make it)- in the second, you might make your deadline but you'll be burnt out and your juniors won't have grown enough to help out by the time you go live.

Furthermore, without a sounding board for technology choices, you're just one error-prone human deciding what to use; you will make mistakes because there is no correct answer, there is only the most optimal choice for where you are and where you're headed (which is unique).

If you pick exciting technologies and adoption is low once you ship, you'll struggle to find talent to keep the lights on within your budget (nobody who wants to get paid $75k p/a can do Haskell and Kubernetes).

If you pick dull technologies and adoption is high once you ship, you'll struggle to attract talent despite your budget (nobody worth $150k p/a wants to do PHP and MySQL).

This approach risks burnout and dissatisfied devs.

## Build a small team of seniors and share in coordination and technology choices

There's much less need for mentoring in this scenario, pairing is done for fun or for efficiency- perhaps even jousting.

Working with a team of experienced seniors is a lot of fun and can be very productive, but years of successful runs on the board needs a lot of humility to offset arrogance; similarly, if everyone in the room is deliberately not putting forward their "one correct thing to do" point of view in an attempt to be reasonable, no progress is made.

Agreeing on technology choices can be tough, everyone finds very sound reasons against various technologies and lots of time ends up being spent executing comparisons or otherwise shaving the yak.

Furthermore, nobody wants to empty the bins (figuratively speaking; nobody wants to do the unexciting work).

This approach risks technical stalemate and damaged friendships.

## Build a small-medium team of senior, mid and junior developers, have a dedicated role for coordination and drive technology choices with collaboration

Now you've got developers at every stage of their career, so any problem you have no matter how mundane it may be for some developers, will be interesting and challenging for others.

You've got enough interesting problems to keep your seniors excited and because they're not burnt out with boring tasks, they're energized to pair and mentor with those less experienced.

You'll of course need to take a step back, technically- you can't really be both above the team and in the team without also coordinating the team and you've got somebody else doing that.

It's okay though you've got more free time to think about the future and evaluate technologies without being under pressure, but you can't do that on an island, you'll need to share some of the decisions with the seniors in the team (much like the earlier approach) and simply reserve some tie-breaker capacity.

The more of a dictator you are here, the less buy-in you'll get from your seniors- but again, give too much, and you'll be caught in a stalemate.

This approach has a good chance of success as long as you're mature in your understanding of using a team to attain success; you have to be willing to let go if not only the technical approach but the specific details of the product (somebody like a product owner needs to you know, own the product to be effective).

## Find a co-founder, share everything exciting, outsource everything else

I don't really have any experience with this, but it's a bit of a dream for me that I end up in a world where between me and the founder(s) is a great idea and a great capacity to deliver.

My preference would be two people, maybe three- we'd be doing it for free after hours and we'd be respective of each other's time and needs and try not make demands. The goal is to succeed but not at the cost of happiness or friendship.

We'd work really hard to not need to hire anyone for as long as possible and we'd be so absolutely sure that it was the right time before we did. I don't think poor / overly-optimistic planning is a good enough excuse to fire somebody that we ended up unable to afford.

So this means freelancing or contractors if absolutely required, but before that it would mean hard work and long hours and nobody getting paid until it was feasible.

I can't say for sure if this is a good approach, but I fall asleep every night after unsuccessfully trying to think of an idea good enough to establish such a partnership.

Against a tight enough timeline with a large enough scope, this approach could be a non-starter.

## Do it all yourself

I don't really have any experience with this either, outside of personal projects. It's basically the same as the above but without any comrades.

I think it's probably less likely to succeed than the approach above because there's nobody else with the same amount of skin in the game who'll call you out on a bad approach, make you aware that you're burning yourself out- nobody to read your pull requests.

It goes without saying, this approach is even more of a non-starter for a tight timeline with a large scope.

# Wrapping up

Well it feels like we got a long way from Lego so let's try to bring it back around.

Ultimately I'm a one-track record, the takeaway here is the same old boring eternal Ed message: it's all about that trade-off.

If you're starting from scratch to build a product and you already have some kind of timeline, your next constraint will be team you can put together for the money you have- only once you know that can you really determine how to build things.

Keep it somewhere in the middle- don't pick tech that's too exciting because it's fraught with risks, don't pick tech that's too boring because you need some way of getting good developers under your roof to solve difficult problems and it's not with Microsoft Power Apps and MSSQL.

Don't pick only juniors or only seniors, have some folks to solve the sexy problems and some folks to clean the toilets.

Don't let the code base turn to complete garbage, whether passively by not driving code quality or actively by driving too much change / feature velocity from a bad foundation; similarly, don't over-engineer clean code or abstractions or automated testing- there are lots of folks who will speak religiously and absolutely about these things, I think they're like sunshine and drinking water and eating well and doing exercise- a little goes a long way, too much is self-serving.

Focus on the value at every level- developing features, building infrastructure and tooling, delivering the product, supporting it; what's useful, what gets in the way, what's worth fixing.

Anyway, back to the Lego; this looks about how you'd expect in the final product. There's no useful rule of thumb like "always have at least two rows of holes overlapping" or "always cover a join with at least 2 rows of holes either side"- instead you have to look at how each piece gets used and how each piece interconnects, strengthen the weak parts if they break frequently or catastrophically, scavenge parts from the low-use areas, be willing to completely remove areas to reinforce others.

And sometimes, maybe just sometimes, you need to tear it all down and start again- but most of the time the next version will look surprisingly similar with some very familiar feeling failure modes.
