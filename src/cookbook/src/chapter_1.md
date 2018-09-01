#Chapter 1 Debugging techniques 
In a perfect world every line of code you type in immediately works as expected.  If you are like
most people, however, you will make tons of mistakes. SuperCollider does not have a built-in
debugger. So how do we go about debugging problems? We'll go over some ways to debug your
SuperCollider programs.

## Printing techniques 
### Printing content of variables 
SuperCollider offers a few method calls to post information to the post window.  The most common
used ones are `postln`, `postcs` and `debug`.

* `postln` prints the value of the variable in the post window and automatically adds a newline.
  `post` does the same but without adding a newline
* `postcs` prints the value in the form of a compile string in the post window and also adds a newline
* `debug` takes an argument and uses it as prefix when printing the value

Some examples to make it clear

```supercollider
(
var num = 42;
var txt = "important message";
txt.postln; // important message
num.postln; // 42
txt.postcs; // "important message" (note the additional quotes)
num.postcs; // 42 (note no additional quotes) 
txt.debug("text"); // text: important message
)
```

From the examples you can already understand where `postcs` can be useful: one common type of bug is
using a string where you expect a number (or vice versa).  by using `postcs` the difference is
immediately visible.

### Printing type of a variable
Sometimes you want to see more detailed information about the exact type of a variable.  In that
case you can `postln` its class.

```supercollider
(
var pat = Pbind(\instrument, \default);
var player = pat.play;
pat.class.postln; // Pbind
fork {
    player.class.postln; // EventStreamPlayer
    3.wait;
    player.stop;
}
)
```

### Printing intermediate results in complex expressions
Supercollider forces you to declare your variables at the top of the function definition.
Sometimes this can be annoying if you want to quickly inspect the values of some variables
before some later calculations, because it forces you to rewrite your code.

A cool feature of print methods like `postln`, `postcs` and `debug` is the fact that they return the
object they work on after printing it to the post window.

Suppose we want to debug the following code

```supercollider
(
var one = 1;
var two = "2";
var division = one/two;
var double = division*2;
division.debug("division");
)
```

The division.debug will never be executed, because the initialization of division in the line before
already crashes. If you want to examine the values of `one` and `two` before the division happens, 
you could refactor your code to separate the declaration of division from its initialization.
The more complex the code becomes, the more work this takes.

```supercollider
(
var one = 1;
var two = "2";
var division;
var double;
one.postcs;
two.postcs;
division = one/two;
division.debug("division");
double = division*2;
)
```

Because the post and debug family of calls return the object they operate on, you can often avoid rewriting
your code just for debugging purposes.

```supercollider
(
var one = 1;
var two = "2";
var division = one.postcs/two.postcs;
var double = division*2;
division.debug("division");
)
```

_Spoiler: The problem in the code is that you're trying to divide a number by a string._

### Concatenating messages
A final family of techniques that can be useful to know in the context of debugging are some different approaches 
to concatenating strings to form a longer, more informative line of text.

* The operator `++` concatenates representations of objects without space. To get a string as result of the concatenation,
  the left hand side of a ++ expression must be a string.
* The operator `+` concatenates representations of objects and automatically adds a space in between. Either left hand
  side or right hand side has to be a string.

```supercollider
(
var t1 = "txt";
var t2 = "another one";
var n1 = 2.0;
var n2 = 3.0;
(t1 ++ t2).postcs; // "txtanother one"
(t1 + n1).postcs; // "txt 2.0"
//(n1 ++ t1).postcs; // ERROR: Message '++' not understood
(n1 + t1).postcs; // "2.0 txt"
(n1 + n2).postcs; // 5.0
("" ++ n1 + n2).postcs; // "2.0 3.0"
)
```

Notice the use of brackets. If you forget them, as in `t1 ++ t2.postcs` you only print the value of
t2.

Another approach to make longer strings is by using a call to `format`. The following is an example:

```supercollider
(
var n1=2;
var n2=3;
"if you add % to % you get %".format(n1, n2, n1+n2).postln;
)
```

In these strings, the percentage character (%) is used as a placeholder, to be replaced with a
value.  If you need to print a percentage character, use \\\\% instead.

# Debugging patterns
## Printing values produced by patterns
A neat trick to monitor which values your patterns are producing for certain keys is calling
`trace`.  `trace` prints out these values to the post window. Example: why don't the pat1 rhythms
sound as random as those produced by pat2?

