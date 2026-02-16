Title: Addressing XLibre Change and GhostBSD Future
Date: 2026-02-15
Category: Update
Tags: GhostBSD, XLibre, MATE, Wayland, Xorg, Gershwin
Slug: addressing-xlibre-change-and-ghostbsd-future

## Why GhostBSD is Moving to XLibre

GhostBSD is not Linux. It's a FreeBSD-based system for the desktop. That said, we use desktop environment software from the Linux world. On FreeBSD and GhostBSD, there are things that don't exist, like systemd, which a lot of software depends on. MATE is starting to slowly get tied to it as well.

In recent years, Wayland has gained more ground. It seems everything tied to Red Hat is being forced to go with Wayland. I've been concerned about the state of Xorg not making progress. With the state of MATE and Wayland, I've been cautious and not thrilled at the idea of Wayland trying to kill X11.

There was hope at one point that Xorg would get improvements, but that hope was crushed fast. I won't get into the story. I think everyone reading this will know what happened.

When XLibre started, it first looked controversial. Before making a decision to try it, I wanted to see what would happen with it. When it started to look promising, I began to evaluate it more seriously. When the porting to FreeBSD started, we got it ported to GhostBSD by b-aaz to our ports tree. It got into GhostBSD's ports tree before FreeBSD. We tested it and made builds available, but we weren't planning to release it for 26.01.

Following the announcement that Xorg would revert changes, I wasn't sure what to do. On one hand, if I release with Xorg, we might end up with weird side effects when CVEs come our way. People who don't know me well, do not know that I dislike changes that don't seem rational to me. There's a reason why GhostBSD never moved to Gnome 3. The UI of Gnome 2 was extremely good. In my opinion, they should have continued with the same UI but improved it. MacOS has been on the same UI for a long time, and that's why their users feel at home on each release. Yes, it has improved over time, but it never drastically changed. When Gnome 3 was released, GhostBSD was in an identity crisis. We started to have multiple versions of GhostBSD. To this day, we only have XFCE left. When I discovered MATE, I switched my efforts to it. I digress, but this is to say that I always look at changes in a project carefully. Xorg reverting code just because of a dislike for someone, without considering the community using it, is a big red flag to me.

I looked at XLibre's improvements and was impressed. I started to think: if I release 26.01 with XLibre, I'll have to push the release for a month or two, but at least we should be in good shape. The change to XLibre is because GhostBSD is not ready for Wayland, and Wayland is not ready for GhostBSD. Some could say that Wayland works. Yes, it does. I've used it on Linux and it works. But GhostBSD is not Linux, and MATE is not ready for it yet. XFCE is not ready for it, and Gershwin isn't either. From what I know, GNUstep is not ready for Wayland.

In short, it's a technical decision: MATE is not ready for Wayland, and Xorg is going backward.

## The Uncertain Future of GhostBSD

Right now, the state of MATE is uncertain. We don't hear much from the devs, so I'm unsure what the future of MATE looks like, other than knowing that Wayland support has started. Also, MATE hasn't improved much in the last few years. One thing is for sure: I do like GTK 3, but I don't know how long it will be maintained. GTK 4 is fine, but GTK 5 will not support X11. That's where I see the crossroads. GTK could be forked, but I'm only one man doing most of the work on GhostBSD. I'm not alone, but I think in the vision of GhostBSD's future, I've always been alone.

Since Joe Maloney has returned to help GhostBSD and brought his project Gershwin to GhostBSD, I've been thinking about what I should do. I like GTK's versatility and vast language support. I see what Gershwin could become, but I can't make up my mind about only having Objective-C to make software. Sure, if people look at Gershwin right now, it looks too much like MacOS and incomplete. But it's a young project and themes are a work in progress. I have seen how modern GNUstep can look, and that we can make it look closer to Windows and other DE layouts.

What I'm unsure about right now is whether I should continue to replace MATE tooling or focus on helping Gershwin development. Before Gershwin came on the horizon, I was planning to replace all software settings with Setting Station. That was the first stage to start making a new DE. I was looking to do what Cinnamon did with Gnome: slowly replacing MATE with an in-house-built DE. The problem is GhostBSD is way too big, and it's hard to keep up with everything. So Gershwin could become the future, I am just trying to get over the Objective-C only part. That said swift support might get added in the near future that would help the situation a bit more.

For now, GhostBSD continues its old trajectory with Gershwin in mind, but everything is up to change.

I was asked lately about GhostBSD's longevity in a PM by a user, and I've also had other discussions with contributors about the burden of the project. One thing I can't shake is that if I were to step away from the project and give the keys to the community, I don't think GhostBSD would survive. I won't lie. There are days I'd like to move on, and other days I'd like to build something to sustain full-time development. One thing is for sure, I can't continue like this. The project is too big for me, and there are a bunch of things I dislike doing for the project. Some I have automated, some I have documented, and others I have let LLMs deal with. I'll probably make a list of what I'd like others to take over and make sure it's documented to see if the community will take care of some of those tasks.

This project started in 2009 and became real in 2010. In all those years, I've learned a lot, but the one thing I've only learned in recent times is to delegate. Though I haven't started to do that yet. It's the next step for the project to prevent it from disappearing.

I hope this helps to shine a light on where GhostBSD is at right now.