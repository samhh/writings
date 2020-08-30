---
slug: "less-is-more"
date: "2020-08-30"
title: "Less is more"
---

Something I've noticed this weekend is that a lot of my technological choices are tending towards simplicity. Here's a quick summary of where I'm at with things:

- Operating system: [Arch Linux](https://www.archlinux.org)
- Desktop environment / window manager: [xmonad](https://xmonad.org)
- IDE / text editor: [Neovim](https://neovim.io)
- Preferred programming language: [Haskell](https://www.haskell.org)
- Browser: [vimb](https://fanglingsu.github.io/vimb)
- Music player: [vimpc](https://github.com/boysetsfrog/vimpc) w/ [mpd](https://www.musicpd.org) backend
- Email client: [aerc](https://aerc-mail.org)
- Managed [dotfiles](https://github.com/samhh/dotfiles)

It might not be immediately obvious what about this is "simple"...

Running Arch requires some upfront effort to familiarise with Linux to a greater extent than you've likely ever done prior.

xmonad is configured in Haskell.

Neovim requires a lot more effort than something like VSCode if you want LSP integration, which as a professional software engineer I consider a must.

Haskell is a notoriously difficult language with an emphasis on correctness.

vimb - a web browser - doesn't support tabs.

My music player is split between a client and a server on the same machine.

My email client isn't a Gmail tab!

And to make all of this viable I have to track everything in a dotfiles repo.

But it is genuinely the simplest setup I've ever had, having spent long periods of time with Windows, macOS, and various flavours of Linux. In fact, I still dual boot with Windows for VR and am forced to run macOS on my work machine, so it's not as if I'm out of the loop on alternative technological lifestyles.

I think the common thread that ties these together is that _less is more_, analagous to the [Unix philosophy](https://en.wikipedia.org/wiki/Unix_philosophy). Alternative operating systems are very easy to get started with, but I think everyone has had the experience of things going so wrong that your only recourse was to reinstall the whole thing. In my experience, large updates (as opposed to [rolling release](https://en.wikipedia.org/wiki/Rolling_release)) are more likely to break things, and when they do it's much harder to debug and fix, not only because there are more changes to sift through but also because these operating systems tend to treat the user like an idiot who shouldn't be allowed to see beyond the facade of a shiny UI. They're not wrong to do so, most people don't spend as long as their computer as me; I'm certainly thankful when, say, DIY products are designed with novices like me in mind. The consequence however is that it does leave power users out in the cold, and all too often threads on Microsoft or Apple's forums about obscure issues will be met with little more than "try a reinstall".

I can see a parallel between that and my increasing fondness of Haskell, too, and to a lesser extent Rust. These languages emphasise correctness. They don't make many compromises in the name of getting new users up and running faster, and experienced users benefit long-term. This contrasts sharply with my experience with JavaScript, a language too simple for its own good. You need only look at the meteoric rise of TypeScript to see that experienced developers have been clamouring for something more reliable than "I hope this property is here like I'm expecting it to be". Even TypeScript, though, is like lipstick on a pig; compromises have been made to ensure that all valid JavaScript is valid TypeScript, and that TypeScript has no runtime presence. This results in a language which is misleadingly unsafe if you're not very careful to manage the unsafety at the edges of your application, and aren't very vigilant to the many JavaScriptian flaws permeating the type system.

It's a similar story with everything else I listed and everything I've omitted to mention. I don't want a clunky web UI or walled garden for playing music. I don't even want a native music player that manages everything for me, because that needlessly ties my playlists et al to my preferred interface, making migration a pain. And why would I want browser tabs when my window manager handles it natively far, far better?

Granted, it's not all sunshine and rainbows. It takes time to set new things up. If I wanted to install a camera into my desktop tomorrow, I'm not confident it'd work out of the box. I'd have to research it a bit ahead of time, and expect to spend maybe 15 minutes or so setting it up. But if this is the cost of, after that initial setup, everything _just working_, then that's a worthwhile time investment to me.

As for what prompted this blog post, I rewrote [samhh.com](https://samhh.com) this weekend to host my blog posts, inspired by some posts from [sircmpwn](https://drewdevault.com) on the creeping centralisation of everything on the web. During development I noticed that I'd previously set it up such that any 404 was redirected to the homepage. A simple question arose: What benefit does this provide? It takes power away from the user. They can no longer see the URL they were trying to get to, and if they wanted my homepage they could always get there with ease anyway. So why do it? I've since removed that redirect and replaced it with a simple [404 page](https://www.samhh.com/404).

(Not sponsored, just happy with it.) I feel compelled to shoutout [Gatsby](https://github.com/gatsbyjs/gatsby) here. I first set it up a couple of years ago and it's still really easy to work with for anyone who already works with React at their day job. It produces static sites that can be deployed virtually anywhere without devoting any time to devops. My site is currently hosted by Netlify, but I'm not locked in at all in terms of the project's structure, which I think tangentially ties into the above themes.

