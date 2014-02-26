---
layout: post
title: "Fighting DISHFIRE: The State of Mobile, Cross-Platform, Encrypted Messaging"
---
Mobile Messaging is pretty hot right now. You could easily spend an entire afternoon installing the myriad WhatsApps and Kiks and WeChats of the world. You should already be preparing yourself to tune out the endless series of imminent TechCrunch interviews and "Show HN" posts that are inevitable when a wave of forty identical proprietary messaging apps are soft-launched at [SXSW][1] this year. And who can blame developers for jumping on the bandwagon when prominent analytics companies are effusively proclaiming that mobile usage is being ["Propelled by Messaging Apps"][2]?

But what if you care about privacy in a post-[DISHFIRE][3] world and would like to securely communicate with your friends and family? What if you have an iOS phone but some of your friends are on Android (or vice-versa)? What if you want something that works as well as standard SMS or, perish the thought, even better? Sadly, things are currently quite bleak. But there are a few options out there, and some of them are getting tantalizingly close to making your dreams come true.

The ideal client would:

*   Provide strong encryption, preferably using peer-reviewed common libraries and well-designed protocols instead of home-grown implementations
*   Be open source
*   Run on Android and iOS
*   Use the data channel instead of SMS/MMS 
    *   Avoids leaking [metadata][4] to the telecom companies who have seemingly been desperate to provide as much information as possible, on a completely unprecedented scale, to any and all three-letter agencies
    *   Much better performance (e.g. messages are sent and received more rapidly)
    *   Allows for higher fidelity images, and videos that don't make RealPlayer streams circa 1996 look like flawless 4k encodings in comparison
*   Work well in an asynchronous mobile environment and be able to handle imperfect Internet connections or situations where both clients might not be online simultaneously

It seems more than appropriate on [The Day We Fight Back][5] to take a look at the various options that are available right now:

* * *

## [ChatSecure][6]

[The Guardian Project][7] recently rebranded their Gibberbot client as ChatSecure in order to match Chris Ballinger's [iOS client of the same name][8]. The move was [not without controversy][9]. But, hey, ChatSecure uses [OTR][10] and everyone loves OTR! It's a proven solution, and as a nice side-benefit ChatSecure is able to work with all of the various desktop applications that support [XMPP][11].

The bad news is that maintaining a persistent connection to an XMPP server on a mobile device doesn't perform very well. It sort of works on Android, but frequent server disconnects, dropped messages, and reduced battery life make it a frustrating experience. It's not even possible on iOS. The iOS version of ChatSecure can only run in the background for ten minutes before it gets shut down by the operating system. This is annoying enough that the sole question in the [ChatSecure FAQ][12] is one that addresses the issue.

When it's running, ChatSecure works incredibly well; messages arrive instantly and OTR protects their contents. The Guardian Project also deserves special recognition for the ancillary tools like [KeySync][13] that they are developing alongside ChatSecure. KeySync makes it possible to transfer your desktop OTR keys and fingerprints to the Android client so that you can seamlessly communicate with existing contacts while leveraging the trust database that you've already established.

But the iOS situation makes regularly using ChatSecure completely, heartbreakingly, untenable---even if you have a very patient friend. You can try to let the iOS client initiate all of the conversations and condition yourself to reply really, really quickly. Or perhaps the Android side could send an SMS to the iOS side saying "Relaunch ChatSecure so we can talk" and then wait patiently, ready to pounce? God help you if you are both on iOS because then ten minutes becomes a *best-case* scenario.

The Android and iOS clients both fail to provide adequate warnings when a conversation is not encrypted. If you install a program called ChatSecure whose description proudly (and repeatedly) touts its built-in cryptographic prowess, security should be enforced. Bogging users down in the mundanity of manually-triggered OTR handshakes and miniscule on/off switches is the wrong approach, and it is potentially very dangerous if it leads to people speaking freely in accidental plaintext (which has almost certainly occurred). A warning message that says something like "A conversation with this user is not possible because their client does not support OTR encryption" would be dramatically better than falling back to no encryption whatsoever. Users should have to dig deep in an Advanced Settings menu and dismiss several prominent warning dialogs before they are allowed to communicate in plaintext.

