---
layout: post
title: "The confusing state of RCS"
subtitle: "As of late 2024"
author: Henry
category: telco
tags:
  - SMS
  - RCS
  - Google
  - Apple
  - iMessage
  - Universal Profile
  - Jibe
  - WhatsApp
---

As of late 2024

## Disclaimer

While I'm employed by a mobile network operator, I'm not involved in this space at all. I can't speak of the relationship between carriers, Google, and Apple. This article reflects my experience as an end user, based on research and speculation from publicly available information.

## History

Here's a short and biased summary of RCS's milestones.
Feel free to skip to the next section if you just care about today's situation.

### 2007 - 2008: early days

Coordinated work on the RCS specification starts, first among a few industry players, then under the GSMA.

### 2009 - 2010: over the top instant messaging services

The rise of *over the top* instant messaging is now unstoppable in most of the world. It's abundantly clear to anyone paying attention, including carriers and other GSMA members, that the idea of a new standard to replace individually billed SMS and MMS is moot to a large extent.

### 2011: iMessage

While a bit late to the party, Apple’s answer quickly wins the US market due to several factors: dominant market position, seamless integration with the default text message application and fallback to SMS/MMS not being a problem, unlike in other parts of the world.

### 2011 - 2015: Google’s answer, or lack thereof

As someone who was using Google Talk’s video chat on 3G roughly one year before FaceTime would allow the same, it’s hard to overstate how messy Google’s strategy was during that time, let alone count the number of missed opportunities, for instance letting Facebook acquire WhatsApp.

### 2015 - 2016: Jibe and Universal Profile

Until then, RCS had only materialized as small scale and mostly local experiments, labeled Joyn or RCS-e by a few operators.

Eventually, Google decides to back the RCS horse more seriously. In 2015 the company acquires Jibe, one of the major RCS solution providers. The RCS standard is revamped by the GSMA and the first version of *Universal Profile* gets published in 2016.

Google's strategy becomes clearer: provide a turnkey, SaaS solution to carriers willing to run an RCS backend, as well as a matching RCS client, Google Messages.

### 2017 - 2018: some real adoption

A few carriers jump on the Jibe ship.[^1] If you are on Google Messages, an Android trick can enable the feature regardless of carrier support, connecting to Jibe's default sandbox endpoint. Meanwhile, Google Allo's short lived existence influences Messages rich chat features.

In practice, RCS is still fragmented and feels like a ghost town.

###  2019: RCS for everyone, and the Cross-Carrier Messaging Initiative

In 2019, two things happen. First, Google enables RCS for anyone using Messages, with or without carrier support.[^2]

Secondly, American MNOs publicly communicate about an initiative that should bring interconnection and interoperability between the different RCS backends and clients.

### 2020 - 2021: RCS's centralization

Despite many carriers and several solution providers working on the deployment and interconnection of RCS backends in both the US and Europe, users regularly complain about reliability and interoperability issues. Some third party RCS backends are interconnected with Jibe hubs, some are not. The CCMI is abandoned.[^3]

On the client side, things aren't much better. There are very few alternatives to Google Messages, namely Samsung's own messaging application, and a few carrier branded clients, like Verizon Message+.

I speculate here, but this finally pushes Google to pick a more agressive strategy, effectively strong-arming carriers who aren't using Jibe and Google Messages. Big ones agree to the switch, small ones are left out with no prospect of interconnected RCS.

### 2022: E2EE, *Get The Message* campaign

Google deploys end-to-end encryption in Messages as an add-on, i.e. outside the base Universal Profile protocol, taking advantage of the User Compatibility Exchange allowed by the specification, furthering fragmentation with recipients who are still using other clients.

Google also launches the marketing campaign *\#GetTheMessage*, putting pressure on Apple, still not supporting RCS nor having shown any intention to do so.

###  2023: RCS is finally coming to iOS

In a complete turn of events, Apple announces that RCS will be introduced next year but that it won't support Google's ad-hoc E2EE; Apple would rather work with the GSMA and add E2EE to a future version of the specification. This brings questions, as the iOS client will surely need to work on Jibe, now being almost ubiquitous worldwide.

By the end of 2023, RCS has reached 930 million active users and is on track to surpass one billion in 2024.

## Current situation

This brings us to 2024.

### Android

