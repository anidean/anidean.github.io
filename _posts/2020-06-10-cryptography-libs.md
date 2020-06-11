---
layout: post
bigimg: /img/encryptionlibs.png
title: Cryptography Libraries for Applications
subtitle: Some Cryptography libs from the perspective of an application programmer
date: 2020-06-10
tags: [encryption, cryptography, libraries, pgp, libsodium, short]
---

In order to make a scalable end to end encrypted messaging service on a bare-bones budget I've looked into several libraries: [GPGME](https://gnupg.org/software/gpgme/index.html), [SequoiaPGP](https://sequoia-pgp.org/), and [Libsodium](https://github.com/jedisct1/libsodium).  I'm hopping off the treadmill of progress for the time being since new ones keep coming out.  Google now has [Tink](https://github.com/google/tink) which is supposed to be harder to screw up so hopefully someone else can check that out.  What these incredibly talented cryptographers and programmers are creating has allowed me, a lowly software glue technician to build some very cool stuff.

**TLDR**: I'm plugging my experimental encryption command line program that I made using libsodium, a cryptography library.  Use it as inspiration for your own projects. [Agginym Key Utility](https://github.com/anidean/agginym-cli).

When I was learning how usual end to end encrypted systems worked everything seemed in one way or another to boil down to these six actions from which a state in a distributed system can be inferred.  The distributed system I'm talking about consists of the connections between you, me, some rando across the world, our messages and the myriad of interactions between them.  The six actions are:

1. **Create** encryption and signing keys of some kind.
2. Share your keys in some way
3. **Sign** a message where you say you know about it in some way.  This creates a signature.
4. **Verify** a signature by using the signer's public key.  This tells you that the signer is aware of the message in some way.
5. **Encrypt** a message for an individual using their public key.  This creates an encrypted message.
6. **Decrypt** a message from an individual using your own private key.  This reveals the original sent message.

In order to learn how these libraries worked I made a c++ [wrapper](https://gist.github.com/anidean/aaf803fdb68a2bc22994762d74a879d2) using defaults I would want.  Then I made [another](https://gist.github.com/anidean/dc3b4dd75ae6259cfb317c2918950ca0)...then [another](https://github.com/anidean/agginym-cli/blob/master/keyutil.cpp).  This happened over several months.  Hopefully you'll be able to gain something from my pain.

What I found after doing it three times is that all of the libraries accomplish the highlighted steps above.  It doesn't really matter too much which one you use so long as you follow expert advice and use the encryption settings they say you should use.  There are lots of them and thankfully [they](https://latacora.micro.blog/) [like](https://www.schneier.com/) [to](https://security.googleblog.com/2017/02/announcing-first-sha1-collision.html) [talk](https://protonmail.com/blog/what-is-pgp-encryption/)!

To that end if you want the fastest cryptography library that takes of some hard stuff like which algorithm to use just use [Libsodium](https://github.com/jedisct1/libsodium).  In my somewhat complicated scenario of chain verification it was about 5x the speed of SequoiaPGP.  I could have been doing many things wrong but there are probably costs associated with maintaining compatibility with PGP. Libsodium has no need to do that.

It even has a bunch of extra stuff like a [Javascript library](https://github.com/jedisct1/libsodium.js/), general purpose hashing and url safe base64 binary encoding and decoding among other things.  Creature comforts, so to speak.

If you need to interact with GPG systems for some reason, use [SequoiaPGP](https://sequoia-pgp.org/).  It's 2-4 times as fast as synchronous GPGME from my testing.  Other [pretty smart people](https://twitter.com/kristamonster/status/1145633410280566785) have found performance results that point in that direction as well.  There is an [OpenPGP.js](https://github.com/openpgpjs/openpgpjs) library which is very stable and allows you to work with the web, (extension or native application only, websites are never to be trusted). Keep in mind that you'll have to deal with preventing legacy usage.

If you want to stick with GNU all the way down, use [GPGME](https://gnupg.org/software/gpgme/index.html).  That's the only reason I can find to stick with it.  You're essentially going to the shell every single time you use a library function.  It kinda slows things down.

Now that that's been said, here's part of what I've been working on using the tech above: [Agginym Key Utility](https://github.com/anidean/agginym-cli).  It supports the creation of key pairs and the signing, verifying, encrypting and decrypting of messages/files.  Give a try and if you're willing to go through the effort, tell me how it sucks or doesn't suck.  If you need utilities that work and most people are sure that they work, just use [Minisign](https://github.com/jedisct1/minisign) for signing and [age](https://github.com/FiloSottile/age) for encryption.