Compare the output of the duration pattern in the following two code fragments:

```supercollider
(
s.waitForBoot({
    var pat1 = Pbind(
        \instrument, \default,
        \dur, Pwhite(0,1,inf).trace,
        \midinote, Pbrown(40,70,1,inf)
    );
    ~player = pat1.play;
});
)

// output: 
1 
0 
1 
0 
1 
1 
...
```


versus

```supercollider
(
s.waitForBoot({
    var pat2 = Pbind(
        \instrument, \default,
        \dur, Pwhite(0.0,1.0,inf).trace,
        \midinote, Pbrown(40,70,1,inf)
    );
    ~player = pat2.play;
});
)

// output:
0.01384711265564 
0.56002390384674 
0.79622256755829 
...
```

`trace` can be used on any pattern, so you could just as well write the following to observe all
values produced by the pattern

```supercollider
(
s.waitForBoot({
    var pat1 = Pbind(
        \instrument, \default,
        \dur, Pwhite(0,1,inf),
        \midinote, Pbrown(40,70,1,inf)
    );
    ~player = pat1.trace.play;
});
)

// output:

( 'instrument': default, 'dur': 1, 'midinote': 43 ) 
( 'instrument': default, 'dur': 0, 'midinote': 44 ) 
...
```

## Customized debug messages

If you need more flexibility in how to represent the values, you can add an extra key in the
pattern.  One property of patterns is that every key you add to them is evaluated when it's time to
produce a new event. By using Pfunc, you get access to all the keys specified before the extra key
in the pattern.

In the following example, we add the key \\myfunnykey. Note that this key has no predefined meaning,
but like any other key it is evaluated when it's time to produce a new event. The Pfunc pattern
takes an event as argument. Via this event argument you can access the values of the keys
\\instrument and \\dur  which are specified before \\myfunnykey in the pattern. 

In the example, the value of \\midinote is not yet known when \\myfunnykey is evaluated because it
is specified after \\myfunnykey, and therefore we cannot print it. If you move \\myfunnykey to the
very bottom of the Pbind specification, you'd be able to print the value of \\midinote as well.

```supercollider
(
s.waitForBoot({
    var pat1 = Pbind(
        \instrument, \default,
        \dur, Pwhite(0.0,1.0,inf),
        \myfunnykey, Pfunc({
            | event |
            "The pattern produces a duration of % with instrument %".format(event[\dur], event[\instrument]).postln;
            "midi note is set to % (this should be wrong because midinote is not known yet)".format(event[\midinote]).postln;
        }),
        \midinote, Pbrown(40,70,1,inf)
    );
    ~player = pat1.play;
});
)

//output: 

The pattern produces a duration of 0.782435297966 with instrument default
midi note is set to a Function (this should be wrong because midinote is not known yet)
The pattern produces a duration of 0.40264010429382 with instrument default
midi note is set to a Function (this should be wrong because midinote is not known yet)
...
```

Try to move \\midinote before \\myfunnykey in the previous example and observe the difference in
output.

