# Arpeggiation

So let's now use some of the techniques we've been discussing to build an arpeggiator. Let's start
with a simple chord progression:

## A Failed Arpeggiator

```
~chordProgression = [ [ 0, 4, 7 ], [ 5, 9, 12 ], [ 7, 11, 14, 17 ], [ 0, 4, 7 ] ];
```

And let's send these events to a ```Pdef``` called ```\arp```:

```
~myArp = (
Pbind(
    \type, \phrase,
    \instrument, \arp,
  	\chord, Pseq(~chordProgression,inf),
    \legato, 1
    \dur,4,
)
)
```

Note that in the arpeggio above we have defined ```\legato``` as 1 so that the arpeggio lasts for
the duration of our chord, rather than being cut of part way through (the default for legato being
0.8). Now let's finally define our arpeggiator:

```
(
Pdef(\arp, {|chord|
  Pbind(
    \instrument, \default,
    \dur, 1/2,
    \note, Pseq(chord, inf),
    \legato, 0.7
    )
})
);

~myArp.play;
```

Unfortunately this didn't work. Instead our code fails and we get a fairly incomprehensible error
message. To see what is happening run the following code:

```
(
~chordProgression = [ [ 0, 4, 7 ], [ 5, 9, 12 ]];

~test = Pbind(
    \type, \phrase,
    \instrument, \test,
    \chord, Pseq(~chordProgression).trace(),
    \dur,4,
    \legato, 1
);

Pdef(\test, {|chord|
    chord.postln;
    nil;
}
);
~test.play;
  
)
```

## Preventing Multi-Channel Expansion

Our Pbind is generating a chord, but ```Pdef(\test)``` is receiving individual notes. This is
because (as hopefully you remember from previous chapters) SuperCollider expands arrays, creating a
new event for each member of the array. So when our Pseq returns ```[0,4,7]```, SuperCollider
expands that into three events, the first event has `chord` set to 0, the second has `chord` set to
4 and the third has `chord` set to 7. Each event is sent to a new instance of ```Pdef(\test)```. If
we want to pass an array to Pdef, then we need to wrap our chord in another array. There are two
ways to
do this. The first way would be the following:

```
(
~chordProgression = [ [[ 0, 4, 7 ]], [[ 5, 9, 12 ]]];

~test = Pbind(
    \type, \phrase,
    \instrument, \test,
    \chord, Pseq(~chordProgression).trace(),
    \dur,4,
    \legato, 1
);

Pdef(\test, {|chord|
    chord
    nil;
}
);
~test.play;
  
)
```

The second is to use ```collect``` in the ```Pseq``` and wrap each chord in another array:

```
(
~chordProgression = [ [ 0, 4, 7 ], [ 5, 9, 12 ]];

~test = Pbind(
    \type, \phrase,
    \instrument, \test,
    \chord, Pseq(~chordProgression).collect({|c| [c]}),
    \dur,4,
    \legato, 1
);

Pdef(\test, {|chord|
  ("chord: " ++ chord).postln;
    nil;
}
);
~test.play;
  
)
```

## A Working Arpeggiator

So let's build a working arpeggiator:

```
(
~chordProgression = [ [ 0, 4, 7 ], [ 5, 9, 12 ], [ 7, 11, 14, 17 ], [ 0, 4, 7 ] ];

Pdef(\arp, {|chord|
  Pbind(
    \instrument, \default,
    \dur, 1/2,
    \note, Pseq(chord, inf),
    \legato, 0.7
    )
});

~myArp = Pbind(
  \type, \phrase,
  \instrument, \arp,
  \chord, Pseq(~chordProgression).collect({|c| [c]}),
  \legato, 1,
  \dur,4,
);

~myArp.play;
)
```

## An Alberti Bass Line

It worked! How about we try some more arpeggiators. Let's start with an Alberti Bassline:

```
~chordProgression = [ [ 0, 4, 7 ], [ 5, 9, 12 ], [ 7, 11, 14], [ 0, 4, 7 ] ];

Pdef(\arp, {|chord|
  Pbind(
    \instrument, \default,
    \dur, 1/2,
    \note, Pseq(chord[[0,2,1,2]], inf),
    \legato, 0.7
    )
});
```

The magic here is ```chord[[0,2,1,2]]``` which reorganizes our chord into the alberti bass line:

```SuperCollider
v = [0,1,2,3];
v[0]; // returns 0
v[[1,2]] // returns [0,1]
v[[0,0]] // returns [0,0]
```

By passing an array of indices (```[1,2]``` rather than ```1```) we get an array of all the values
contained at those indices.

So in the case above:

```SuperCollider
chord = [0,4,7];
chord[[0,2,1,2]] // [0,7,4,7]
```

However this doesn't work properly for chords larger than triads, so let's fix that:

```
(
~chordProgression = [ [ 0, 4, 7 ], [ 5, 9, 12 ], [ 7, 11, 14, 17 ], [ 0, 4, 7 ] ];

Pdef(\arp, {|chord|
  var arpSeq = [0] ++ (chord.size-1) ++ (1..chord.size-1);

  Pbind(
    \instrument, \default,
    \dur, 1,
    \note, Pseq(arpSeq, inf),
    \legato, 0.7
    )
});

~myArp = Pbind(
  \type, \phrase,
  \instrument, \arp,
  \chord, Pseq(~chordProgression).collect({|c| [c]}),
  \legato, 1,
  \dur,4,
);

~myArp.play;
)
```

And obviously we could use a more interesting rhythm if we wanted:

```
(
~chordProgression = [ [ 0, 4, 7 ], [ 5, 9, 12 ], [ 7, 11, 14, 17 ], [ 0, 4, 7 ] ];

Pdef(\arp, {|chord|
  var arpSeq = [0, Rest()] ++ (chord.size-1) ++ (1..chord.size-1);

  Pbind(
    \instrument, \default,
    \dur, Pseq([0.5, 0.5, 1.5, 0.5, 1], inf),
    \note, Pseq(arpSeq, inf),
    \legato, 0.7
    )
});

~myArp = Pbind(
  \type, \phrase,
  \instrument, \arp,
  \chord, Pseq(~chordProgression).collect({|c| [c]}),
  \legato, 1,
  \dur,4,
);

~myArp.play;
)
```

or even pass our sequence and rhythm in:

```
(
~chordProgression = [ [ 0, 4, 7 ], [ 5, 9, 12 ], [ 7, 11, 14, 17 ], [ 0, 4, 7 ] ];

Pdef(\arp, {|chord, rhythm|
  var arpSeq = [0] ++ (chord.size-1) ++ (1..chord.size-1);

  Pbind(
    \instrument, \default,
    \dur, Pseq(rhythm, inf),
    \note, Pseq(arpSeq, inf),
    \legato, 0.7
    )
});

~myArp = Pbind(
  \type, \phrase,
  \instrument, \arp,
  \chord, Pseq(~chordProgression).collect({|c| [c]}),
  \rhythm, Pseq([[[0.5, 1.5, 1, 1]], [[1.5, 1, 0.5, 1]]], inf),
  \legato, 1,
  \dur,4,
);

~myArp.play;
)
```