If you talk to someone who is extremely tech-savvy at exactly the same time every day for around ten minutes, ChatSecure is perfect! Otherwise it's not even remotely close to a cross-platform replacement for SMS. Fortunately, The Guardian Project and Chris Ballinger appear to be aware of this and are working on a [solution that uses push notifications][14], though the last commit happened almost four months ago.

**Pros:**

*   OTR has stood the test of time and is widely considered to be very secure
*   Both clients are open source and on GitHub
*   Available today
*   The Android version is capable of working with The Guardian Project's [Orbot][15] client to route traffic over the [Tor][16] network for greater anonymity
*   Self-hosted XMPP servers are fully supported

**Cons:**

*   Apple doesn't allow background processes, and ChatSecure does not support push notifications. The iOS version is therefore almost useless.
*   The clients share a name but do not share the same functionality. The Android version, for instance, offers encrypted picture messaging and voice attachments. The iOS version does not. The Android version lets you import your OTR identity with KeySync. The iOS version does not. The rebranding effort probably should have waited until the two clients worked together under realistic circumstances and when they were closer to feature parity.
*   The future of XMPP is looking bleak, particularly now that Google---one of its former champions---has abandoned the standard in favor of their proprietary Hangouts format
*   XMPP does not work well in an asynchronous mobile environment
*   It's way too easy to end up having a plaintext conversation, especially when communicating with desktop users. This shouldn't be possible, and the developers should feel ashamed that they even allow this to happen. It's a crazy default to not require encryption for all conversations and it should be changed immediately. Hopefully no one has gotten hurt.

**ChatSecure's [Response][17]**

> Nice summary! We have some new plans for solving push on iOS. I'm sorry we haven't finished it sooner.

This is excellent news. No apology is necessary!

**The Guardian Project's Response** (Edited to condense replies that [were][18] [spread][19] [across][20] [multiple][21] [Tweets][22])

> Many of the limitations you pointed out are based on support for XMPP interop, which we are committed to. All the other systems you reviewed (save for Cryptocat), roll their own servers and protocols, which is great, but different. Also, supporting Tor/anonymity means push is not really viable for all. Ultimately, we have made life hard for ourselves!
> 
> Also, it was a conscious choice not to require OTR always, and not something we are ashamed of by any means. Along the same lines, TextSecure using SMS is a bad choice from a privacy perspective, but logical for practical reasons.

These are great points, and I have taken them into account by adding two new bullets in the Pros category for ChatSecure. I still think it is worth asking whether or not the convenience of allowing plaintext conversations to happen by default is worth the potential downsides. I remain convinced that the answer in this case is "no" but I can see how the argument is more nuanced than I originally thought.

It's only possible on Android, but ChatSecure's ability to combine OTR with a self-hosted server that can be anonymously accessed via Tor is damn impressive. I have an enormous amount of respect for the ChatSecure team and for The Guardian Project as an organization. Their feedback is sincerely appreciated and I am eager to see what else they have in store.

* * *

## [Cryptocat][23]

Cryptocat has gone from an upstart that people viewed with bemused skepticism (and sometimes outright disdain) to a relatively well-respected open source project that has an admirable record of transparency. There have been some extremely serious vulnerabilities in the past, but they have always been addressed quickly. The project has recently been placing a larger emphasis on security than ever before, even going so far as to commission several outside audits of its codebase by paid consultants. The most [recent analyses][24] were conducted by [Least Authority][25] and [iSEC Partners][26].

Cryptocat's efforts to prioritize usability and simplicity, thereby making encrypted messaging more accessible to mere mortals, is an example that more projects should follow. In contrast with ChatSecure, for instance, it's not possible to accidentally have a plaintext chat using Cryptocat. Encryption is an implementation detail that is largely obscured from users. This is a good thing.

