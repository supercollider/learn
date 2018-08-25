So let's start off by writing a very simple scale:

```
(
Pbind(
  \midinote, Pseq([60, 62, 64, 65, 67, 69, 71, 72]),
  \dur, 1
).play
)
```

Now type ctrl-enter (Windows and Linux) or Cmd-Enter (Mac).

[Put a picture of ctrl-enter and cmd-enter]


In the post window in the bottom right you should also see the following, while in the background you will hear a simple C major scale.

[TODO Screenshot]

Whenever you successfully evaluate a piece of code in SuperCollider, this window will give you a little bit of feedback about what just happened. Don't worry yet about what that means.

#Didn't Hear Anything?

If you see something different then there was probably something wrong with what you just typed in. Check it again, paying careful attention to placement of brackets and commas. In particular SuperCollider requires brackets to be balanced. If all else fails, try copy and pasting the code from the book. If that doesn't fix it there may a deeper problem. Contact the mailing list, forum or Slack channel [links to follow].

If you see the following, then you need to start the SuperCollider sound server:

[TODO Screenshot]

Finally, if you still don't hear anything then make sure that the volume is turned up sufficiently. I make this mistake more often than I'd like.

# Explaining What Just Happened

So we're going to take the piece of code that you just ran [terminology - this is sometimes also called 'evaluating'] and examine each piece of it so that you understand what happened. If you're a confident coder then you can probably skim this part.

[Image with brackets colored]

So the code we just ran is surround by brackets. Whenever we surround code by brackets in the SuperCollider IDE this is our way of telling the IDE that we want it to run all the code between the brackets. Note that whenever you have an open parenthesis - you must have a matching closing parenthesis. So what happens if we remove one of those parentheses:

[Screenshots]

We get the mysterious ```parse error in interpreted text```, but also see that a little further down we see ```unmatched ')' in interpreted text line 1 char 1```. So SuperCollider is telling us that it didn't understand the code ('parse error'), where it thinks the problem starts ('line 1' where the open paranthesis is) and what the problem is ('unmatched )' - that is we don't have a closing parenthesis to match the opening one). If you put the parenthesis back and run the code again you should now hear the scale played back.

Now place the cursor (the blinking horizontal line that marks where new text will appear) after the final parenthesis. See how the entire block is colored yellow. Now remove the final parenthesis. The yellow highlight disappears. This is another way that you can tell if a code block can be run, and if you have missing parentheses.

[IMAGE of BLOCK]

Restore the code block to it's initial state (hint: you can use undo to do this: `ctrl-z` or `cmd-z`). When you reevaluate it it should play normally. Now let's look deeper. First of all what is ```\dur```? This is short for duration, and tells SuperCollider that the code following the comma (```1```) should be used for the duration in beats of the next note.

```\midinote``` is used to tell SuperCollider that the code that follows the comma: ```Pseq([60, 62, 64, 65, 67, 69, 71, 72])``` will generate 'midi notes'. MIDI notes are an industry standard where particular numbers (these are defined as part of the MIDI standard [wikipedia link]) are treated as musical notes. For example middle C is 60, D#/Cb is 61, D is 62, etc.

[Image of the Midi notes].

At this point hopefully you have guessed that ```Pseq``` is being used to define our scale:
+ **60** - C4
+ **62** - D4
+ **64** - E4
+ **65** - F4
+ **67** - G4
+ **69** - A4
+ **71** - B4
+ **72** - C5

So this little bit of code has generated the following sequence of notes:
+ C4 for a duration of 1 beat
+ D4 for a duration of 1 beat
+ D4 for a duration of 1 beat
+ D4 for a duration of 1 beat
+ E4 for a duration of 1 beat
+ F4 for a duration of 1 beat
+ G4 for a duration of 1 beat
+ A4 for a duration of 1 beat
+ B4 for a duration of 1 beat
+ C5 for a duration of 1 beat

[Image of a simple ascending scale here]

Similarly the following piece of code will generate a descending C Major scale, where each note has a duration of 2 beats:

```
(
Pbind(
  \midinote, Pseq([72, 71, 69, 67, 65, 64, 62, 60]),
  \dur, 2
).play
)
```

[Image of a simple descending scale here]

If we want to vary the number of beats, then we use a Pseq for the ```\dur``` key:

```
(
Pbind(
  \midinote, Pseq([60, 62, 64, 65, 67, 69, 71, 72]),
  \dur, Pseq([8, 4, 2, 1, 1/2, 1/4, 1/8, 1/16])
).play
)
```

[Image of what this is here]

Note that when we want to create eighth notes, or sixteenth notes we write it as a fraction (e.g. 1\8 for one eighth of a beat). We could also write these durations as decimal places if we liked - ```0.5``` and ```1/2``` are interchangable.

