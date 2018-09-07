# Chapter 1: SuperCollider architecture

Ah, the intermediate book! If you've launched in here, I'm assuming that you
have a bit of programming experience. We'll move a bit faster and delve
into far more technical details, and we'll kick off with a very interesting
part of SuperCollider — its architecture.

SuperCollider is an audio programming language, but there's a lot more to it
than just a textual interface. More specifically, SuperCollider consists of a
audio server architected to support for on-the-fly definition and reuse of DSP
algorithms. Uniquely, the audio is separated from a client, which provides the
actual sequencing and control.

The best way to summarize how SC differs from its peers is that it's not a toy.
It's robust, efficient, and meant to gracefully handle a wide range of sound
synthesis architectures no matter how extreme. Every architectural decision has
been made with these goals in mind.

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

## sclang: what's so special about it?

Some programmers come to SuperCollider and are understandably highly skeptical
of the idea of learning a new programming language, especially one that was
designed in the 90's. And I won't mince words with you: sclang is far from
perfect. It has aged better than some of its contemporaries, but has some
really odd design decisions and long-standing limitations. With SuperCollider's
server-client architecture, why not leave the langauge behind and use an
established one? After all, others have written clients for SC in Scala,
Python, Lua, Haskell, and many other languages.

First off, sclang is "soft real-time safe." sclang isn't under the constraints
of an audio thread like scsynth is, but it does need to sequence events with
musically accurate rhythm under normal conditions, so it is written with
features like real-time safe garbage collection. Few interpreted languages are
designed with this in mind (Lua is a major one).

Second, sclang has a feature known as multichannel expansion. These three lines
are identical:

    // Naive way:
    [SinOsc.ar(440), SinOsc.ar(660), SinOsc.ar(880)]

    // "Don't Repeat Yourself" way, like in most programming languages:
    [440, 660, 880].collect { |freq| SinOsc.ar(freq) };

    // Best way, using multichannel expansion:
    SinOsc.ar([440, 660, 880])

Most programming languages would at best offer the second syntax. But the third
one is true idiomatic sclang, and it's more terse and elegant. I know you may
be thinking that focusing on syntax is shallow, but this makes a *huge*
difference when actually working on sound synthesis in your artistic practice.
Being able to duplicate any part of a sound synthesis graph with ease gives
sclang a convenience advantage over both ordinary programming languages and
graphical dataflow languages.

Finally, if you are a SuperCollider user, you should be aware that sclang is
the #1 SuperCollider client. By only learning an alternate client, you are
distancing yourself from the SC community and limiting your ability to receive
or give help. Most SC users are sclang users, and that's a very important
network effect if you're new to the platform.

Don't get me wrong ­— I'm not here to condemn alternate clients and the hard
work that goes into them. I'm sure you'll like some of them better than than
sclang. But especially for people new to SC, sclang is strongly recommended.

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
most users are working with a single server.

## A quick OSC primer

The client talks to scsynth and vice versa using Open Sound Control (OSC).
Despite the name, OSC is not very audio-specific at all — it's a lightweight
binary message protocol that lets you send over arrays of common data types.
OSC is usually sent over TCP or UDP.

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

## The signal processing hierarchy of scsynth

A simple real-time modular audio application might have the ability to create
and destroy some pre-fab audio processing nodes on the fly, and functionality
to modulate and rewire them.

It seems nice. Unfortunately, in the real world, a naive implementation of this
formula really starts to buckle. Musicians want polyphony — spawning the same
synthesis algorithms over and over again from a template like on a keyboard
synthesizer. The ability to efficiently define and reuse DSP algorithms is a
rarely mentioned, but very important place where SC really outshines its
modular competitors.

In this section, we'll look at scsynth's unique hierarchical approach to
scaffolding of modular audio processing.

### Layer 1: Signals and block-based processing

Currently, all signals in the server are 32-bit floating-points.