The Android and iOS versions were [announced on December 5th][27] and the source code for the projects is available for review on [GitHub][28]. A public release of both clients should happen soon. Unfortunately it looks like Cryptocat is going to suffer from many of the same issues as ChatSecure. A cursory review of the source code does not reveal any push notification support at the moment, for either platform, which likely means that users will need to have the application open in order to communicate or else they will run into the dreaded 10-minute time limit on iOS. This is a huge step backwards compared to SMS in terms of usability.

Normal users always choose convenience over security and Cryptocat Mobile is unlikely to win any new converts. But it will significantly improve the lives of existing users. Even if it may not immediately be the secure messaging replacement that we all want right now, the world will be better off with a mobile Cryptocat. I look forward to seeing how it develops over the coming months.

**Pros:**

*   Another solid, cross-platform, OTR-based contender is a good thing
*   Uses sane defaults (chats are always encrypted)
*   Open source

**Cons:**

*   Not yet available
*   The iOS client doesn't support APN push notifications, and the Android client doesn't support GCM push notifications
*   Does not support photos, videos, or voice messages

* * *

## [Heml.is][29]

Heml.is made a big splash last July when their crowdsourcing initiative successfully reached its lofty $100,000 funding goal in only [36 hours][30]. The timing was perfect, coming right on the heels of the initial [PRISM][31] outrage. Oh and when you are pursuing an "us against the MAN" narrative, it really helps if your project is the brainchild of Peter Sunde, the co-founder of [The Pirate Bay][32].

The Heml.is team haven't revealed enough details for me or anyone else to have much of an opinion yet, but I was wildly unimpressed by the passive-aggressive tone in [Hemlis update #3][33] that was posted on August 28, 2013. Is the Heml.is team really so worried about their feelings getting hurt that they have chosen to proceed blindly and begin implementing their protocol without first verifying its design? Meanwhile, cryptographers and security researchers are worried about properly structuring systems so that no one gets tortured and killed. Step it up, Heml.is, or get out of the game. This quote is just pathetic:

> Most questions about heml.is is about the encryption we're going to use. How it's going to work and details about it. For different reasons, we've stayed away from talking too much about the details. It's not because we're arrogant, it's just that dealing with the crypto community is really time consuming. Whatever solution we've decided on would be criticized and we aren't really interested in the flame war that's inevitable. We'd rather create and get things going. Maybe a small lesson for the crypto geeks out there would be to be supportive instead of negative.

There aren't enough sighs in the world.

But [now they are claiming][34] that their implementation will use [libsodium][35] (which is based on the highly-regarded [NaCl library][36] from Daniel J. Bernstein, Tanja Lange, and Peter Schwabe). They are also claiming that the clients [will be open source][37]. Perhaps there is hope after all? There's no set release date, but it's probably [only a few months away][38]. I am *so* skeptical that this will become my messaging platform of choice, but perhaps it won't be the unmitigated disaster that it was looking like last summer.

**Pros and Cons:**

*   To Be Determined. So far they haven't released protocol documentation or any information at all really. The design of the application looks quite nice, however.

* * *

## [Silent Circle][39]

The Silent Circle [Founders and Leadership][40] page is impressive enough that you almost have to forgive these titans of cryptography for the "Created by Navy SEALs and Silicon Valley cryptography experts" tagline. It's kitschy and that seems like the wrong order to me, because off the top of my head I don't know what Navy SEALs have to do with cryptography or digital security. Maybe Silent Circle's data centers have been thoroughly fortified against rival amphibious assault teams? Maybe their logo is still visible when you are wearing night vision goggles? In all seriousness, over-the-top marketing aside, the Silent Circle suite of products is basically the only available cross-platform option at this point that covers both voice and text communications.

Silent Text, their messaging solution, is impressively calculated in its use of rhetoric. Everything about the application is designed to make you feel like the main character in a Tom Clancy novel. You don't delete a message, you issue a "Burn Notice," and instead of verifying fingerprints that are comprised of stodgy old hexadecimal groups you do it using phrases like "Six Romeo India Five." I think the Android version is pretty ugly overall, like a real-life manifestation of an early 90s CGI sequence, but it gets the job done.