So what happens with the following code:

```
(
Pbind(
  \midinote, Pseq([60, 62, 64, 65, 67, 69, 71, 72]),
  \dur, Pseq([1, 2, 2])
).play
)
```

It plays C4 for 1 beat, D4 for 2 beats, E4 for 2 beats and then stops. So Pbind will keep playing until one of the sequences runs out of notes to play. If we provided it with infinitely long ```\midinote``` and ```\dur``` sequences then it would play for ever. One way that we can provide such an infinite sequence is by providing a single value. The following will play for as long as you let it (remember ```Ctrl/Cmd-period``` will stop the audio server):

```
(
Pbind(
  \midinote, 60,
  \dur, 1
).play
)
```

So single values result in an infinite repeating sequence.

So now we have enough to create a simple tune:

[Twinkle Twinkle little star score]


```
// Twinkle Twinkle Little Star
(
Pbind(
  \midinote, Pseq([60, 60, 67, 67, 69, 69, 67, 65, 65, 64, 64, 62, 62, 60,67, 67, 65, 65, 64, 64, 62,67, 67, 65, 65, 64, 64, 62,60, 60, 67, 67, 69, 69, 67, 65, 65, 64, 64, 62, 62, 60]),
  \dur, Pseq([1, 1, 1, 1, 1, 1, 2, 1, 1, 1, 1, 1, 1, 2,1, 1, 1, 1, 1, 1, 2, 1, 1, 1, 1, 1, 1, 2,1, 1, 1, 1, 1, 1, 2, 1, 1, 1, 1, 1, 1, 2,1, 1, 1, 1, 1, 1, 2, 1, 1, 1, 1, 1, 1, 2]),
).play
)
)
```

And hopefully after pasting this into your IDE, you should hear Twinkle Twinkle little star played at the very sedate pace of 60bpm. If you're thinking this is pretty verbose and annoying way to enter music, then let's fix that.

First of all note in the score above that the rhythm is a repeating motif of 6 quarter notes, followed by a half note. So now we can change the above to:

```
// Twinkle Twinkle Little Star
(
Pbind(
  \midinote, Pseq([60, 60, 67, 67, 69, 69, 67, 65, 65, 64, 64, 62, 62, 60,67, 67, 65, 65, 64, 64, 62,67, 67, 65, 65, 64, 64, 62,60, 60, 67, 67, 69, 69, 67, 65, 65, 64, 64, 62, 62, 60]),
  \dur, Pseq([1, 1, 1, 1, 1, 1, 2], 6)
).play
)
)
```

So by passing in a second value to Pseq (separated from the array by a ```,```) we were able to tell SuperCollider to play this sequence 6 times in a row. We can't repeat the `dur` trick, but fortunately we have options available to us:

```
(
var a = [60, 60, 67, 67, 69, 69, 67, 65, 65, 64, 64, 62, 62, 60];
var b = [67, 67, 65, 65, 64, 64, 62];

Pbind(
  \midinote, Pseq(a ++ b ++ b ++ a),
  \dur, Pseq([1, 1, 1, 1, 1, 1, 2], 6),
).play
)
```

So what was that? The first two lines create two new `variables` called `a` and `b` respectively and assign arrays to those labels. [Definition of an array to go here]. Then in Pseq we concatenate those variables using `++` to create one large array:

```
(
var a = [1,2,3];
var b = [4,5,6];

a.postln;
b.postln;
a ++ b;
)
```

[TODO Previous chapter - explain parentheses]

In the post window you should see:

```
[ 1, 2, 3 ]
[ 4, 5, 6 ]
-> [ 1, 2, 3, 4, 5, 6 ]
```

```a ++ b``` creates a new array which combines `a` and `b`. So above we create a new sequence of note values that consisted of `a` followed by `b` twice followed by `a` again. There is another way to achieve this, and when working with ```Pbind``` this is generally preferred. We can use ```Pseq``` to chain several sequences together. So the above would be rewritten:

```
(
var a = [60, 60, 67, 67, 69, 69, 67, 65, 65, 64, 64, 62, 62, 60];
var b = [67, 67, 65, 65, 64, 64, 62];

Pbind(
  \midinote, Pseq([Pseq(a), Pseq(b, 2), Pseq(a)]),
  \dur, Pseq([1, 1, 1, 1, 1, 1, 2], 6),
).play
)
```

Or if you prefer, you can use ```++``` to combine the patterns:

```
(
var a = [60, 60, 67, 67, 69, 69, 67, 65, 65, 64, 64, 62, 62, 60];
var b = [67, 67, 65, 65, 64, 64, 62];

Pbind(
  \midinote, Pseq(a) ++ Pseq(b, 2) ++ Pseq(a),
  \dur, Pseq([1, 1, 1, 1, 1, 1, 2], 6),
).play
)
```

The last two pieces of code are interchangable - the way you write this comes down to personal preference (I personally prefer the latter). SuperCollider will convert the second piece of code into the the former for you.

Okay, so this is definitely better, but you probably don't know all the midinote numbers. Can we do better? Yes we can:

```
(
var a = Pseq([0, 0, 4, 4, 5, 5, 4, 3, 3, 2, 2, 1, 1, 0]);
var b = Pseq([4, 4, 3, 3, 2, 2, 1]);

Pbind(
  \scale, Scale.major,
  \root, 0,
  \octave, 5,
  \degree, a ++ b ++ b ++ a,
  \dur, Pseq([1, 1, 1, 1, 1, 1, 2], 6),
).play
)
```

So this time the numbers in our array correspond to notes in the C scale in the middle octave on the piano (the octave that begins with middle C, or C4), with `0` for `C4`, `1`, for `D` all the way to `8` for `C5`. So let's explain the new keys in this `Pbind`:

+ **scale** - this Pbind uses the major scale. SuperCollider has pretty much any scale you can imagine. If you don't set this value then the default value is Scale.major.
+ **root** - The key for this scale. 0 is C, 1 is C#, etc. The default is `0`.
+ **octave** - The default octave. 5 is confusing used for the octave that starts with `C4`. If you don't set the octave key, then `5` is the default.
+ **degree** - The degree of the current scale, where 0 is the root note of the scale in the current octave. In the above example if wanted to play `C3` then we would use the value `-8` while C4 would be `8`. We can also play sharps and flats. C# would be 0.1 - while Db is 0.9. C## (also known as D) would be 0.2. 

So transposition is pretty easy. By key to G:

```
(
var a = Pseq([0, 0, 4, 4, 5, 5, 4, 3, 3, 2, 2, 1, 1, 0]);
var b = Pseq([4, 4, 3, 3, 2, 2, 1]);

Pbind(
  \scale, Scale.major,
  \root, 7,
  \octave, 5,
  \degree, a ++ b ++ b ++ a,
  \dur, Pseq([1, 1, 1, 1, 1, 1, 2], 6),
).play
)
```

So in the example above `0` means the value G4, as we're starting 7 intervals (or 7 notes in the chromatic scale) above C4. The bass point that we're using is up the keyboard from middle C.

or to A minor (note that we went down to the A3 before C4):


```
(
var a = Pseq([0, 0, 4, 4, 5, 5, 4, 3, 3, 2, 2, 1, 1, 0]);
var b = Pseq([4, 4, 3, 3, 2, 2, 1]);

Pbind(
  \scale, Scale.minor,
  \root, -3,
  \octave, 4,
  \degree, a ++ b ++ b ++ a,
  \dur, Pseq([1, 1, 1, 1, 1, 1, 2], 6),
).play
)
```

We can even transpose it to a different mode, such as F Lydian, by using ```mtranspose```:
```
(
var a = Pseq([0, 0, 4, 4, 5, 5, 4, 3, 3, 2, 2, 1, 1, 0]);
var b = Pseq([4, 4, 3, 3, 2, 2, 1]);

Pbind(
  \scale, Scale.major,
  \root, 0,
  \octave, 5,
  \degree, a ++ b ++ b ++ a,
  \dur, Pseq([1, 1, 1, 1, 1, 1, 2], 6),
).play
)
```

Note that in the above example the scale and root are very important. As `F Lydian` is built upon the 4th degree of the C major scale, then we need to pass in a major scale and a root of `0` so that we're using the C major scale. C lydian built upon `C4` would require a root of `F` and the octave value of `4` and would look like this:

```
(
var a = Pseq([0, 0, 4, 4, 5, 5, 4, 3, 3, 2, 2, 1, 1, 0]);
var b = Pseq([4, 4, 3, 3, 2, 2, 1]);

Pbind(
  \scale, Scale.major,
  \root, 5,
  \octave, 4,
  \degree, a ++ b ++ b ++ a,
  \dur, Pseq([1, 1, 1, 1, 1, 1, 2], 6),
).play
)
```

See if you can reassure yourself that this is correct, and if you're interested in modal music try playing around with some other values.

If you don't understand what a mode is, don't worry about it. If you're interested, then this wikipedia article is a good place to start.

**Important** Everything written above assumes that you are working in equal temperment and using western scales. SuperCollider will happily support microtuning, Indian scales or anything else you can think of. Don't worry this will be explained in later chapters. For now you must remain shackled within conventional western notions of musicality...
