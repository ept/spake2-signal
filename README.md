Signal protocol + SPAKE2
========================

The [Signal protocol](https://www.signal.org/docs/) is currently one of the best
cryptographic protocols for secure communication between two users, providing strong
security properties such as [forward secrecy](https://en.wikipedia.org/wiki/Forward_secrecy)
and [post-compromise security](https://eprint.iacr.org/2016/221.pdf).

However, a weakness of the protocol is that in order to establish a secure session
with another user, you need to know their public key. How do you get that public key
if all you have is their phone number? In Signal's case, there is essentially a
trusted database server. You query the server with the phone number of your contact,
and you get back their public key. You have to trust that the server gives you the
correct key. If the server instead gives you the NSA's public key, would you ever
notice? (You would notice if you compare security numbers, but honestly, who ever
does that? Probably only the kind of person who used to attend
[PGP key signing parties](https://en.wikipedia.org/wiki/Key_signing_party).)

This repository is a proof-of-concept of using the Signal protocol without a
trusted key server. The idea is fairly simple: we use
[SPAKE2](https://tools.ietf.org/id/draft-irtf-cfrg-spake2-10.html), a
password-authenticated key agreement protocol, to estabish a shared secret, and then
the users exchange the key material necessary to set up the Signal channel,
encrypted and authenticated using that secret. This protocol has the following
properties:

* The users need to establish a shared one-time password via some existing
  communication channel, for example by sending a code by SMS or reading out a few
  numbers over the phone. The password only needs to be secret for the duration of
  the protocol; after the session is established, it is no longer needed.
* It's okay for the password to be fairly low-entropy; for example, a six-digit
  number should be plenty. This is because in every run of the protocol, an
  adversary (even one who can perform a person-in-the-middle attack) only gets a
  single attempt at guessing the password. If somebody is trying to attack the
  protocol, it will fail and the users have to retry. If it is successful, either
  the adversary guessed correctly, or the users know that there was no malicious
  interference. Users will most likely only retry a few times, which gives the
  adversary only a small number of attempts to guess the password.
* Users need some way of connecting to each other and sending each other messages.
  This could be via an untrusted relay server, or via a peer-to-peer protocol
  such as WebRTC. The communication layer is not part of this prototype.
* The protocol sends a total of 5 messages: Alice and Bob send each other a first
  message (these can be sent concurrently); after receiving the first message,
  Alice and Bob each send a second message in reply; and after receiving the
  second message, one of the two (determined by the protocol) sends a third
  message to the other. This means the protocol takes a total of 3 one-way
  network delays (1.5 round trips). This means it works best if both users are
  online at the same time.
* After the protocol completes, both users have a fully initialised Signal
  protocol session that either user can use to send a message to the other.
  Once the session is set up, it's fine for users to go offline, as long as
  encrypted messages have some way of getting delivered to the recipient when
  they come back online (probably via an untrusted relay server).
* Once the Signal session is set up, both users know each others' public keys,
  without having to trust any key database. This is useful for various purposes.
  For example, if Alice is a member of a group and she has just established a
  Signal protocol session with Bob, Alice can now add Bob's public key to the
  group, which will allow every other member of the group to establish a direct
  Signal session with Bob without having to perform another password exchange.

The code of this prototype is in `index.html`. It uses a
Rust implementation of SPAKE2 compiled to WebAssembly, and the
[JavaScript implementation of libsignal-protocol](https://github.com/signalapp/libsignal-protocol-javascript).
Most of the code of the prototype only really exists in order to satisfy
libsignal-protocol's rather obtuse and poorly-documented API.

This is a proof-of-concept prototype that has not undergone any review or testing,
or verification. Use it at your own risk under the terms of the
[MIT license](https://opensource.org/licenses/MIT), with no warranty whatsoever.