The biggest problem that I have with the package is that Silent Phone and Silent Text both require an active Silent Circle subscription. At $9.95 per month (or $99.95 annually) the prices aren't unreasonable if you are a business, but there's essentially no chance that you'll be able to convince anyone you know that they should spend more than they do on Netflix for an esoteric messaging application that almost no one uses. Silent Phone and Silent Text are also both closed source. Silent Circle appears to be attempting to do some sort of ["open core" strategy][41] but this ends up being more confusing than comforting. For example, their Silent Text [repo][42] was last updated on March 26, 2013 and refers to version 1.5.0. But the most recent version in the Google Play Store came out on December 17th, 2013 and is version 1.2.0. Are they decrementing version numbers as they move forward, and why is this repo so damn musty and nearly a year removed from its last commit?

**Pros:**

*   Complete solution, including encrypted phone calls and messaging
*   Perfect Forward Secrecy and the [trendiest non-NIST ciphers and elliptic curves][43]
*   The founder of the company, [Phil Zimmerman][44], invented [PGP][45] and [ZRTP][46]. It's unlikely that we are dealing with a team of amateurs.

**Cons:**

*   Closed source (mostly)
*   $9.95 a month probably makes this an instant pass for everyone in your life

* * *

## [Surespot][47]

Surespot has by far been my favorite encrypted messaging application on Android. I have had fewer problems with it than with anything else that I've tried, and it has also universally been the most positively received option among my friends. For the past month or so I have been using it with one friend in particular who is a beta tester for the iOS version and it has performed wonderfully. We might only be a few days away from an open source, cross-platform messaging solution that actually works. The iOS version was [submitted to the App Store on January 28th][48] and [rejected for a seemingly silly reason][49] ten days later. It has since been resubmitted, and it shouldn't be long now.

**Update:** Surespot for iOS is [now available][50].

Surespot is extremely fast, simple, and well-designed. The interface is spartan, yet playful. Messages are sent and received instantly using Socket IO when the app is open. Apple's APN and Google's GCM push notification frameworks deliver notifications to iOS and Android devices respectively. It's free, and there are no ads.

There is a voice messaging feature that is very useful and also a lot of fun. Unlocking the ability to send voice messages requires a $2 in-app purchase, but anyone can listen to them. Users are also able to donate money to the project directly and the Surespot documentation promises that a portion of the proceeds will be shared with the [EFF][51]. The in-app voice purchase and donation strategy is a clever way to subsidize the development of the application and so far it seems to be working well. The 2 million message threshold was announced on [November 19th][52] and took more than six months to reach. Just over two months later on January 27th, Surespot had delivered [over 4 million messages][53]. It looks like it is picking up momentum, and the iOS release will certainly help.

Perhaps the biggest compliment that I can pay to Surespot is that it's easy to articulate to friends why they should install it. It offers clear advantages over SMS in terms of speed; image quality; support for long messages that won't be split into pieces first and frequently received in the wrong order; and voice messaging. The fact that the messages are encrypted can be a big deal to you even if it's not on your friend's radar.

**Pros:**

*   Open source
*   Elegant and simple design
*   Did I mention that it's fast? It is.
*   Supports picture and audio messages
*   Native GCM and APN push support means that it works well in an asynchronous mobile environment and does not depend on a persistent connection to a server

**Cons:**

*   Only 1,000 messages are stored at a time. Old messages will fall away chronologically to make room for new messages. Some people might look at this as a security-enhancing feature, and it's always possible to copy the text of important messages, but personally I wish that I could save my conversation history.
*   No video support
*   No group messaging
*   No Perfect Forward Secrecy 
    *   New keys can be generated at any time, but there's no automatic per-message ratcheting like there is in OTR, Silent Circle's [SCIMP protocol][54], or in systems that use the new [Axolotl Ratchet][55]
