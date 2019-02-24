# Composing in SuperCollider

It’s easy to write code in SC that sounds great for around 10 seconds, but immediately tires itself out. SuperCollider users from beginners to experts have long struggled with the question: how do I turn sound design patches into complete works of music with a beginning, middle, and end?

SC being a programming language, the options are limitless and there’s no specific attempt to impose a workflow on you. But like all music software, it isn’t neutral. When incorporating SC into a compositional practice, the strengths and weaknesses of the SC interface need to be taken into account.

Here are some ideas on how you can incorporate SC into a compositional practice. Probably the most important point before we proceed: *you don't need to constrain yourself to make your piece entirely in SC*. Many people use it as just one layer in a larger mix.

## Live interaction

SC particularly excels one thing: live interaction. Even if you only plan on making fixed-media works, don't discount the option of using real-time interaction in your process. It is totally okay to make a live patch and record yourself using it. The live element doesn't need to become a performance -- it can just be part of your compositional workflow.

One obvious one is to use a MIDI controller with SC. Large-scale compositional form can be created by turning knobs and pressing buttons.

Or you can make a GUI in sclang, and use that as a live interface.

Or you can analyze microphone input and use analysis parameters to modulate parameters of your patch.

Or you can use other hardware interfaces. Hook up sensors via Arduino and SerialPort. On a laptop, you can also make use of the mouse, keyboard, webcam, and accelerometer.

Or you can run blocks of code live -- even if you aren’t interested in live coding as a performance practice, this is often a very easy way to create section transitions.

## Sequencing in sclang

Also be aware of SC's major weakness: note entry via arrays is really, really awkward. I don't recommend typing in melodies with Pbind unless they're very simple.

If you're composing music with relatively traditional notions of melody and harmony, you might want to look into using SC as a MIDI synthesizer rather than a compositional interface. Try composing MIDI in a DAW, routing the MIDI to SuperCollider, and routing SuperCollider's audio output back into the DAW. (Or, of course, hook up a MIDI keyboard and play it, but I already mentioned that.)

There are also specialized score programs such as IanniX and OSSIA that work great with large-scale sequencing in SC. Alternatively, depending on your needs, you could build a timeline view in SC's GUI features -- although that sounds like a ton of work to me.

You can also embrace text-based entry if you like it. One easy way to get sequencing in SC is to write a parser for a domain-specific language for entering notes, drum patterns, etc. I have programmed drum patterns using single characters like this: "k.k. s.th thkk s.t.". Try developing your own personalized sequencer notation that specifically fits your musical practice.

If your piece evolves slowly, you can also use a Routine and schedule when certain synths stop and start. This is a good choice for drone and ambient music.

Unsurprisingly, the SC environment is great for algorithmically generated melody, harmony, and rhythm, which could replace ordinary sequencing.

## Other ideas

Use SC as an effect on an instrument.

Make a patch in SC and record yourself messing around with parameters. Use a DAW to sample your favorite parts and assemble them into a piece.

Design individual note sounds in SC. Record them and load them into a sampler. Use it in a larger work.

## Conclusions

Unlike a lot of other music software, SC has a very open workflow and finding the best process is a matter of exploration.