For an HTML version check out the [Manual](http://mrbmueller.github.io/MidGen.html) under [Software Projects](http://mrbmueller.github.io/index.html).



[TOC]

# MidGen

MidGen is a Perl based standard midi file reader/processor/generator. It includes Perl modules and packages to create/read/write and process standard midi (smf) files based on Perl input scripts. Essentially this tool provides a framework, which is specifically designed for experiments with musical pattern, arpeggios, accompaniment styles, arrangements or entire scores. A build-in "micro sequencer" function translates text-based micro-sequences into smf midi data. Since everything runs in perl environment, you can take advantage from programming languages like using variables for micro-sequences, storing arrangement parts in sub-procedures, repeat sequence iterations with loops, etc. together with convenient smf input/output functionality. This approach allows writing music in traditional sequenced format in combination with algorithmic or procedural programming functionalities. In addition, there are many functions for SMF data manipulation implemented such as copy/paste/insert/transpose which are working either on track level or across the entire arrangement. Also you can include and merge additional external smf midi data into your arrangement so that for instance live played parts get merged with generated accompaniment pattern. Essentially there is no limit for creativity since its an open framework were you can easily add additional features, functions, procedures or other extensions with your own ideas.



Example smf output arrangement in score representation:

<img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/img3.png width="100%">



The score was entirely generated from the example source code below using micro sequences:

<img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/img1.png width="100%">



To compile the midi output file, just run:

<img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/img2.png width="100%">



The output is typically a type 1 smf midi file saved in your current working directory from where the script gets executed. In addition, the console output provides an overview and displays general song- and track-information of your arrangement. For more detailed result debug, individual event lists per track are written as regular text files along with the smf output.



------

## usage

Once you have perl installed, just run:

    perl MidGen.pl <project.pl>

If there is no project-file specified, the default project (Projects\Current.pl) will get processed.

To get started quickly and to see how the output looks like, you can also just process a smf midi file thru the program or try one of the provided project examples. MidGen was tested with ActiveState Perl, Strawberry Perl portable and various Linux/Ubuntu Perl versions.

Example: read/write a smf midi file:

<img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/img0.png width="100%">

------

## internal smf data structure

Internally the smf is nothing else than a multidimensional hash structure with integer key values across all hierarchy levels. The top level key represents the track number (except "track" -1 which is used for smf specific information), the 2nd level key represent the eventtime in ticks based on the smf PPQ setting and the 3rd level key is the event ID representing the event order within a given tick. This allows simple access and iterations across all smf events for easy data manipulation.

Reading a midi file is as simple as:

```perl
my %smf = MIDI::Read("filename.mid");
```

**timestamps and durations**

Timestamps and note- or continuous controller-durations for functions such as (insert, micro-sequencer, copy, etc.) are typically provided in whole notes as floating point values to decouple them from timesignatures. Internally all times are stored in ticks based on the smf PPQ setting. Therefore its important to define the required resolution in the beginning of a project.

**supported event types**

MidGen supports all types of midi events including channel-, SysEx-, Meta- and Escape-messages. NoteOn-Off pairing is done internally while reading smf files. That means all internal operations refer to Note- rather than Note-On/Off events unless they are inserted individually on purpose. Therefore each note is represented as a single event with a given duration and a optional note-off velocity.

------

## Micro-Sequencer

The micro-sequencer is a central and important function especially written for convenient note- , controller- and other data insertions. Essentially it inserts a sequence of notes and/or controller/meta/sysex events into a specified track, based on a given start-time, scale (e.g. chromatic, major, minor, etc.) and base note (e.g. 60 - middle C). The sequence is provided as a consecutive, white space separated list of `<duration>:<data>` pairs representing duration timestamps with associated event data in text format. The micro-sequencer keeps internally track of timestamps, note-pitches and other data and typically advances in time with each provided event-duration value unless otherwise specified (e.g. for overlap notes or chords). In addition, the sequencer returns the total duration of the provided micro-sequence in whole note units. This allows the caller thread keeping track about the total timestamp if you simply sum up all micro-sequence durations in case they belong logically together. The micro sequencer function is called from Edit package as follow:

    <duration> = Edit::Seq(<smf-hash-pointer>, <track>, <start-time>, <base-note>, <scale>, <sequence>);

Actually there are much more arguments available, but those are the most important ones to start with.

 - `<smf-hash-pointer>` is simply a pointer into your smf midi data structure where the sequence get inserted. Typically you'll use the standard smf output structure `%main::out`, but in case you work on multiple smf structures in parallel, you need to specify which one to use.
 - `<Track>` is almost self-explanatory. It simply specifies the tracknumber where the sequence goes into. Track numbers start from zero.
 - `<start-time>` is basically the timestamp in whole note units where the sequence gets inserted.
 - `<base-note>` determines the sequence reference note `0` as the sequencer works relative from there. In conjunction with the scale it provides finally the sequence key.
 - `<scale>` 0:chromatic (typically used in combination with base-note zero to access regular midi note numbers - e.g. for percussion sequences); 2:major; 3:minor
 - `<sequence>` is a text string argument representing the actual sequence to insert

Example - simple micro sequence:

<img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/Example3.png style="zoom:100%;"  >

The sequence above is mostly self explanatory.

### Timestamps and Durations
Timestamps and event-durations are typically provided in whole note units either as integer or floating point numbers or as equations such as `1/4` or `1/4+1/8`, `1/4+1/8+1/16` etc. To shortcut dotted note equations, you can just append tailing `+` signs to the duration value. For instance a dotted quarter can be written either as `1/4+1/8` or alternatively as `1/4+`. Double or triple dotted notes are written respectively with tailing `++` or `+++` signs. Similarly you can shorten a note by its half length with tailing `-` signs. The example below shows few different variants of duration timestamps.

<img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/img4.png width="100%"  >

> ***Note: If there is no timestamp provided, the sequencer automatically applies the latest valid timestamp value.***

#### Duration alignment
To avoid complex arithmetic equations and to keep sequences more readable, it is possible to align event durations to given timestamp grid boundaries by using the alignment operator `|` in front of the value. This advices the sequencer to proceed in time until the next grid boundary timestamp is reached. So in the example above, the `|1/1:%` event will simply insert a rest in alignment with the next whole note timestamp value. In this case it proceeds to the next bar since the time signature is 4/4 (whole note grid boundary).

#### Visual beat/bar separator
In order to keep the sequence more structured and readable, it is possible to insert additional `|` character symbols to visualize bar, beat or other timestamp separations. Semantically they dont have any meaning and are just ignored like comments or remarks by the sequencer as long as they stay in clear separation from events.

<p align="center"> <img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/img5.png style="zoom:90%;"  > </p>

### Events and event types

The microsequencer supports different smf event types such as

- note events
- controller events including after touch, sysex, program change and tempo events either as single event or as continous controller series
- marker and lyric text events
- time- and key-signature changes
- pause/rest/NOP "events"

Events are always paired with duration timestamps even though some event types do not really require a duration such as marker or lyric events. In this case a zero duration can be used if the sequencer should not advance in time. Events can also get paired to insert multiple event types at the same time with the same duration in one sequencer event. For instance you can insert a note and in parallel a continous controller using the same timestamp and duration.

#### Note events

##### absolute note events

Note events are typically provided as numerical values rather than traditional musical symbols. This decouples them from scales and tonal systems and keeps the flexibility for additional arithmetic operations. They can get specified either in absolut- or relative (interval) values by preceding `^` (up) or `v` (down) symbols. Absolute values can be either positive or negative in case they refer to notes below the given basenote. Notes will follow the given scale and key provided by micro sequencer arguments `<scale>` and `<basenote>`. In result, note zero is basically the basenote.

Example - Two consecutive micro sequences with additional marker `M<text>` events and bar separators:

<img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/Example1.png style="zoom: 80%;" >

The example above just demonstrates how multiple micro sequences can get concatenated using a global timestamp variable.

##### relative note events
Since the micro sequencer keeps internally track about note values, it is possible to use relative note intervals in addition to absolute ones. Relative notes are determined by preceding up `^` or down `v` symbols in front of the note values. If there is no note number specified, the sequencer assumes just one relative note step.

<img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/img7.png style="zoom:80%;"   >


##### Flat (b) and Sharp (#) symbols
Note values can be increased or decreased in semitones by putting either flat and/or sharp symbols after the note value. Traditional music puts them in front of the note, but here it is required to put them after the note value as additional attributes. The sequencer allows having multiple consecutive and/or mixed symbols by simply summing up all flats and sharps to get the final value.

Example - C major with few additional flats and sharps:

<img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/img6.png style="zoom: 80%;"  >


#### Rests
rests and pauses are simply written by the `%` character symbol.

<img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/Example2.png  >

### repetitions, sub-sequences and loops

#### repeat event operator

To repeat single events or rests with the same duration and note value, you can just use the `.` (dot) symbol.

<img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/img13.png style="zoom: 80%;"      >

#### continue event operator

The continue event operator `>` is similar to regular repeats, but ignores rest events and simply 'continues' playing with the latest note value. This event type is mainly used in combination with pattern sequence templates.

<img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/img15.png style="zoom: 80%;"      >

#### subsequence repetitions

In addition, the micro-sequencer comes with several build-in loop and sub-sequence functions for sequence repetitions. Essentially you can enclose sequences or parts of sequences within parentheses or braces `n{<sub-sequence>}` and put a repetition number `n` in front of them. If no repetition number is given, the sequencer assumes one insertion w/o repetition, otherwise the sequencer will insert the enclosed sub-sequence(s) n times. Since the sequencer allows the insertion of recursively nested sub-sequences, repetition pattern can quickly get large and complex by just a few instructions. The example below shows a small input sequence with two nested sub-sequences.

<img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/img9.png style="zoom: 67%;"    >


Finally, different types of parentheses have different meanings:

 - curly `n{<sub-sequence>}` - normal repeat
 - regular `n(<sub-sequence>)` - preserve/restore duration and note values when a sub-sequence gets entered, but advance in time
 - square `n[<sub-sequence>]` - preserve/restore duration/note and timestamp values when a sub-sequence gets entered (doesnt advance in time)

Since the sequencer keeps track of timestamps, durations and notes, you can preserve the previous duration/note and timestamp values when a sub-sequence get entered. In result, the sequencer can restore those values when returning to the main-sequence. This is especially important when building pattern templates using relative note values.

### additional timestamp directives
Usually the sequencer takes care about timestamp values and advances in time with each provided event duration. This function is typically usefull for monotonic lines like lead melodies, bass lines, etc. with non-overlapping notes. However in many cases you need note or event overlaps such as in chords or many other situations. Therefore the sequencer provides additional timestamp directives by having additional `<` symbols in front and/or after the given timestamp value.

<img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/img10.png style="zoom: 80%;"  >

The examples above show 4 different scenarios:

The 1st one is a regular insert of a quarter note where the sequencer advances in time with the event duration. Therefore the total sequence length is effectively one quarter note.

The 2nd scenario shows that the return marker merges with the entry marker because the sequencer doesnt advance in time when the quarter note gets inserted. In result, the total sequence length is effectively zero although a note was inserted.

The 3rd scenario demonstrates how the sequencer goes back in time before the note gets inserted. Therefore the note appears earlier in time as the sequence entry point. Since the sequencer still advances in time with the inserted event, in result the return marker merges again with the entry marker and the total effective sequence length is zero.

The 4th scenario demonstrates the combination of both previous scenarios. The sequencer goes back in time before the event gets inserted, advances in time with the event and finally goes back in time again after the event insertion. In result, the note and the return marker appear earlier in time than the entry marker. Effectively the sequencer is running backward and returning a negative sequence length in this case.

Finally timestamp directives can be used to write overlap notes, chords or sub-sequence overhangs at the beginning or ending of an sequence. The example below shows a simple chord triad with 3 stacked notes. In this case only the last note get advanced in time while the 1st and 2nd notes doesnt advance. If there is no timestamp value provided, the given timestamp directive is still valid and the sequencer will not advance after the event insertion.

<img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/img11.png style="zoom:80%;"  >

**micro-sequence overlaps (overhang)**
Another example below shows how to setup sequence overlaps or overhangs using timestamp directives. In some cases it is required having extra events or overlap events either before the sequence actually starts or after the sequence has been finished. This can easily get achived by timestamp directives going back in time before the sequence starts or after the sequence has been finished. The example below shows two concatenated micro-sequences with additional overhang events on each sequence side.

<img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/img12.png style="zoom: 67%;"  >

### Pattern sequence templates

Pattern sequence templates are typically sub- or partial sequences refering exclusively to relative notes based on the sequencers internal states. This decouples them from absolute note values and keeps the templates  more generic. Usually they are assigned to perl variables for further usage in higher level sequence strings.

This allows for instance building different chord-type templates or small arpeggio pattern templates without attaching them to absolute note values for later instanciation within sequences.

<img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/img14.png style="zoom: 80%;"  >

The example above shows how a simple chord triad is setup as a template and beeing used in a small sequence.

> Remark: Events like `1/1<:0_%` are so called NOP (no operation) events since they do not advance in time neither inserting a note, however they instruct the sequencer to set the current timestamp and the current note value for further sequence events.

The following example is very similar and shows how to insert a small arpeggio sequence using templates.

<img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/img16.png style="zoom: 80%;"  >

Now one template example demonstrating how to use subsequences with and without sequencer value restoration. Actually when we simply concatenate a sub-sequence template such as `" > ^2 ^2 . "`  multiple times, the sequencer will increase continously all note values with each insertion. In contrast, if we put the sub-sequence into parentheses, the sequencer will take care about storing and restoring note values when the template gets entered or left. Therefore the resulting sequence is totally different.

<img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/img17.png style="zoom: 80%;"  >

### additional note attribute data

Each note- or pause-event of the micro sequencer supports additional (optional) attributes such as on/off velocity values and/or attached controller data series. This way you can easily attach additional articulation attributes to each individual note in alignment with note-on and duration timestamps. Attributes are either single events inserted at the current timestamp in sequence or continous event series inserted along the given note duration. Attributes can be any kind of controller, aftertouch, pitchbend, sysex or tempo events.

<img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/Example4.png style="zoom:90%;"  >

------

## tutorial pages
<img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/Tut_p3.PNG width="100%">
<img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/Tut_p4.PNG width="100%">
<img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/Tut_p5.PNG width="100%">
<img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/Tut_p6.PNG width="100%">
<img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/Tut_p7.PNG width="100%">
<img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/Tut_p8.PNG width="100%">
<img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/Tut_p9.PNG width="100%">
<img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/Tut_p10.PNG width="100%">
<img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/Tut_p11.PNG width="100%">
<img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/Tut_p12.PNG width="100%">
<img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/Tut_p13.PNG width="100%">
<img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/Tut_p14.PNG width="100%">
<img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/Tut_p15.PNG width="100%">
<img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/Tut_p16.PNG width="100%">
<img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/Tut_p17.PNG width="100%">
<img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/Tut_p18.PNG width="100%">