*   Potential MITM issues 
    *   Key fingerprints can be verified manually, but the server is in a position where it can send new public keys for an arbitrary contact that will then be used by other clients. The issue is documented on Surespot's [Data and Threat Analysis page][56], and there's an open GitHub [ticket][57] to fix it. In the meantime, periodically reviewing fingerprints is easy to do for the paranoid among us.

* * *

## [Telegram][58]

Smarter people than me [have][94] [analyzed][59] [Telegram][60] [to][61] [death][62]. I have very little to add. The consensus is that the [MTProto][63] protocol is not all that great and is based on puzzlingly obscure techniques and ancient cryptographic primitives like [SHA-1][64] that are eagerly being deprecated by the rest of the world. In the interest of fairness, Telegram's response to all of this can be found [here][65].

Telegram is growing like crazy and recently has been racking up [more than 200,000 new users every day in Spain alone][66]. I was really excited when I first heard about it. I mean, here was an encrypted messaging application that was available for iOS and Android, and it was open source! People are actually using it! But cryptography is hard, and they seem to have done a poor job of it. I'm staying away for now.

**Pros:**

*   Very popular
*   Open source
*   Looks exactly like WhatsApp

**Cons:**

*   Everyone that I respect hates it
*   Secret Chats are not the default, and default chats are not end-to-end encrypted
*   No Perfect Forward Secrecy at the moment, though they [claim their protocol can support it with client modifications][67]
*   Looks *exactly* like WhatsApp

* * *

## [TextSecure][68]

If I could pick a single client and magically will it into immediate cross-platform existence, this is the one. Moxie Marlinspike and the rest of the fine individuals at Open Whisper Systems have somehow improved upon OTR's design in every possible way and managed to do so while solving all of the problems that would typically prevent it from working well in a mobile environment. TextSecure's [V2 protocol][69], lovingly detailed in a series [of][70] [blog][71] [posts][72], represents the current state-of-the-art in encrypted messaging.

So far the TextSecure V2 protocol has only been rolled out to [Cyanogenmod][73] users as part of its [WhisperPush functionality][74]. This is a major accomplishment in its own right because it means that approximately 10 million people will automatically get the benefits of encrypted messaging without any effort on their part. TextSecure is available on Android, of course, and has been for years, but the current version in the Play Store is still using the old protocol. The release of the new Android version that incorporates a [brand-new design][75], push-based data channel messaging (instead of SMS), the [Axolotl ratchet][55], prekeys, Curve25519, and all of the other massive improvements [looks to be imminent][76]. I can't wait.

**Update:** TextSecure version 2.0 is [now available][92] for Android!

The current Android version is tough to recommend to everyday users because the MMS support is so poor. To be clear, I place all of the blame for this on the carriers who developed such an awful standard in the first place. When I try to send a picture to an AT&T user they receive only an obscure message size failure warning even though Verizon, Sprint, and T-mobile subscribers can all receive them just fine. TextSecure has no support for group messaging either, so MMS messages that are sent to multiple people don't get threaded properly. You used to be able to work around these issues by telling TextSecure not to handle MMS and using the standard messaging application for pictures, videos, and group messages, but this is no longer possible in Android 4.4 KitKat. More than half of the people I convinced to install TextSecure ended up uninstalling it due to the MMS and group messaging issues. The new version will solve this by skipping the SMS/MMS transport layer entirely.

**Update:** TextSecure version 2.0 fixes most of these issues. Using it as a standalone application that only processes messages from other TextSecure users is once again possible. MMS group messages also work perfectly. Sending images via MMS to AT&T users is still broken for me.