Google's questionable change of strategy paid off. The last major carriers that weren't on Jibe have migrated and are shutting down their previous RCS solutions. Even Samsung is officially abandoning its own RCS client for Google Messages.[^4]

I can't help but feel RCS is almost indistinguishable from an OTT service at this point, controlled almost end-to-end by Google, who is driving the specification evolution, and both the only relevant server implementation and Android client.

I don't think there's a huge demand for custom text messaging clients on Android, but things are different on the server side. Carriers are typically able to choose solutions from multiple vendors. For instance, Google's stance had pretty dire consequences on Mavenir's RCS business.[^5]

And one question remains: how much does this cost carriers, and what will Google do to the ones that aren't willing to pay?

### iOS

On the iOS side, things are a bit more complex. Right now the RCS client is compatible with Universal Profile version 2.4, which is 5 years old. As expected, there's no E2EE. While there are a few rough edges, it seems to work reasonably well with Jibe and Android users though.

### Availability

On a more annoying note, RCS is still pretty far from being enabled worldwide. Like with other IMS configuration, the feature needs to be set up in iOS carrier bundles.

Due to unclear reasons, this is still not the case in many countries.[^6]
Even in the US, some MVNO subscribers can't use RCS yet.[^7]

It isn't the first time Apple selectively delays standard IMS features. It's a bit hard to believe carriers are at fault here, especially the ones that have been running their own Jibe deployment for years and have publicly communicated about the upcoming iOS support.[^8]

### Apple's motives

Why did Apple make a U-turn on RCS in the first place?

I don't buy the Chinese theory.[^9] It's true that Chinese MNOs wanted RCS compatibility, but Apple could have added it as a China specific feature, buried under its local name: *5G messaging*. After all, China's RCS network is an island, it will probably never be interconnected with the rest of the world. Case in point, RCS was first enabled for US carriers running Jibe during the iOS 18 beta, and came to China only later, with the 18.1 update.

Funnily, iMessage isn't popular enough in the EU to be considered a gatekeeper under the Digital Markets Act. Still, Apple might want to avoid pushing its luck there, and at the same time divert attention from the US DoJ.[^10]

Apple might also be feeling WhatsApp's threat on its dominance in the US market, as Meta's service surpassed 100 million activer users.

This is again pure speculation on my end, but adopting RCS seems like a fairly cheap way to keep legislators happy, discourage users from installing a third party application, all while keeping the blue bubbles and their network effect.

### Future evolutions
While there's no public roadmap, Google intends to add the IETF end-to-end encryption standard MLS to Android. This could indicate a move towards interoperability capabilities in the near-term future.[^11] Assuming players like Meta make a similar decision under the EU's DMA, this could really be a game changer from the user's perspective, and I hope we'll see that materialize.

This might also explain why Apple didn't bother implementing the current Signal protocol used by Google.

### Update (2024-11-25)

I completely forgot to mention another consequence of Google’s policy: there’s no RCS client for AOSP forks and alternative mobile OS such as HarmonyOS. Even though Huawei’s RCS client works in China, Google won’t allow it on Jibe.

On the iOS front, more European carriers have announced the feature would be enabled soon or early next year. The roll-out still looks confusing though, if I take France as example again, both Bouygues and Free have it enabled in the current iOS beta, while Orange communicated about RCS availability coming during the first half of 2025.

[^1]: https://www.wired.com/2017/02/google-support-for-rcs/

[^2]: https://www.theverge.com/2019/6/17/18681573/google-rcs-chat-android-texting-carriers-imessage-encryption

[^3]: https://arstechnica.com/gadgets/2021/04/verizon-att-and-t-mobile-kill-their-cross-carrier-rcs-messaging-plans/

[^4]: https://www.androidauthority.com/samsung-drop-samsung-messages-google-messages-reason-3463520/

[^5]: https://www.fierce-network.com/wireless/mavenir-executes-some-layoffs-related-rcs-disappointment

[^6]: https://foxt.dev/ios-rcs/

[^7]: https://9to5google.com/2024/09/16/google-fi-rcs-iphone/

[^8]: https://hellofuture.orange.com/en/rcs-rich-interoperable-messaging-apple-on-board/

[^9]: https://daringfireball.net/2024/02/eu_rcs_imessage

[^10]: https://www.wired.com/story/apple-doj-antitrust-imessage-encryption/

[^11]: https://www.androidauthority.com/google-mls-e2ee-messages-3457483/
