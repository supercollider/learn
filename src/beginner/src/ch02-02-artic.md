# Volume and Articulation

So far we've seen how to write a simple melody, but the results have been a little robotic. So in
this section we're going to look at ways we can vary the amplitude (also known as velocity) and
articulation of notes.

## Stacatto to Legato 

While ```\dur``` controls the length of the note event, we can control the articulation of each note
with ```legato```:

```
(
// stacatto
Pbind(
  \degree, Pseq([0, 0, 4, 4, 5, 5, 4, 3, 3, 2, 2, 1, 1, 0]),
  \dur, Pseq([1, 1, 1, 1, 1, 1, 2], 2),
  \legato, 0.1
).play
)
```

```
(
// legato
Pbind(
  \degree, Pseq([0, 0, 4, 4, 5, 5, 4, 3, 3, 2, 2, 1, 1, 0]),
  \dur, Pseq([1, 1, 1, 1, 1, 1, 2], 2),
  \legato, 1
).play
)
```

and obviously we can vary the legato if we so desire:

```
(
Pbind(
  \degree, Pseq([0, 0, 4, 4, 5, 5, 4, 3, 3, 2, 2, 1, 1, 0]),
  \dur, Pseq([1, 1, 1, 1, 1, 1, 2], 2),
  \legato, Pseq([0.1, 0.1, 0.1, 0.1, 0.1, 0.1, 0.9], 2) 
).play
)
```

## Volume

We can vary the amplitude directly with the ```\amp``` key. Amplitude is a value between 0 (silence)
and 1 (maximum volume). 

```
(

Pbind(
  \degree, Pseq([0, 0, 4, 4, 5, 5, 4, 3, 3, 2, 2, 1, 1, 0]),
  \dur, Pseq([1, 1, 1, 1, 1, 1, 2], 2),
  \amp, 0.5
).play
)
```

We can also vary it from note to note quite easily:

```
(
Pbind(
  \degree, Pseq([0, 0, 4, 4, 5, 5, 4, 3, 3, 2, 2, 1, 1, 0]),
  \dur, Pseq([1, 1, 1, 1, 1, 1, 2], 2),
  \amp, Pseq([0.8, 0.5, 0.6, 0.4], inf)
).play
)
```

This probably looks quite familiar except for one thing. What is this ```inf``` value. This is
SuperCollider's way of defining infinity, and when used in a pattern in this way it tells
SuperCollider to repeat this pattern for ever. This is useful when you have a pattern that repeats
for the entirety of a piece, and you don't want to work out how many times it should repeat.

This works, if you remember, because Pbind stops creating new note events when one of it's patterns
ends.

```\amp``` isn't the only key that defines the amplitude. ```Pbind``` also supports the ```\db```
key. This value allows you to define the value in decibels.

[Explanation of decibels go here.]

```
(
Pbind(
  \degree, Pseq([0, 0, 4, 4, 5, 5, 4, 3, 3, 2, 2, 1, 1, 0]),
  \dur, Pseq([1, 1, 1, 1, 1, 1, 2], 2),
  \db, Pseq([-2, -4, -3, -6], inf)
).play
)
```