OK, great, it's already decent on Android if you can deal with some hiccups and is about to get even better. What about iOS? The good news is that it's [already in progress][77]. The bad news is that it's probably going to be a while. At Whisper System's recent Spring Break of Code event, Christine Corbett, one of the lead developers of TextSecure for iOS, announced plans to merge TextSecure and RedPhone (Whisper System's open source encrypted telephony program) into a [single unified application][78]. This is a great move because it means that users won't be fragmented across two different mediums. If someone has Whisper installed you will be able to use your choice of text or voice to communicate with them. Sometime soon, Whisper for iOS will come out with voice support. Text support will come later.

How much later? That's a good question. My hope is that the RedPhone for iOS team members (who are currently toiling away in a skunkworks non-GitHub repo) will join forces as soon as possible with [Christine][79] and [Frederic][80] to help them finish the TextSecure functionality so that it can be merged into the nascent iOS application shortly after it is released. Not having to duplicate efforts on contact list management and other shared UI areas should prove to be helpful. At the moment, TextSecure's V2 protocol is [still being fleshed out on iOS][81] and there are several other components that have not been implemented yet. Major progress has been made, but there is still a lot of work to be done. If I had to guess (and I should stress that this is just a hunch) I would say that it's probably going to be sometime this summer at the earliest before Whisper for iOS supports text messaging functionality.

**Update:** I'm extremely happy to report that I might be wrong about this. The rate of commits has picked up significantly over the past few days, and Christine Corbett now says that they are ["racing the RedPhone iOS team"](https://twitter.com/corbett/status/438037900975169536) instead of waiting for them.

When Whisper is released, Silent Circle will finally have a cross-platform competitor that supports voice and text. Unlike Silent Circle, it will be open source (instead of "kinda") and supported by community donations (instead of mandatory $9.95 subscriptions). Moxie et al. are in their own league entirely, and they have big plans. Whisper will be worth the wait.

**Pros:**

*   The very best protocol design, implemented by people who know what the hell they are doing
*   A very large userbase, thanks to the Cyanogenmod integration

**Cons:**

*   <del>Encrypted SMS/MMS messages leak metadata to the telcos</del> This transport layer is now optional in v2. Messages are sent over the data channel by default.
*   <del>MMS is horrible, and TextSecure has some issues dealing with it</del> Most of these problems have been solved, but sending pictures to non-TextSecure users is still hit-and-miss
*   <del>No group messaging</del> v2 supports encrypted group messaging and it works very well
*   The anticipation for an iOS release is killing me

* * *

## [Threema][82]

Threema hails from the country that brought us no-questions-asked banking and all-purpose knives with scissors and corkscrews, Switzerland! All of its servers are located there for instant marketing cachet, as though any text-based secrets stored therein will be held as sacrosanct as the One Percent's tax havens. But then they go ahead and make that analogy fall apart because Threema is really easy to use---a messaging application for the 99%, if you will.

It does a lot of things right: Threema is using the excellent [NaCL Cryptography Library][36]; the onboarding process for new users is very friendly; it looks great on both iOS and Android; it uses push notifications; and the [FAQ][83] does an excellent job of explaining technical concepts in an accessible way. Threema also prominently features three dots (which are the application's namesake) next to every contact. These dots represent that user's verification level. One red dot means no verification, two yellow dots mean that the server has verified a user's identity, and three green dots mean that you have personally scanned and verified that user's key and ID. This is a smart way to motivate users to practice strong security and avoid MITM attacks.

So far, so good! Except, Threema is closed source. That makes it significantly harder to trust and wipes out many of these benefits. Are they really doing what they claim to be doing? Did they make any mistakes? Does the Swiss NSA (if there even is such a thing) have a secret backdoor?

Threema has introduced a unique [validation feature][84] in an attempt to assuage these concerns. The idea is that you can tell the application to log its encrypted payload, and then you can use some open source tools that they provide in order to independently verify that the payload has been encrypted properly using NaCl. This is great, and it's dramatically better than nothing, but as with any closed source application it's impossible to fully audit. Perhaps the program logs one strong payload but delivers another that is significantly weaker? Maybe it only does this occasionally (and only when it is on a non-wifi network) to make it more difficult to detect? You can't really be sure, and that's a problem when your target demographic is paranoid.

Like skeptical parents who are on the verge of letting you go to the cool sleepover, Threema seems to really want to do the right thing. They can keep their server proprietary because a proper encrypted messenger never trusts the server in the first place, but the clients should be open source. People will still pay for the clients because they are useless without the servers, and access to the servers can be controlled. Come on, Threema! Make it official and you can help continue the proud open source tradition of being a [great program with an awkward name][85].

**Pros:**

*   <del datetime="2014-02-17T05:14:09+00:00">By far the best cross-platform option that is available right now, and I have tried them all</del> Threema remains an excellent choice, but Surespot takes the crown now that it is available for iOS (and fully open source)
*   The best option for cross-platform encrypted group messaging
*   Good documentation
*   Extremely easy to set up and use

**Cons:**

*   Closed source
*   No Perfect Foward Secrecy in the protocol, though they [claim to have implemented it at the transport layer][83]

* * *

# Just Tell Me What To Do

## Today

**Android to Android:**

*   Use [TextSecure version 2.0][93]

**iOS to Android (or vice-versa), iOS to iOS:**

*   Use [Surespot][47]
*   If you need encrypted group messaging, use [Threema][82]

## The Hopefully Not Too Distant Future

**Android and iOS:**

*   Use Whisper as soon as it comes out. Encrypted phone calls and text messaging bundled together in a unified interface that will be available on both iOS and Android? Hell yes.

* * *

# Final Thoughts

That's the state of things at this point in time. Perhaps some of you may be wondering where [myEnigma][89], [Redact][90], [TigerText][86], [Wickr][87], and various other options are in the above comparison. Well, if they can't be bothered to release *any* source code, fail to provide even basic protocol documentation, and have not posted a threat model analysis, then they are not worthy of your time or your attention. BBM should also be [avoided at all costs][91].

If there is something that I have missed, or if you notice any mistakes, please [let me know][88] and I will take care of it immediately. If the developers of any of the above applications would like to respond in detail to any of the points that were raised, I will happily include such comments alongside my writeup.

My sincere thanks to all of the organizations that are fighting to protect our messages from dragnet surveillance. I am optimistic that we will soon have several viable choices for securing the thoughts and hopes and dreams that we share with all of the people who are close to us---a list that does not include the NSA or GCHQ.

SMS is dead, long live privacy.

 [1]: http://sxsw.com/
 [2]: http://blog.flurry.com/bid/103601/Mobile-Use-Grows-115-in-2013-Propelled-by-Messaging-Apps
 [3]: http://en.wikipedia.org/wiki/Dishfire
 [4]: https://www.eff.org/deeplinks/2013/06/why-metadata-matters
 [5]: https://thedaywefightback.org/
 [6]: https://chatsecure.org/
 [7]: https://guardianproject.info/
 [8]: https://github.com/chrisballinger/Off-the-Record-iOS
 [9]: https://twitter.com/moxie/status/390501258294882305
 [10]: https://otr.cypherpunks.ca/
 [11]: http://en.wikipedia.org/wiki/Xmpp
 [12]: https://chatsecure.org/faq/
 [13]: https://guardianproject.info/apps/keysync/
 [14]: https://github.com/ChatSecure/ChatSecure-Push-Server
 [15]: https://guardianproject.info/apps/orbot/
 [16]: https://www.torproject.org/
 [17]: https://twitter.com/ChatSecure/status/433346160112062464
 [18]: https://twitter.com/guardianproject/status/433364638525186048
 [19]: https://twitter.com/guardianproject/status/433364914808180736
 [20]: https://twitter.com/guardianproject/status/433365166021824514
 [21]: https://twitter.com/guardianproject/status/433371922404245504
 [22]: https://twitter.com/guardianproject/status/433372178248400896
 [23]: https://crypto.cat/
 [24]: https://twitter.com/kaepora/status/432991985968771072
 [25]: https://leastauthority.com/
 [26]: https://www.isecpartners.com/
 [27]: https://blog.crypto.cat/2013/12/cryptocat-for-iphone-call-for-review/
 [28]: https://github.com/cryptocat
 [29]: https://heml.is/
 [30]: http://hemlismessenger.wordpress.com/2013/07/11/funded-100-in-36-hours/
 [31]: http://en.wikipedia.org/wiki/PRISM
 [32]: https://thepiratebay.se/
 [33]: http://hemlismessenger.wordpress.com/2013/08/28/hemlis-update-3/
 [34]: http://hemlismessenger.wordpress.com/2014/01/17/hemlis-update-5-new-year-new-progress/
 [35]: https://github.com/jedisct1/libsodium
 [36]: http://nacl.cr.yp.to/
 [37]: https://twitter.com/HemlisMessenger/status/425250256293732352
 [38]: https://twitter.com/HemlisMessenger/status/420939276055244801
 [39]: https://silentcircle.com/
 [40]: https://silentcircle.com/web/founders-leadership/
 [41]: https://github.com/SilentCircle
 [42]: https://github.com/SilentCircle/silent-text
 [43]: https://silentcircle.wordpress.com/2013/09/30/nncs/
 [44]: http://en.wikipedia.org/wiki/Phil_Zimmermann
 [45]: http://en.wikipedia.org/wiki/Pretty_Good_Privacy
 [46]: http://en.wikipedia.org/wiki/ZRTP
 [47]: https://www.surespot.me/
 [48]: https://twitter.com/adam2fours/status/428231046694322176
 [49]: https://twitter.com/adam2fours/status/432575358534049792
 [50]: https://itunes.apple.com/us/app/surespot/id790314865
 [51]: https://www.eff.org/
 [52]: https://twitter.com/surespot/status/402918501909688320
 [53]: https://twitter.com/surespot/status/427999718396796928
 [54]: https://silentcircle.com/web/scimp-protocol/
 [55]: https://github.com/trevp/axolotl/wiki
 [56]: https://www.surespot.me/documents/threat.html
 [57]: https://github.com/surespot/android/issues/8
 [58]: https://telegram.org/
 [59]: https://news.ycombinator.com/item?id=6913456
 [60]: http://unhandledexpression.com/2013/12/17/telegram-stand-back-we-know-maths/
 [61]: http://thoughtcrime.org/blog/telegram-crypto-challenge/
 [62]: http://translate.google.com/translate?hl=en&sl=ru&u=http://habrahabr.ru/post/206900/
 [63]: https://core.telegram.org/mtproto
 [64]: http://en.wikipedia.org/wiki/SHA1
 [65]: https://core.telegram.org/techfaq
 [66]: https://twitter.com/telegram/status/431035097119088640
 [67]: https://core.telegram.org/techfaq#q-do-you-have-forward-secrecy
 [68]: https://whispersystems.org/
 [69]: https://github.com/WhisperSystems/TextSecure/wiki/ProtocolV2
 [70]: https://whispersystems.org/blog/simplifying-otr-deniability
 [71]: https://whispersystems.org/blog/asynchronous-security
 [72]: https://whispersystems.org/blog/advanced-ratcheting
 [73]: http://www.cyanogenmod.org/
 [74]: https://whispersystems.org/blog/cyanogen-integration/
 [75]: https://whispersystems.org/blog/roosters-and-a-mountain-of-design/
 [76]: https://github.com/WhisperSystems/TextSecure
 [77]: https://github.com/WhisperSystems/TextSecure-iOS
 [78]: https://whispersystems.org/blog/a-whisper/
 [79]: https://github.com/corbett
 [80]: https://github.com/FredericJacobs
 [81]: https://github.com/WhisperSystems/TextSecure-iOS/tree/cryptopipeline
 [82]: https://threema.ch/en/
 [83]: https://threema.ch/en/faq.html
 [84]: https://threema.ch/validation/
 [85]: http://www.gimp.org/
 [86]: http://www.tigertext.com/
 [87]: https://www.mywickr.com/en/index.php
 [88]: mailto:josh@joshlund.com
 [89]: https://www.myenigma.com/
 [90]: http://www.redactapp.com/
 [91]: /2014/02/avoid-bbm/
 [92]: https://whispersystems.org/blog/the-new-textsecure/
 [93]: https://play.google.com/store/apps/details?id=org.thoughtcrime.securesms
 [94]: http://www.cryptofails.com/post/70546720222/telegrams-cryptanalysis-contest
