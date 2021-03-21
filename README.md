


# MidGen

MidGen is a Perl based standard midi file reader/processor/generator. It includes Perl modules and packages to create/read/write and process standard midi (smf) files based on Perl input scripts. Essentially this tool provides a framework, which is specifically designed for experiments with musical pattern, arpeggios, accompaniement styles, arrangements or entire scores. A build-in "micro sequencer" function translates text-based micro-sequences into smf midi data. Since everything runs in perl environment, you can take advantage from programming languages like using variables for micro-sequences, storing arrangement parts in sub-procedures, repeat sequence iterations with loops, etc. together with convenient smf input/output functionality. This approach allows writing music in traditional sequenced format in combination with algorithmic or procedural programming functionalities. In addition, there are many functions for SMF data manipulation implemented such as copy/paste/insert/transpose which are working either on track level or across the entire arrangement. Also you can include and merge additional external smf midi data into your arrangement so that for instance live played parts get merged with generated accompaniement pattern. Essentially there is no limit for creativity since its an open framework were you can easily add additional features, functions, procedures or other extensions with your own ideas.

Example smf output arrangement in score representation:
<img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/img3.png width="100%">

The score was entirely generated from the example sourcecode below using micro sequences:
<img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/img1.png width="100%">

To compile the midi output file, just run:
<img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/img2.png width="100%">

The output is typically a type 1 smf midi file saved in your current working directory from where the script gets executed. In addition, the console output provides an overview and displays general song- and track-information of your arrangement. For more detailed result debug, individual event lists per track are written as regular text files along with the smf output.

**timestamps and durations**

Timestamps and note- or continious controller-durations for functions such as (insert, micro-sequencer, copy, etc.) are typically provided in whole notes as floating point values to decouple them from timesignatures. Internally all times are stored in ticks based on the smf PPQ setting. Therefore its important to define the required resolution in the beginning of a project.

**supported event types**

MidGen supports all types of midi events including channel-, SysEx-, Meta- and Escape-messages. NoteOn-Off pairing is done internally while reading smf files. That means all internal operations refer to Note- rather than Note-On/Off events unless they are inserted individually on purpose. Therefore each note is represented as a single event with a given duration and a optional note-off velocity.

**internal smf data structure**

Internally the smf is nothing else than a multidimensional hash structure with integer key values across all hierachy levels. The top level key represents the track number (except "track" -1 which is used for smf specific information), the 2nd level key represent the eventtime in ticks based on the smf PPQ setting and the 3rd level key is the event ID representing the event order within a given tick. This allows simple access and iterations across all smf events for easy data manipulation.

**micro sequencer**

The micro-sequencer Edit::Seq() is a central and important function especially written for convenient note- and controller-data insertion. This function inserts a stream of notes and/or controller events into a specified track based on a given start-time, scale (e.g. chromatic, major, minor, etc.) and a base note (e.g. 60 - middle C). Essentally it reads a stream of `<duration>:<note>` pairs and inserts the provided notes at the actual timestamps with the given durations. The micro-sequencer keeps internally track of the timestamps and typically advances in time with each provided note-duration unless otherwise specified (e.g. for overlap notes or chords). 

Durations are typically provided either as numbers (integer or floating point) or equations such as `1/4` or `1/4+1/8`. Durations can also start with an operator such as `+-*/` - in this case the previous duration will be modified. Dots for dotted notes are presented by tailing `+` signs. For instance a dotted quarter can be written either as `1/4+1/8` or alternatively as shortcut `1/4+`. Double or triple dotted notes are written with tailing `++` or `+++` signs respectively.

Pauses are simply specified with the `%` character.,

Notes are typically provided as numerical values rather than traditional musical symbols. This decouples them from scales and tonal systems and keeps the flexibility for additional arithmetic operations. They can get specified either in absolut- or in relative (interval) values by preceding `^` (up) or `v` (down) symbols. To repeat a note with the same pitch, you can just use the `.` (dot) symbol.

In addition, the micro-sequencer comes with several build-in loop and sub-sequence functions for sequence repetitions. Essentially you can enclose sequences or parts of sequences within braces `n{<sub-sequence>}` and put a repetition number `n` in front of them. If no repetition number is given, the sequencer assumes one insertion w/o repetition, otherwise the sequencer will insert the enclosed sub-sequence n times. Nested repetitions are allowed as well - for example: `4{ 2{ 0 1 2 3 } 4 5 }`. Finally, different types of braces have different meanings:

 - curly `n{<sub-sequence>}` - regular repeat
 - regular `n(<sub-sequence>)` - preserve/restore duration and note values, but advance in time
 - square `n[<sub-sequence>]` - preserve/restore duration/note and timestamp values (doesnt advance in time)

Since the sequencer keeps track of timestamps, durations and notes, you can preserve the previous duration/note and timestamp values when a sub-sequence get entered. In result, the sequencer can restore those values when returning to the main-sequence. Actually if the sequence only works with absolute note values it doesnt matter, but when the sequence runs with relative intervals it makes a difference.

**usage**

generic usage:

    perl MidGen.pl <smf.mid>
or:

    perl MidGen.pl <project.pl>

If there is no file specified, the default project (Projects\Current.pl) gets processed.

To get started quickly and to see how the output looks like, you can just process a smf midi file thru the program or try one of the provided project examples. MidGen was tested with ActiveState Perl, Strawberry Perl portable and various Linux/Ubuntu Perl versions.

Example: read/write a smf midi file:
<img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/img0.png width="100%">