_Tip: The same mechanism that is used here to produce custom debugging messages can also be used to
build real-time visualizations of patterns while they are playing. Instead of printing messages to
the post window, one then updates elements in a user interface. One example of such a system can be
found on [sccode.org](http://sccode.org/1-56u)_

# Debugging UGens
## Printing values produced by UGens
A low-tech, but useful operation is to print out values produced by some UGen. As UGens typically
run on the server, and printing is something that happens in the language, you might expect we'll
need some mechanism to transfer data from the server to the language. You would be right, but
SuperCollider UGens implement a convencience method `poll` that hides all of this complexity.

Example:

```supercollider
(
s.waitForBoot({

    // specify what a wobble looks like
    SynthDef(\wobble, {
        var out=\out.kr(0), freq=\freq.kr(440), amp=\amp.kr(0.1);
        var env = EnvGen.ar(Env.perc(0.01, 2.0), doneAction:Done.freeSelf);
        var sig = amp*env*SinOsc.ar(3).range(0.5,1)*SinOsc.ar(freq);
        Out.ar(out, sig!2);
    }).add;
    
    // make sure the synth has arrived on the server before continuing
    s.sync;
    
    // play a wobble
    Synth(\wobble); 
});
)
```

Imagine you want to examine in more detail why wobble wobbles. The call to `range(0.5,1)` catches your
attention, and you are curious what values it produces.  By using `poll`, its value will be printed to
the post window at regular intervals:

```supercollider
(
s.waitForBoot({

    // specify what a wobble looks like
    SynthDef(\wobble, {
        var out = \out.kr(0), freq = \freq.kr(440), amp = \amp.kr(0.1);
        var env = EnvGen.ar(Env.perc(0.01, 2.0), doneAction:Done.freeSelf);
        var sig = amp*env*SinOsc.ar(3).range(0.5,1).poll*SinOsc.ar(freq);
        Out.ar(out, sig!2);
    }).add;
    
    // make sure the synth has arrived on the server before continuing
    s.sync;
    
    // play a wobble
    Synth(\wobble); 
});
)

// output:
UGen(MulAdd): 0.750098
UGen(MulAdd): 0.987736
UGen(MulAdd): 0.602984
UGen(MulAdd): 0.603118
```

Note that as written above, you only see the output of `SinOsc.ar(3).range(0.5,1)` but not of other
stuff surrounding it.  If you want to see the output of - say - `amp*env*SinOsc.ar(3)*range(0.5,1)`
instead, you could slightly restructure your code as in the following example.

```supercollider
(
s.waitForBoot({

    SynthDef(\wobble, {
        var out = \out.kr(0), freq = \freq.kr(440), amp = \amp.kr(0.1);
        var env = EnvGen.ar(Env.perc(0.01, 2.0), doneAction:Done.freeSelf);
        var partialsignal = amp*env*SinOsc.ar(3).range(0.5,1);
        var sig = partialsignal.poll*SinOsc.ar(freq);
        Out.ar(out, sig!2);
    }).add;

    s.sync;

    Synth(\wobble);
});
)

// output:
UGen(BinaryOpUGen): 0.000635417
UGen(BinaryOpUGen): 0.082192
UGen(BinaryOpUGen): 0.0408766
UGen(BinaryOpUGen): 0.0332704
```

Calling `poll` will print new values with a predefined frequency. If you want more flexibility you
can pass in some arguments. In the following example, the values are printed 30 times per second
(everytime the first argument produces a pulse) with a prefix "my funky msg".

```supercollider
(
s.waitForBoot({
    SynthDef(\wobble, {
        var out = \out.kr(0), freq = \freq.kr(440), amp = \amp.kr(0.1);
        var env = EnvGen.ar(Env.perc(0.01, 2.0), doneAction:Done.freeSelf);
        var partialsignal = SinOsc.ar(3).range(0.5,1);
        var sig = amp*env*partialsignal.poll(Impulse.ar(30), "my funky msg")*SinOsc.ar(freq);
        Out.ar(out, sig!2);
    }).add;
    
    s.sync;
    
    Synth(\wobble);
});
)

// output:
my funky msg: 0.750098
my funky msg: 0.897103
my funky msg: 0.987823
my funky msg: 0.987705
```

_Tip: Other ways exist to send data from the server to the language, which allow greater flexibility
in what to do with it (like building custom graphical visualizations) but they would lead us a bit
too far in the context of debugging. If you can't wait to learn about these other ways to
communicate data from server to language, you can research some [examples from a course given by
Stelios Manousakis](http://modularbrains.net/dx490a/DX490A\_su2010\_02.1\_[Server-language\_communication].html).
One of these alternative mechanisms sends data over a bus. This also happens to be the mechanism
underlying the next topic._

## Visualizing UGen output with a FreqScope
FreqScope is another way to observe output created by a UGen. It monitors data posted on a Bus and
visualizes it on an oscilloscope. Because it looks at a bus, it remains completely decoupled from
the UGen that produces the data.

In the following example, the \sine UGen posts data on output bus 0 and FreqScope observes output
bus 0.

```supercollider
(
s.waitForBoot({
    SynthDef(\sine, {
        var out = \out.kr(0), freq = \freq.kr(440), amp = \amp.kr(0.1);
        var env = EnvGen.ar(Env.perc(0.01, 5.0), doneAction:Done.freeSelf);
        var partialsignal = SinOsc.ar(3).range(0.5,1);
        var sig = amp*env*partialsignal.poll(Impulse.ar(30), "my funky msg")*SinOsc.ar(freq);
        Out.ar(out, sig!2);
    }).add;

    s.sync;

    // create a new analyzer
    FreqScope.new(400, 200, 0, server: s);

    // play sine
    Synth(\sine).play;
});
)
```

