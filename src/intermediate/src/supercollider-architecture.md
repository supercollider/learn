# SuperCollider architecture

SuperCollider is first and foremost an audio programming language, but there's
a lot more to it than just a textual interface. More specifically,
SuperCollider consists of a audio server architected to support for on-the-fly
definition and reuse of DSP algorithms. Uniquely, the audio is separated from a
client, which provides the actual sequencing and control.

The best way to summarize how SC differs from its peers is that it's not a toy.
It's robust, efficient, and meant to gracefully handle a wide range of sound
synthesis models, no matter how extreme. Design decisions in SC have been made
with these goals in mind. It's not necessarily perfect software, but if didn't
do its job pretty dang well, I wouldn't be here writing this tutorial.

In this chapter, we'll discuss specific details of server-client separation and
how the server works, and explain why they are important. We won't get very
deep into the implementation of SC — just the architecture that you, the user,
have to work with and understand to fully harness the power of the platform.

## Real-time audio constraints

Real-time audio software is under tight time constraints. If a single buffer of
samples takes too long to compute, a horrible glitch is audible. When you're
performing a live work in front of hundreds of people, you generally do not
want your audio to glitch.

Efficiency is a huge concern for audio software that advertises itself as
suitable for real time performance. This is true today, and it was especially
true for computers in 1996. For a lot of software, performance is convenience.
For real-time audio, performance is reliability.

The little CPU meter that you see on many audio programs, SuperCollider
included, is an indicator of how fast DSP is happening. If CPU usage is low and
stable, you can probably rest easy knowing that the program won't glitch out.
If it's seriously fluctuating, you are in trouble.

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
adds support for efficient parallel processing on multi-core CPUs, which we'll
discuss later in this chapter.

Both are actively maintained, but supernova is more experimental and buggy due
to its younger age. Furthermore, supernova is not supported on Windows yet, and
was not packaged with the macOS binary build until version 3.10. If you're a
new SC user, scsynth is recommended.

Since scsynth and supernova implement much of the same functionality, I will
use "scsynth" as a shorthand for "scsynth or supernova" throughout these
tutorials.

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

It seems nice. Unfortunately, a naive implementation of this formula really
starts to buckle when put under real-world conditions. Musicians want polyphony
— spawning the same synthesis algorithms over and over again from a template
like on a keyboard synthesizer. The ability to efficiently define and reuse DSP
algorithms is a rarely mentioned, but very important area where SC really
shines.

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
signal to deal with. In DSP parlance, control rate signals are "downsampled
processing."

Block-based sample processing is fairly common in well-written audio software,
but marking some signals as audio and others as control is a somewhat SC-unique
deal.

### Layer 2: UGens and Units

A UGen (unit generator) is a chunk of DSP code, such as an oscillator or filter.
When instantiated, it creates a *unit*, which is an object that processes input
signals and produces output signals.

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

As soon as your arrangement of Synths gets complex with multiple Synths writing
to the same buses, order becomes very important. For example, if you have
instrument Synths and a separate Synth for an effect, the effect needs to
process its audio *after* the instrument. `/s_new` has a parameter which allows
you to insert the Synth relative to another.

An entirely linear approach to Synth order gets a little bit flimsy the more
complex the order is. Groups offer a solution for robust, hierarchical Synth
order, and the hierarchy formed is known as the "server tree," and the Synths
and Groups within as "nodes."

supernova has a special feature known as a "parallel group" (`ParGroup`) where
you *don't* care about order, and instead giving supernova a chance to split
the Synths' workload across different threads. Since many musical applications
involve separate, superimposed sound events that don't interact with each other
(multiple tracks, polyphony), this can result in tremendous CPU improvements
when used properly.

## Conclusion

SuperCollider built unlike pretty much any other audio software out there, but
a lot of its quirky designs are well justified for its goals as an efficient
and flexible sound synthesis platform. The language and server are built from
the ground up for real-time safety, and the `SynthDef`/`Synth` distinction
allows for arbitrary polyphony with low overhead.
