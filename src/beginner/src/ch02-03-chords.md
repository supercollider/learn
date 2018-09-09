
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

## Harmonizing a Melody

So now let's look at how we can harmonize an existing melody by returning to old standby - Twinkle
Twinkle Little Star.

So first of all let's create our chord progression:

```
(
var c = [0, 2, 4];  // C E G
var f = [0, 3, 5];  // C F A
var g7 = [-1, 1, 3, 4]; // B D F G

var progA = Pseq([c, f, c, f, c, g7, c]);
var progB = Pseq([c, f, c, g7, c, f, c, g7]);

var a = [60, 60, 67, 67, 69, 69, 67, 65, 65, 64, 64, 62, 62, 60];
var b = [67, 67, 65, 65, 64, 64, 62];

var prog = Pbind(
  \octave, 5,
  \degree, progA ++ progB ++ progB,
  \dur, Pseq([4, Pseq([2], 14), 4, Pseq([2], 6)]));
```

So hopefully this all looks fairly similar to how we created melodies in a previous chapter. All
that's changed here is that now we are playing a chord, rather than a single note. Now let's add
the melody:

```
(
var c = [0, 2, 4];  // C E G
var f = [0, 3, 5];  // C F A
var g7 = [-1, 1, 3, 4]; // B D F G

var progA = Pseq([c, f, c, f, c, g7, c]);
var progB = Pseq([c, f, c, g7, c, f, c, g7]);

var a = [60, 60, 67, 67, 69, 69, 67, 65, 65, 64, 64, 62, 62, 60];
var b = [67, 67, 65, 65, 64, 64, 62];

var prog = Pbind(
  \octave, 5,
  \degree, progA ++ progB ++ progB,
  \dur, Pseq([4, Pseq([2], 14), 4, Pseq([2], 6)]));

var melody = Pbind(
  \octave, 6,
  \midinote, Pseq(a) ++ Pseq(b, 2) ++ Pseq(a),
  \dur, Pseq([1, 1, 1, 1, 1, 1, 2], 6),
);

Ppar([melody, prog]).play
)
```

So here you can see we have a new pattern type: ```Ppar```. Ppar allows us to play two independent
patterns simultaneously - allowing us to construct more complex musical arrangements. As we'll see
in future chapters - pattern constructs such as this allow us to flexibly and quickly build up
complex musical arrangements quite easily.