Very importantly, audio signals are processed in blocks, not individual
samples. The size of the block is determined by a command-line option to
scsynth. By default, sclang sets a block size of 64. By processing audio in
blocks, DSP code can take advantage of vectorization and caching to help speed
up computation. However, there is a disadvantage that single-sample feedback
is intractable without making your own UGen in C++. (There *is* a hack to get
around this using demand-rate UGens, but I won't discuss it here.)

As you might know from the beginner tutorial, SC signals aren't just audio —
some are control-rate signals, conventionally denoted `.kr` in UGen classes.
These signals define only one sample per block. Performing arithmetic
operations on such signals is significantly faster since there's simply less
signal to deal with.

Block-based sample processing is fairly common in well-written audio software,
but marking some signals as audio and others as control is a somewhat SC-unique
deal.

### Layer 2: UGens and Units

UGens ("unit generators") are like classes, and units are their instances. A
UGen is a chunk of DSP code, such as an oscillator or filter. When
instantiated, it creates a unit that processes input signals and produces
output signals.

In compiled form, UGens physically live in *plugins*, which are shared
libraries (`.scx` files on Windows and macOS, `.so` files on Linux). When
scsynth boots up, it searches for plugin files and loads them. The connective
tissue between scsynth and plugins is known as the *plugin interface*, a binary
interface written in C.

### Layer 3: SynthDefs and Synths

A SynthDef is a blueprint describing how UGens are connected together. When
instantiated, the resulting object is a connection of UGens known as a Synth.
Despite the name, a Synth is nowhere close to the physical synthesizer object.
It's a generic, miniature DSP program that can do pretty much anything from
producing sound to processing sound to analyzing it: whatever is possible with
a combination of UGens.

The client can't directly manipulate UGens or units — it has to first create a
SynthDef using `/d_recv` (`SynthDef:add`) or related commands, then instantiate
that SynthDef as a Synth using `/s_new` (`Synth.new`). To be able to
communicate SynthDefs, SuperCollider defines a binary file format for the
client to write and the server to read. The SynthDef file format is designed to
be very compact: a typical SynthDef may be less than a kilobyte.

SynthDefs can define parameters for the Synths they instantiate and plug them
into UGens like other signals. However, these parameters can only modulate
inputs to UGens — they can't add new UGens, take them away, or rewire the graph
dynamically. It's a bummer sometimes, but it's the price we pay for efficient
and stable modular audio software. Parameters may be directly modulated by the
client using commands such as `/n_set` (`Synth:set`).

Synths also talk to the outside world using special UGens — the big ones are
`In` and `Out`, which allow reading from and writing from buses. A bus in SC is
a signal, uniquely identified by a global index, which any Synth can write to
or read from. Buses that correspond to hardware inputs and outputs are the most
important.

Once again, we have a class vs. instance distinction between SynthDefs and
Synths. This particular class vs. instance distinction is very musically
important, because it enables efficient polyphony by spawning multiple sound
events from a template. Furthermore, as part of scsynth's aim of real-time
safety, spawning and freeing Synths is a very efficient process. You can easily
fire off hundreds of Synths per second for something like granular synthesis.

### Layer 4: Synth order and Groups

As soon as your arrangement of Synths gets complex with multiple Synths
writing to the same buses, order becomes very important. For example, if you
have instrument Synths and a separate Synth for an effect, the reverb needs to
process its audio *after* the effect. `/s_new` has a parameter which allows you
to insert the Synth relative to another.

An entirely linear approach to Synth order gets a little bit flimsy the more
complex the order is. Groups offer a solution for robust, hierarchical Synth
order, and the hierarchy formed is known as the "server tree," and the Synths
and Groups within as "nodes."

Groups also offer a very handy feature for polyphonic work: they also accept
`/n_set` commands, which are broadcasted to all their child nodes. This is
extremely useful if you want to modulate a parameter on a dozen similar nodes
in a polyphonic context.

supernova has a special feature known as a "parallel group" (`ParGroup`) where
you *don't* care about order, and instead giving supernova a chance to split
the Synths' workload across different threads. Since many musical applications
involve separate, superimposed sound events that don't interact with each other
(multiple tracks, polyphony), this can result in tremendous CPU improvements.
