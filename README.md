There are many methods for triggering audio in Unity. Which one you
choose depends on what you’re trying to achieve.

For example, you can do a lot with only the basic Play function, which
plays an Audio Source as soon as the method is called. Want to delay
playback by a few seconds? That’s easy too, using PlayDelayed.

With these functions, and some conditional logic, it’s easy to create
simple music and audio events that will serve many purposes.

But what if you want to do more?

For example what if you want to queue different Audio Clips to play back
to back seamlessly? Or if you want to build a dynamic music system.
Maybe you want to trigger events in musical time, beat matching sounds
to a precise musical structure.

To do any of these requires very accurate audio timing that can’t be
achieved with Play or PlayDelayed.

Luckily, however, Unity provides a very reliable and accurate method for
scheduling audio precisely, using PlayScheduled and Audio DSP Time.

In this article I explain everything you need to know to get the most
out of PlayScheduled; When you should use it over other methods, how to
start stop and pause scheduled audio as well as some best practice
techniques that I’ve learned from implementing audio scheduling in my
clients’ games.

What you’ll find in this article
--------------------------------

1.  [A bit about me and why I wrote this
    article](https://gamedevbeginner.com/ultimate-guide-to-playscheduled-in-unity/#about_this_post)
2.  [When you should use PlayScheduled (and when you don’t need
    to)](https://gamedevbeginner.com/ultimate-guide-to-playscheduled-in-unity/#play_scheduled_intro)
3.  [How to queue Audio Clips in Unity (How to start and stop scheduled
    clips)](https://gamedevbeginner.com/ultimate-guide-to-playscheduled-in-unity/#how_to_schedule)
4.  [What happens to scheduled Audio Clips when you pause the
    game](https://gamedevbeginner.com/ultimate-guide-to-playscheduled-in-unity/#pausing)
5.  [What you can do with
    PlayScheduled](https://gamedevbeginner.com/ultimate-guide-to-playscheduled-in-unity/#playscheduled_uses)
6.  [How to queue Audio Clips to play back to back
    seamlessly](https://gamedevbeginner.com/ultimate-guide-to-playscheduled-in-unity/#queue_clips)
7.  [How to beat match audio events to a musical
    structure](https://gamedevbeginner.com/ultimate-guide-to-playscheduled-in-unity/#beat_matching)

A bit about me and why I wrote this article {.p1}
-------------------------------------------

I’m a [game composer and sound
designer](https://johnleonardfrench.com/), I do a lot of work in Unity
and, when I’m working with developers, I often help create custom music
and audio systems.

It might be that I build a prototype system to demonstrate an idea or
that I’ll use Unity to test if my own work is going to sound as I
imagined it would when it’s triggered dynamically.

I’ve had the opportunity to build music and audio systems for all kinds
of uses, each of them different in their own way.

All of them, however, required a reliable method of scheduling clips
accurately.

And it seems that I’m not alone…

While researching for this article I found numerous forum posts from
people asking for a way to do the same, how to queue clips back to back
seamlessly, how to prevent pops, clicks and avoid timing errors created
by other methods.

But here’s the thing…

While everything in this guide is either available somewhere online,
detailed in the Unity documentation or can be easily tested in the Unity
editor, the majority of solutions and answers I found in forum posts
were, very often…

Just plain wrong.

Answers were either inaccurate, out of context or were links to an Asset
Store extension that maybe, kind-of, solved the issue but definitely
isn’t necessary.

But don’t worry.

I’ve had a lot of practice putting together music systems, and in the
past I’ve worked through a lot of problems, tried a lot of ideas and
experimented with all kinds of audio systems. Some turned out great and
are, right now, powering the audio in released games. Others, like some
of my early attempts at dynamic audio, were not so great and were a
valuable learning experience.

I’ve used my experience to develop the, now tried and tested, methods
that I use every day with my clients. I wrote this article to show you
how you can do the same, so that you can make better audio for your game
more easily, with less frustration, using proven, best-practice
techniques.

I hope you enjoy it.

Advertisement

When should you use PlayScheduled? {.p1}
----------------------------------

The main reason for using PlayScheduled over other methods of triggering
audio is so that you can schedule audio events using the Audio System’s
DSP Time, which is not just accurate but also operates independently of
frame rate.

The DSP Time value, which you can access in a script with
‘AudioSettings.dspTime’, is based on the actual number of samples
processed by the audio system and returns a double value, which is more
accurate than the float values that are used by PlayDelayed, Time.time
and WaitForSeconds.

![Unity double variable compared to
float](./How%20to%20Queue%20Audio%20Clips%20in%20Unity%20(the%20Ultimate%20Guide%20to%20PlayScheduled)%20-%20Game%20Dev%20Beginner_files/floats-compared.jpg)

![Unity double variable compared to
float](https://gamedevbeginner.com/wp-content/uploads/floats-compared.jpg)

This is what makes it so suitable for precision audio work.

Put simply, if you want to queue audio clips to play back to back
seamlessly (without audible gaps) or if you want to create audio events
that trigger in a musical structure (and not sound out of time) then you
should be using PlayScheduled for the extra accuracy that it provides.

If however, all you want to do is delay a sound, or a music loop, by a
few seconds then, in many cases, PlayDelayed is enough and you should
use that instead.

Advertisement

How to schedule Audio Clips in Unity? {.p1}
-------------------------------------

So how does PlayScheduled work, and what can you do with it?

### Setting a Clip to start {.p5}

Calling PlayScheduled on an Audio Source allows you to schedule it to
play at an exact time in the future. It takes one double value
parameter, which is the time that the Audio Source will play.

It’s important to remember that PlayScheduled takes a time value, not a
delay.

This is different to the PlayDelayed function and the older, now
deprecated, Delay parameter of Play, which both specify a delay before
the Audio Source will trigger.

For example if you wanted to use PlayScheduled to trigger an Audio Clip
with a two second delay, you would need to pass the current Audio DSP
Time plus two seconds:

``` {.prettyprint .prettyprinted style=""}
// Schedule an Audio Source to Play in 2 Seconds.
AudioSource.PlayScheduled(AudioSettings.dspTime + 2);
```

![Diagram: How to play Audio with a 2 second delay in Unity using
Playscheduled](./How%20to%20Queue%20Audio%20Clips%20in%20Unity%20(the%20Ultimate%20Guide%20to%20PlayScheduled)%20-%20Game%20Dev%20Beginner_files/PlayScheduled-Examples-2.jpg)

![Diagram: How to play Audio with a 2 second delay in Unity using
Playscheduled](https://gamedevbeginner.com/wp-content/uploads/PlayScheduled-Examples-2.jpg)

Keeping track of when audio events started is important when calculating
the trigger times for other, related, audio events.

Because of this it’s often a good idea to store the scheduled time in a
double variable when using PlayScheduled, so that you can reference the
time a clip started in later calculations.

``` {.prettyprint .prettyprinted style=""}
// Saves the start time for later use
Double startTime = AudioSettings.dspTime + 2;
```

#### **Scheduling a clip to** play**immediately** (and why** you probably shouldn’t do it)** {.p4}

Although it is possible to call PlayScheduled at the current DSP Time,
which will cause the audio to start as soon as possible, one of the main
benefits of using PlayScheduled is to avoid instantly preparing the
sound for playback.

For this reason, to avoid timing errors when scheduling the first Clip,
I add a slight delay.

``` {.prettyprint .prettyprinted style=""}
// Start the first Clip as soon as possible
audioSource.PlayScheduled(AudioSettings.dspTime + 0.5);
```

### What if you’ve changed your mind? How to reschedule PlayScheduled {.p5}

It’s possible to change the scheduled time of an Audio Source by calling
SetScheduledStartTime in the exact same way as PlayScheduled.

Doing so will reschedule when the Audio Source will trigger,
interrupting it if it’s already playing.

``` {.prettyprint .prettyprinted style=""}
// Change when a scheduled Audio Source will play
audioSource.SetScheduledStartTime(dspTime);
```

It’s worth noting however that this will only work if PlayScheduled has
already been called and, my experience, you can achieve the same result
by simply calling PlayScheduled on the same Audio Source a second time.

### How to cancel PlayScheduled: Stop an Audio Clip that’s already scheduled to play

To cancel PlayScheduled, and stop an Audio Source from playing in the
future, simply call Stop on that Audio Source, just like you would with
any Audio Source that’s already playing.

``` {.prettyprint .prettyprinted style=""}
// Stop a scheduled Audio Source
AudioSource.Stop();
```

This works because, even if a clip on a scheduled Audio Source hasn’t
started yet, the Audio Source itself is considered to be playing. In
fact AudioSource.isPlaying will return true on any Audio Source that’s
scheduled, whether it’s outputting audio or not.

This is important to remember if you’re using any logic that checks
against the isPlaying parameter of the Audio Source as it will return
true even before the scheduled AudioSource begins to play.

### Stopping a clip at a precise time {.p4}

Just as you may want to start a clip at a precise time, you may also
wish to end a clip at an exact time too. For example: if you want to
transition a piece of music into a new Clip at the next bar (more on how
to do that later).

To stop a clip at an exact time call SetScheduledEndTime on the Audio
Source in the same way as PlayScheduled, passing in the time value of
when you want the clip to end.

``` {.prettyprint .prettyprinted style=""}
// Stop a Scheduled Audio Source at an exact time
audioSource.SetScheduledEndTime(dspTime);
```

Advertisement

### What happens to scheduled audio when you pause the game? {.p5}

Using the common method of setting the timescale to zero
(Time.timeScale=0;) to pause the game won’t affect DSP Time, or stop any
Audio Sources from playing regardless of when they were scheduled. Just
like any other Audio Source, they will continue to play while the game
is paused.

This is great if, for example, you want to continue the same music into
your pause menu but is less useful if you’re using an audio system
that’s being driven by events that are dependant on scaled time.

For this reason, it’s a good idea to avoid logic calculations that mix
scaled and unscaled time, for example comparing Time.time to dspTime in
an IF statement.

#### How to pause audio events in Unity with AudioListener.pause

Alternatively, you can freeze the entire audio system when the game is
paused, including freezing DSP Time, by setting AudioListener.pause to
true whenever you pause the game.

``` {.prettyprint .prettyprinted style=""}
// Pause all Audio Sources
AudioListener.pause = true;
```

This pauses every Audio Source, and prevents scheduled audio sources
from playing whenever the game is stopped. They will resume from their
exact position when AudioListener.pause is set back to false and the
game is re-started again. 

In this scenario, you’re probably still going to want some sounds to
play when the game is paused, such as user interface sound effects or
menu background music for your pause menu, and because of this it’s
possible to allow selected Audio Sources to continue playing even when
AudioListener.pause is set.

To do this, set ignoreListenerPause to true on any Audio Source that you
want to be able to use when the game is paused and it will continue to
play as normal.

``` {.prettyprint .prettyprinted style=""}
// Use an Audio Source when the Listener is paused
AudioSource.ignoreListenerPause=true;
```

Even scheduled Audio Sources will continue to work as normal, despite
the fact that the DSP Time value has been stopped.

Please note however that in this specific scenario, while PlayScheduled
will work, DSP Time based logic conditions won’t (because the visible
DSP Time is stopped) so although you can use PlayScheduled while DSP
Time is paused, you won’t be able to reference DSP Time in IF
statements.

Advertisement

What can you do with PlayScheduled? {.p1}
-----------------------------------

Now that you understand the fundamentals of how to schedule audio in
Unity, it’s time to put them to creative use.

### How to queue Audio Clips to play back to back seamlessly {.p5}

One of the main benefits of using PlayScheduled is the ability to stitch
any two, or more, Audio Clips together to play back to back seamlessly.
As soon as one Clip ends, the next one starts, with no gap in between.

This is especially useful for building music systems that use sequential
parts, that you can swap out on the fly, or for adding an intro to a
looping track.

It’s done by recording the start time of the first Audio Source,
calculating the length of the Audio Clip that’s going to play and
scheduling a second Audio Source to play at the exact moment the first
ends:

![Diagram showing how to queue Audio Clips back to back in
Unity](./How%20to%20Queue%20Audio%20Clips%20in%20Unity%20(the%20Ultimate%20Guide%20to%20PlayScheduled)%20-%20Game%20Dev%20Beginner_files/PlayScheduled-Queued.jpg)

![Diagram showing how to queue Audio Clips back to back in
Unity](https://gamedevbeginner.com/wp-content/uploads/PlayScheduled-Queued.jpg)

Here’s how to do it:

#### **First, set up two Audio** Sources {.p4}

It’s not possible to do this with a single Audio Source so you will need
to use two.

This is because you can’t schedule an Audio Source that’s already been
scheduled to play in the future without resetting it

Using two Audio Sources allows you to schedule one while the other is
playing. You can then switch between the two as needed. If you want to
play an endless number of Clips back to back this too can be achieved
with two Audio Sources. Just toggle whichever Audio Source is used next.

A method I often use to switch between Audio Sources in scripting is by
creating an Audio Source Array and using a integer toggle that I can
switch every time a new Clip is scheduled. The toggle is simply an
integer variable that switches between 0 and 1. When you want to flip
the switch, simply set the value to 1 minus itself. I then use the
current value of the toggle as the Array Index to select which Audio
Source to play next.

``` {.prettyprint .prettyprinted style=""}
// Use two Audio Sources in an Array
public AudioSource[] audioSourceArray;
int toggle;

// Whenever you schedule a clip
toggle = 1 - toggle;
audioSourceArray[toggle].PlayScheduled(dspTime);
```

#### How to accurately calculate the length of a Audio Clip (without using AudioClip.length) {.p4}

In order to schedule the next Clip when the current one ends, you will
need to calculate an accurate clip duration.

The AudioClip object includes a value for length but it’s a float value,
so it won’t be accurate enough to use.

Instead, take the samples value of the Clip (AudioClip.samples), which
is the total number of samples in the audio file, and divide it by the
Clip’s frequency (AudioClip.frequency), which is the sample rate (the
number of samples in each second of audio).

The result is a highly accurate duration value which you can then use to
calculate the next Clip’s start time:

``` {.prettyprint .prettyprinted style=""}
// Calculate a Clip’s exact duration
double duration = (double)AudioClip.samples / AudioClip.frequency;
```

Note that in order for this to work you have to cast samples, which is
an integer, as a double when calculating the duration.

Now, to calculate when the next Clip should play, simply add the
calculated duration to the Start Time of the last clip and pass it into
the PlayScheduled function.

``` {.prettyprint .prettyprinted style=""}
// Queue the next Clip to play when the current one ends
audioSourceArray[toggle].PlayScheduled(startTime + duration);
```

#### Creating endless playlists: When to schedule the next Clip {.p4}

Exactly when you run the code that queues up the next clip to play
depends on your use case. For example, if you want to play a music
intro, immediately followed by a looping Clip, then both of these events
can be queued up at the same time.

``` {.prettyprint .prettyprinted style=""}
// Play an intro Clip followed by a loop

AudioSource introAudioSource;
AudioSource loopAudioSource;

void Start () {

double introDuration = (double)introAudioSource.clip.samples / introAudioSource.clip.frequency;
double startTime = AudioSettings.dspTime + 0.2;
introAudioSource.PlayScheduled(startTime);
loopAudioSource.PlayScheduled(startTime + introDuration);

}
```

If, however, you want to endlessly feed in clips that play back to back,
perhaps for a dynamic music system or a never ending playlist, you will
need to define a point in time to evaluate which Clip is going to play
next be and exactly when it’s going to play.

Typically, when I do this with dynamic music systems, I will have the
system look one second ahead until just before the next clip is needed.

To do this, use Update to check when the next Clip is due to finish:

``` {.prettyprint .prettyprinted style=""}
public AudioSource[] audioSourceArray;
public AudioClip[] audioClipArray;

int nextClip;

void Update () {

    if(AudioSettings.dspTime > nextStartTime - 1) {

    AudioClip clipToPlay = audioClipArray[nextClip];

    // Loads the next Clip to play and schedules when it will start
    audioSourceArray[toggle].clip = clipToPlay;
    audioSourceArray[toggle].PlayScheduled(nextStartTime);

    // Checks how long the Clip will last and updates the Next Start Time with a new value
    double duration = (double)clipToPlay.samples / clipToPlay.frequency;
    nextStartTime = nextStartTime + duration;

    // Switches the toggle to use the other Audio Source next
    toggle = 1 - toggle;

    // Increase the clip index number, reset if it runs out of clips
    nextClip = nextClip < audioClipArray.Length - 1 ? nextClip + 1 : 0;
    }
}
```

### Beat matching audio events (how to trigger audio to play on the next bar, beat or musical note) {.p5}

It’s possible to use PlayScheduled and DSP Time to schedule events in
precise musical time. This is great for swapping out audio clips at an
exact bar or beat, or triggering audio events to play over another track
in musical time.

In the example below, I’ll show you how you can use beat matching to end
a looping track on the next bar.

#### How to calculate the length of a note, beat or bar {.p4}

To calculate musical time, you first need to know exactly how long a
beat is.

To do this, you will need to know the tempo and, if you’re calculating
bar length, you will need to know the time signature too.

#### Find the track’s tempo

You may already know the tempo of your track but if you don’t, try a
beat finder tool like
[beatsperminuteonline.com](http://www.beatsperminuteonline.com/) to work
it out.

Once you know what it is, divide a double value of 60 seconds by the
tempo to get the duration of one beat:

``` {.prettyprint .prettyprinted style=""}
// To calculate the beat length of a 95 bpm track
double beatLength = 60d / 95
```

Note the ‘d’ after 60, which specifies it as a double. Without it, it
would be treated as an integer, breaking the calculation.

You can use the beat length to calculate other musical unit lengths.

For example: Multiply the beat length by the number of beats in a bar to
get a bar length (Not sure how many beats are in each bar? See the next
section for more about time signatures).

Divide the beat length by four to get the length of a 16th note (a
semiquaver):

``` {.prettyprint .prettyprinted style=""}
// To calculate the semiquaver note length of a 110 bpm track in 4/4: 
double noteLength = (60d / 110) / 4;
```

 …and so on.

#### Finding the length of a bar

A time signature consists of two numbers. It will tell you first how
many beats are in a bar and, second, how long each beat is.

For example a typical time signature is 4/4 meaning four beats to a bar
that are each a quarter note (crotchet) in length.

#### The first number of the time signature shows how many beats are in a bar

You’ll usually be able to count the number of beats in each bar just by
listening to it. Very often this value will be either 4 beats in a bar,
or 3 (e.g. 4/4 or 3/4).

``` {.prettyprint .prettyprinted style=""}
// To calculate the bar length of a 100bpm track in 4/4
double barLength = 60d / 100 * 4

// To calculate the bar length of a 70bpm track in 3/4 
double barLength = 60d / 70 * 3
```

#### The second number in a time signature shows you how long the beat is. {.p2}

The second number in the time signature is, very often, four, which
means a quarter note length for each beat (or a crotchet in the UK).

Sometimes other note lengths are used and, in some pieces of music, the
time signature will change throughout the track.

However…

Unless you already know what it is, it’s not possible to work out the
second value of the time signature is just by listening to the music.

This is because a difference in tempo can also be interpreted as a
difference in time signature. For example: 4/4 at 100bpm will
technically sound the same as 4/8 at 50bpm.

With that in mind, the following only applies if you know what the time
signature is: For example, you wrote the music and, even then, you
probably won’t need to modify your calculation.

If you do know what it is, for the purpose of the calculation explained
here, if the second number is a four, then there’s nothing more to do.
If, however, it’s any other value, for example: 4/8, then you can modify
the bar length using this method:

Divide the first number in the time signature by the second to give you
a modifier value. For example, if the time signature is 4/8:

4 / 8 = a modifier of 0.5

Multiply the bar length by this modifier to adjust for a time signature
other than 4/4.

``` {.prettyprint .prettyprinted style=""}
// To calculate the bar length of a 100bpm track in 4/8:
double barLength = (60d / 100 * 4) * (4/8);
```

### Calculating when the next bar will occur {.p4}

Once you know the duration of a note, beat or in this case a bar, you
can use that value to calculate when the next musical unit will occur.

Returning to our example, say that you’re playing back a combat music
loop and you want to transition the track to an ending cue at the next
bar because all of the enemies are dead.

You can use the current position in the Audio Clip, and the bar duration
that was just calculated, to work out when the next bar will occur.

This is done using the Modulo Operation, which is a computing function
that returns the remainder, after division, of one number by another.

In C Sharp, it’s represented by the percent symbol %, and it works like
this:

5 % 2 = 1

Why does it return 1? Because 2 goes into 5 twice (a total of 4),
leaving 1 left over.

Some more examples:

25 % 7 = 4

18 % 5 = 3

10 % 2 = 0

#### How to use the Modulo Operation to calculate musical time: {.p4}

Using the Modulo Operation to divide the time elapsed so far by the
duration of the musical unit you want to snap to will return a remainder
value.

That value represents the progress through the current, incomplete, bar,
beat or note at the time of the calculation.

Subtracting the remainder value from a full bar (or note, or beat –
whatever you’re using) will return the time left in seconds until the
next one.

This allows you to schedule an audio event at that moment, using
PlayScheduled.

Here’s what it looks like:

![Beat matching audio to music in Unity using
Modulo](data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7)

![Beat matching audio to music in Unity using
Modulo](https://gamedevbeginner.com/wp-content/uploads/PlayScheduled-Modulo2.jpg)

And here’s how it works:

#### Get the time elapsed in the Audio Clip so far {.p4}

To calculate when the next bar will occur, first you need to find the
elapsed time of the current Clip. There are two ways to do this:

-   Take the DSP Time that the clip started and subtract it from the
    current DSP Time. 
-   Divide the AudioSource sample position by the clip’s sample rate.

``` {.prettyprint .prettyprinted style=""}
// Get the current Time Elapsed
double timeElapsed = AudioSettings.dspTime - startTime;
// Or
double timeElapsed = (double)AudioSource.timeSamples / AudioClip.Frequency;
```

Note that timeSamples is an integer, so I need to cast it as a double
before division.

In this example I’m going to use the second method, using the Clip’s
sample position to calculate the time elapsed.

#### Get the time elapsed through the current bar

Next, divide the time elapsed by the bar duration using the Modulo
operation to return a remainder:

``` {.prettyprint .prettyprinted style=""}
// Use the Modulo Operation to get the time Elapsed in the current bar
double remainder = timeElapsed % barDuration;
```

This will leave a time value that represents the time elapsed of only
the current bar.

#### Find out how long before the next bar

Subtract this value from the length of a full bar to get the time
remaining in the current bar.

``` {.prettyprint .prettyprinted style=""}
// Calculate time remaining in the current bar
double timeToNextBar = barLength - remainder;
```

#### Calculate a value you can use in PlayScheduled

Add this to the current DSP time to get the absolute time value of the
next bar.

``` {.prettyprint .prettyprinted style=""}
// Get the time value of the next bar
double nextTime = AudioSettings.dspTime + timeToNextBar;
```

When you put it all together, here’s how it looks in scripting:

``` {.prettyprint .prettyprinted style=""}
// Calculate the duration of a bar that's 80bpm in 4/4
double barDuration = 60d / 80 * 4;

// This line works out how far you are through the current bar
double remainder = ((double)audioSource.timeSamples / audioSource.clip.frequency) % (barDuration);

// This line works out when the next bar will occur
double nextBarTime = AudioSettings.dspTime + barDuration - remainder;

// Set the current Clip to end on the next bar
audioSource.SetScheduledEndTime(nextBarTime);

// Schedule an ending clip to start on the next bar
endCueAudioSource.PlayScheduled(nextBarTime);
```

#### Other uses for beat matching: Musically timed hits {.p4}

Another use for this method is to create audio triggers that are tied to
game events that can snap to the next musical beat. An example of this
could be creating a drum hit, that sounds every time you kill an enemy
in the game that triggers on the next note, so that it sounds like it’s
part of the music.

