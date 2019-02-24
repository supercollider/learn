# Introduction

In March of 1996, James McCartney [announced to the comp.music.research][announcement] mailing list
the release of his new real-time synthesis language. Originally a Max/MSP object known as Pyrite,
SuperCollider was now available as a standalone program. SuperCollider was inspired by older
programs such as the MUSIC-N family and Csound, but designed to be easier to use and optimized for
real-time processing.

In the late 90's, SuperCollider gained popularity in the academic computer music research community.
Two major versions later, SuperCollider 3 — a major architectural rewrite that separated the
programming language from the audio server — was released as free software under the GNU General
Public License. 

[announcement]: https://groups.google.com/forum/#!topic/comp.music.research/g2f9EcL1mUw

## Why SuperCollider?

**Flexibility.** SuperCollider is lower level than most modern audio synthesis environments, only a
few steps away from writing raw DSP code. Like most "deep modular" programs, it gives you a
tremendous spectrum of options for designing sound.

**Interface.** Graphical patching environments, either software or hardware, have been the dominant
approach to modular audio synthesis throughout history. While visual programming is powerful, many
find it frustrating when scaling up to large patches with hundreds of nodes. Code seems like a
strange interface for sound synthesis, but you might find it surprisingly ergonomic once you spend
some time with it! SC is also hugely popular within the live coding community, and there are many
artists who perform by writing and running SC code live.

**Education.** SuperCollider is a great environment for understanding the fundamentals of sound
synthesis, and many educators teach it as a first programming language.

**Stability.** Efficiency was a major selling point in the early days of SuperCollider, and today it
continues to rival many commercial and open source platforms in signal processing performance. The
language and server were architected for a high degree of real-time safety. For example, the server
has its own real-time safe memory allocator so that the creation and destruction of synthesizer
nodes is lightning fast (in particular avoiding system calls to `malloc` and `free`).

**Free software.** SuperCollider is made available under the GNU GPL. It's available at no cost, and
maintained by a vibrant community of volunteer developers. Unlike most software of its age,
SuperCollider development remains very active — as of 2019, SC averages several commits per week and
has a regular release schedule. The SC developers are a friendly bunch and will often respond to
tickets within hours.

## Why this tutorial?

SuperCollider's learning curve is somewhat notorious. I won't mince words — SC is a fairly tricky
program to pick up, and it often takes a long time to reach proficiency in it. This is not helped by
the fact that the official tutorials in SuperCollider don't go very far. As a result, many SC
educators have individually stepped up over the years with excellent learning materials. In
particular, we'd like to shout out [A Gentle Introduction to SuperCollider][Ruviaro] and [Eli
Fieldsteel's video tutorials][Fieldsteel].

[Ruviaro]: https://ccrma.stanford.edu/~ruviaro/texts/A_Gentle_Introduction_To_SuperCollider.pdf
[Fieldsteel]: https://www.youtube.com/channel/UCAf4fP8QzKkJ_t-c1F2v27Q

"Learn SuperCollider" is a new effort to create an official community-written learning resources for
the program, taking our favorite ideas from the above tutorials.

As SuperCollider 3 has evolved over its 15+ years of existence, multiple ways to accomplish the same
thing have popped up in the language, and many older tutorials are inconsistent with each other.
Learn SC is intended to be very opinionated and prescriptive concerning use of the SuperCollider
language. We are big on recommending the best practices and discouraging older styles of SC code.

Another important focus of these books is to emphasize the musicality of SuperCollider. When new
concepts are introduced, we emphasize how they can be used in music and sound design and celebrate
the huge variety of workflows that SuperCollider offers.
