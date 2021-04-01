---
slug: "protonmail-to-migadu"
date: "2021-04-01"
title: "Switching from ProtonMail to Migadu"
---

I haven't taken much holiday from work since lockdown began a year ago and was feeling a little burnt out, so I took a week off last week despite having nowhere to go and no plans. I had a really pleasant week; don't underestimate the power of just letting yourself relax for a little while. I completed a host of small personal tasks and got firmly back onto the intermittent fasting horse, which puts me in good stead going forward. Habits are hard to make but easy to break!

Towards the latter end of the week my curiosity in [Migadu](https://www.migadu.com) grew. I'd come across it a few weeks earlier via [Drew DeVault's blog](https://drewdevault.com/2020/06/19/Mail-service-provider-recommendations.html) and intended to revisit it in a year or two when my ProtonMail subscription/credit expired, but decided heck to the sunk cost fallacy, let's give it a try. Thus far, a few days into my trial, I'm glad I did. I strongly suspect I'll be sticking with it on the most modest plan. Here's my entire experience with it so far.

## ProtonMail

But first, what's wrong with ProtonMail? It's a good service and I'd still recommend it, but it does have flaws.

First and foremost are the proprietary protocols. The first effect of these is that you're stuck with their mobile app. It has poor long press detection, somehow, on the latest version of Android, in 2021. Much more significantly than that, because of their privacy model, you can't view the contents of emails in notifications. All you're given is something to the tune of "n unread emails". Perhaps that's tolerable, but remarkably that _unread count doesn't update if you read the emails on another device_. For it to update, you must open the mobile app. So if I've read my five unread emails on the desktop, and a new one comes through, the mobile app will incorrectly report at least six unread emails. I'm honestly a bit staggered that this hasn't been fixed yet.

The second effect of the proprietary design is on the desktop, where you'll need to install and leave running their ["bridge"](https://protonmail.com/bridge/). If that fails to work, as it did for me with [aerc](https://aerc-mail.org) due to a self-signed certificate issue, you'll need to instead use [hydroxide](https://github.com/emersion/hydroxide), a third-party, more flexible, alternative bridge. Having gone through this hassle, I found over time that this combination of aerc and hydroxide wasn't terribly stable; aerc would crash not infrequently, and plaintext emails would be incorrectly detected and rendered as HTML.

I suspect another effect of the proprietary, (pseudo-?)privacy-focused design is that performance is just awful. Be it the web client or a lean terminal client, your inbox will take several agonising seconds to load. It's likewise slow to send emails. Everything is several times slower than it should be.

ProtonMail's pricing, following a more traditional model in which you pay more more for features like custom domains, multiple addresses, and catch-all, is needlessly expensive and detached from the real underlying cost of the service. Additionally due to this approach you can't just send email from whichever address you'd like on your domain in your client without first configuring it in the web admin panel. For what it's worth, on the "Professional" plan, you can skirt the address limit by disabling any you're not using to bring you back down to under their arbitary limit, but it's mightily inconvenient.

## Migadu

For the unfamiliar, Migadu is an email hosting provider targeted, at least at the lower end, at technical users who own their own domain and really only need hosting and the email infrastructure taken care of. They have a refreshing take on cost, whereby instead of paying for accounts on your domain - which as they say themselves, are essentially just "folders" - you instead are paying for storage and bandwidth, which I'd imagine is much closer to the cost model of an email hosting provider. They also have a delightfully blunt and honest [pros/cons page](https://www.migadu.com/procon/). If only all businesses could be like this.

Because Migadu doesn't stray from standardised protocols setup is much easier. It was dead simple to start using the account in each of aerc and [FairEmail](https://email.faircode.eu). Migration to another host in the future will be a lot simpler as it's just a case of updating hostnames, usernames, and passwords. It's worth noting that there was one hickup in which I couldn't seem to send emails via `git send-email`. It turns out that `smtpencryption` has to be set to `ssl` instead of `tls` for some reason. Thanks to Kristófer Reykjalín for [blogging about the same issue](https://www.thorlaksson.com/git-send-email-not-working-on-macos/) only a few days prior.

And just like that it works. Better performance. Cheaper. Less restrictive. More practical. More standards-oriented. The only trade-off is theoretically less privacy from my hosting provider.

## pass interop

As a bonus, here's something I discovered about interopability between [pass](https://www.passwordstore.org), aerc, and git.

In aerc, you can specify account credentials dynamically with the `source-cred-cmd` and `outgoing-cred-cmd` config keys, so you can point these directly at [a shell command that accesses pass's store](https://github.com/samhh/dotfiles/commit/a5118fb1336fa5bcfc77a2144e0f1c4c3bf14f04).

In git, you can specify credential "helpers" dynamically with the help of [a ! bang, which too can be pointed at pass](https://github.com/samhh/dotfiles/commit/2a8348df40f32735265c295e28e287ea74093503). Note that the helper needs to provide the password prefixed with "password=".

