
# Chords

So far all we've played has been melodies. But music would pretty uninteresting without harmony.
Fortunately creating chords and chord progressions in SuperCollider is pretty straightforward.

## A C Chord

So let's play one chord every bar:

```
// C - G progression for infinity
(
Pbind(
  \midinote, [60,64,67],
  \dur, 4).play
)
```

Contrast this with a pattern that plays a single note:
```
// C - G progression for infinity
(
Pbind(
  \midinote, 60,
  \dur, 4).play
)
```

The only difference between them is we now have three notes in an array. If you remember, an array
is a data structure that contains multiple data items in a sequential 'list'. Creates an event for
each value in the array and plays them simultaneously. In this case it results in a chord being
played.

## Simple Chord Progression

If we want to create a chord progression, we can just use Pseq and play back multiple arrays, rather
than single values:

```
// C - G progression for infinity
(
Pbind(
  \midinote, Pseq([[60,64,67], [67,71,74]], inf),
  \dur, 4).play
)
```

Note that the first parameter of ```Pseq``` is an array of arrays: ```[[60, 64, 65], [67,71,74]]```
rather than a single array like we have seen before.

Of course, just as before, we can specify the chord using ```\degree```:

```
// C - G progression for infinity
(
Pbind(
  \root, 0,
  \degree, Pseq([[0,2,4], [4, 6, 8]], inf),
  \dur, 4).play
)
```

and we could even cycle through a cycle of 5ths:

```
(
Pbind(
  \root, Pseq([0, -7, -2, 3]), 
  \degree, [4, 6, 8],
  \dur, 4).play
)
```

