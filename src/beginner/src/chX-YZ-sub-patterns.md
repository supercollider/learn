# Generating Patterns from Patterns

In music, something we often want to do is take a simple pattern and develop it. A pianist might
take a chord progression and turn it into an arpeggiated bass line. A jazz pianist will take a
simple melody line and add decorative notes around to create a more complex melodic line. In
SuperCollider we can do that too by using a pattern as in instrument.

## Transforming a Simple Melodic Line With \Phrase

So we'll start with a simple melodic line:


```
TempoClock.default.tempo=1.3

(
~melody = Pbind(
  \note, Pseq([0, 1, 3, 2,1,-1,0,2,1,0]),
  \dur,Pseq([1, 1, 2, 1, 0.5,2,1,0.5,1,2])
)
)

~melody.play
```

Nothing unusual so far. So let's change things up a little:

```SuperCollider
TempoClock.default.tempo=1.3

(
~melody = Pbind(
  \type, \phrase,
  \instrument, \improv,
  \note, Pseq([0, 1, 3, 2,1,-1,0,2,1,0]),
  \dur,Pseq([1, 1, 2, 1, 0.5,2,1,0.5,1,2])
)
)

~melody.play
```

Obviously we are familiar with ```\instrument``` at this point, but what about ```\type \phrase```.
```\type``` is a special key that can be used to tell SuperCollider what type of pattern we are
creating. So far we've used the default pattern, which is used for creating Synths on the server,
and sending them notes. However, there are a whole range of other kinds of patterns that we can
create in SuperCollider for everything from midi, to mixing and everything in between [fn:: You can
also create you own special patterns, something we'll discuss in a later chapter].

The ```phrase``` pattern is used to send events from one pattern to another pattern. So let's do
this:

```SuperCollider
(
Pdef(\improv, {|note, dur|

	Pbind(
    \instrument, \default,
    \dur, dur/4,
    \note, Pseries(note, 1),
    \legato, 0.3
  )
});
)

~melody.play
```

And instantly our rather boring melody from before has become a lot more interesting. 

So instead of creating a SynthDef to play our event, we have instead used ```Pdef``` which contains
a function that takes three arguments and returns a Pbind. Those arguments are set by the
originating event using keys within that event (in this case `\note` and `\dur`). The pattern inside
Pdef only knows about keys from the original pattern if they were passed in as values to the
enclosing function.

To see how
this works we'll use an event:

```SuperCollider
(
(\type: \phrase, 
 \instrument: \improv,
 \note: 0,
 \dur:1
).play
)
```

So this event sent the note `0` to ```Pdef``` and Pdef generated 4 notes (C, D, E and F) using a
```Pseries```. Each of these notes had a duration that was 1\4 of the duration of the originating
event. However, note that the pattern returned by ```Pdef``` is infinite - instead this event will
keep playing for the total duration of the original event:

```SuperCollider
(
// decorative line	
(\type: \phrase, 
 \instrument: \improv,
 \note: 0,
 \amp: 0.1,
 \dur:1
).play;

// bass note for duration of the event
(\note: 5,
 \octave: 4,
 \dur:1,
 \amp:
0.5
).play
)
```

So when we sent our original melodic line to ```Pdef(\improv)```, each note from the originating
melody was transformed into four notes - where each of the four notes had a duration equal to 1/4 of
the original melody's note. If a note from the original melody was an 1/8th note, then it will be
transformed into four 1/32th notes.

[DIAGRAM SHOWING THE TRANSFORMATION]

## Legato and \Phrase

In the previous section we saw that we can use legato to change the articulation of a note. If we
want staccato we might set it to a low value (e.g. 0.1) and if we want legato we could set it to
`1`. What happens if we use ```legato``` for a ```\phrase``` event?

```
(\type: \phrase, 
 \instrument: \improv,
 \note: 0,
 \amp: 0.1,
 \legato: 0.2,
 \dur:1
).play;
```

This time only one note was played. If we use our original melody with a more aggressive
```legato``` value:

```SuperCollider
(
Pbind(
  \type, \phrase,
  \instrument, \improv,
  \note, Pseq([0, 1, 3, 2,1,-1,0,2,1,0]),
  \dur,Pseq([1, 1, 2, 1, 0.5,2,1,0.5,1,2])
).play
)
```

Then for each note of the melody we get two notes played, followed by a silence of duration equal to
the previous two notes. So we can see that the rules for time and ```phrase``` events
are identical to the other events we've seen so far. The total duration that the event is played for
is calculated using ```sustain``` (which as you hopefully remember is calcuated as ```legato *
	dur```), while the time between beats is calculated using ```\dur```.

While this can allow us to occasionally do some fun things:

```SuperCollider
(
Pbind(
  \type, \phrase,
  \instrument, \improv,
  \note, Pseq([0, 1, 3, 2,1,-1,0,2,1,0]),
  \dur,Pseq([1, 1, 2, 1, 0.5,2,1,0.5,1,2]),
  \legato, Prand([0.2, 0.4, 0.6, 0.8], inf),
).play
)
```

most of the time you'll just want to set ```\legato``` to 1 as this will do what you expect it to.

## Passing Event Values to the '\Phrase Pdef'

So earlier in one of the examples you may have noticed that the originating event passed an
```\amp``` value and this was somehow passed to the default synth. So let's see how this works:

```SuperCollider
(
	Pdef(\test, {
	  Pbind();
	});

	(\type: \phrase,
	 \instrument: \test, 
	 \note: 0, 
	 \dur: 1).play
)
```

So event though we don't set anything in the ```Pbind``` returned by the ```Pdef``` it still plays
the note sent by the originating event. So what happens if our function inside ```Pdef``` takes
`note` and `dur` as parameters:

```SuperCollider
(
	Pdef(\test, {|note, dur|
	  	Pbind();
	});

	(\type: \phrase,
	 \instrument: \test, 
	 \note: 0, 
	 \dur: 1).play
)
```

We get an identical outcome. So when we send a `phrase` event to the named ```Pdef```, this event is
passed into the Pbind returned by the function in Pdef. It's only if we override some of those keys
in the Pdef that anything changes:

```SuperCollider
(
	Pdef(\test, {|note, dur|
	  	Pbind(\note, [3,5]);
	});

	(\type: \phrase,
	 \instrument: \test, 
	 \note: 0, 
	 \dur: 1).play
)
```

and as we saw above, we can set the values to anything that we like - they don't necessarily depend
upon the original event. However, by defining the arguments for the function inside the Pdef we get
easy access to the parameters set on the original event and then can manipulate them:

```SuperCollider
(
Pdef(\test, {|note, dur|
  Pbind(\note, [note, note+2, note+4, note+6]);
});

(\type: \phrase,
 \instrument: \test, 
 \note: 0, 
 \dur: 1).play
)
```

## More Complicated Manipulations of the Original Pattern

So far our Pdef pattern has treated each incoming event identically, however we can have custom
responses to each event, because the function inside the Pdef returns a new Pbind each time it is
called:

```
(
Pdef(\test, {
  Pbind(\note, 8.rand);
});
)

// evaluate this event multiple times
((\type: \phrase,
 \instrument: \test, 
 \note: 0, 
\dur: 1).play)
```

So this allows us to do things like the following:

```SuperCollider
(
Pdef(\improv, { |note, dur|
  var notes = 3.rand + 2; // generate a random number between 2 and 4
  var duration = dur/notes; // a duration that will be 1/2, 1/3, or 1/4
  duration.postln;
	Pbind(
    \instrument, \default,
    \dur, duration, // depending upon duration value we will return either 2, 3, or 4 notes
    \note, Pseries(note, 1),
    \legato, 0.3
  )
})
)


(
(\type: \phrase, 
 \instrument: \improv,
 \note: 0,
 \amp: 0.1,
 \legato: 1,
 \dur:1
).play;
)

(
~melody = Pbind(
  \type, \phrase,
  \instrument, \improv,
  \note, Pseq([0, 1, 3, 2,1,-1,0,2,1,0]),
  \dur,Pseq([1, 1, 2, 1, 0.5,2,1,0.5,1,2]),
  \legato, 1,
).play
)
```

Now while this may not be the musical of examples, hopefully you can see how you might use this to
generate a range of musical gestures - either randomly, or based upon values passed in from the
original event, or some other criteria relevant to your piece.