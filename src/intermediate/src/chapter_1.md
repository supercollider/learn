# Chapter 1: SuperCollider architecture

Ah, the intermediate book! If you've launched in here, I'm assuming that you
have a bit of programming experience. We'll move a bit faster and delve
into far more technical details, and we'll kick off with a very interesting
part of SuperCollider — its architecture.

SuperCollider is an audio programming language, but there's a lot more to it
than just a textual interface. More specifically, its interesting features are
a super-efficient real-time DSP engine with an unusual server-client
architecture that isn't really found in any other audio software.

Server-client separation can be confusing and, I'll readily admit, has some
disadvantages. But it is there for very important reasons. Understanding it,
and why it exists, is critical to mastering SC.

## Design goals of SC

Real-time audio software is under tight time constraints. If a single buffer of
samples takes too long to compute, a horrible glitch is audible. When you're
performing a live work in front of hundreds of people, you generally do not
want your audio to glitch.

Efficiency is a huge concern for audio software that advertises itself as
suitable for real time performance. This is true today, and it was especially
true for computers in 1996. For a lot of software, performance is convenience.
For real-time audio, performance is reliability.

SuperCollider's server not only needs to be efficient, but it needs to be
flexible. You have to be able to send it arbitrary sound synthesis networks on
the fly, and it has to compile and run those algorithms under real-time
constraints.

## Server-client separation

In SuperCollider 1 and 2, there was no server-client separation. The language
connected directly to audio driver callbacks, and *every* operation in the
SuperCollider environment was in the audio thread, where everything must arrive
exactly on time.

This means that not only did UGens have to race against the clock to prevent
glitches, but so did an entire interpreted programming language. Even with a
cleverly written language that had real-time safe garbage collection, there are
some operations that simply cannot be done safely with this scheme — loading a
large sound file from disk would usually cause some kind of glitch, so it would
have to be done before a composition starts. 

Modern audio software usually runs separate threads for audio and other non-DSP
(such as graphics). However, back in the 90's, threads were not as powerful and
as widespread on consumer machines.

In SuperCollider 3, the decision was made to split the language and server into
two separate processes. Not only does this allow unsafe operations to be
relegated to one process, but this also keeps a nice separation of concerns
between audio processing and control, allowing for all manner of alternate
clients, alternate servers, multiple clients, multiple servers, and servers
running on a different machine.

## The servers

The SuperCollider server programs are *scsynth* and *supernova*. They both do
roughly the same thing. supernova is a newer, ground-up rewrite of scsynth that
adds support for efficient parallel processing on multi-core CPUs (explicitly
invoked using `ParGroup`).

Both are actively maintained, but supernova is more experimental and buggy due
to its younger age. Furthermore, supernova is not supported on Windows yet, and
was not packaged with the macOS binary build until version 3.10. If you're a
new SC user, scsynth is recommended.

Since scsynth and supernova implement much of the same functionality, I will
use "scsynth" as a shorthand for "scsynth or supernova" throughout these
tutorials. "The servers" gets a little bit awkward in writing especially since
most users are working with a single erver.

## A quick OSC primer

scsynth talks to the client using Open Sound Control (OSC). Despite the name,
OSC is not very audio-specific at all — it's a lightweight binary message
protocol that lets you send over arrays of common data types. OSC is usually
sent over TCP or UDP.

An OSC message starts with an address, indicated with a forward slash. This is
interpreted as the name of a "command" that the receiver understands. It is
then followed by a heterogeneous array containing any mixture of strings,
floats, integers, binary data, and timestamps.

For example, `/s_new` is a command that causes scsynth to create a new Synth
node — analogous to a MIDI note on. In sclang, you can send a message to
scsynth using the `NetAddr:sendMsg` instance method:

    Server.default.addr.sendMsg(['/s_new', "piano", -1, 0, "velocity", 0.5]);

This OSC message tells scsynth to create a new Synth from the `piano` SynthDef,
setting `velocity` to 0.5. It is roughly equivalent to

    Synth(\piano, [velocity: 0.5])

There are a few more details like OSC bundles, but we won't go into them yet.
Check the [Open Sound Control](http://opensoundcontrol.org/) official website,
which clearly explains the protocol as well as its binary representation.

## sclang: what's so special about it?

Some programmers come to SuperCollider and are understandably highly skeptical
of the idea of learning a new programming language, especially one that was
designed in the 90's. And I won't mince words with you: sclang is far from
perfect. It has aged better than some of its contemporaries, but has some
really odd design decisions and long-standing limitations. With SuperCollider's
server-client architecture, why not leave the langauge behind and use an
established one? After all, others have written clients for SC in Scala,
Python, Lua, Haskell, and many other languages.

First off, sclang is designed to be somewhat real-time safe in practice. sclang
isn't under the constraints of an audio thread, but it does need to sequence
events with musically accurate timing under normal conditions. Not many
interpreted languages are designed with this in mind.

Second, sclang has a feature known as multichannel expansion. These three lines
are identical:

    // Naive way:
    [SinOsc.ar(440), SinOsc.ar(660), SinOsc.ar(880)]

    // "Don't Repeat Yourself" way by most programming languages:
    [440, 660, 880].collect { |freq| SinOsc.ar(freq) };

    // Best way using multichannel expansion:
    SinOsc.ar([440, 660, 880])

Most programming languages would at best offer the second syntax. But the third
one is true idiomatic sclang, and it's arguably more terse and elegant. I know
you may be thinking that focusing on syntax is shallow, but this makes a *huge*
difference when actually working on sound synthesis. Being able to duplicate
any part of a sound synthesis graph with ease gives sclang a convenience
advantage over both ordinary programming languages and graphical dataflow
languages.

Finally, if you are a SuperCollider user, you should be aware that sclang is
the #1 SuperCollider client. By only learning an alternate client, you are
distancing yourself from the SC community and limiting your ability to receive
or give help. Most SC users are sclang users, and that's a very important
network effect if you're new to the platform.
