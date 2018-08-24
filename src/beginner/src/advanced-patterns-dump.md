Pattern Stuff to reuse somewhere:

So let's unpack this line a little bit. First of all: ```[60, 62, 64, 65, 67, 69, 71, 72]``` is an array, which is a special form of 'data' that contains a sequential collection of elements. The line containing ```Pseq``` creates an 'object' that allows us to send this sequence of notes to the SuperCollider sound server. So let's find a little bit more about Pseq. Copy and paste the Pseq code to another line:

```Pseq([60, 62, 64, 65, 67, 69, 71, 72])```

position your cursor on the same line and hit ```cmd/ctrl-enter```. In the 'Post Window' you should see:

```-> a Pseq```

So when we executed this line we create a new ```Pseq``` object. Hit ```cmd/ctrl-enter``` a few more times and you'll see the same thing. Each time we do this, we creating a new `Pseq` object that contains the array ```[60, 62, 64, 65, 67, 69, 71, 72]```. But it's not doing anything useful. So let's change that. Execute each line below separately (by going onto the line with the cursor and typing ctrl/cmd-enter):

```
a = Pseq([60, 62, 64, 65, 67, 69, 71, 72]);

b = a.asStream;

b.next;
b.next;
b.next;

c.next; 
c.next;
```

Hopefully you should see the following output in the Post Window:

```
-> a Pseq
-> a Routine
-> a Routine
-> 60
-> 62
-> 64
-> 60
-> 62
```

So let's look at each line in turn:

1. This line create a Pseq object and assigns it to the variable 'a'. [TODO define variable in the previous chapter].
2. This line calls the method ```asStream``` on the ```Pseq``` object that we assigned to variable ```a``` and the result from calling this method is assigned to variable ```b```.


Let's unpack this a little bit. ```Pseq``` constructs a `Pseq` object (don't worry about this terminology for the moment) and takes two 'parameters'

So what just happened? First of all we surrounded the piece of code that we wanted to play