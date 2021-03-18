

# MidGen

MidGen is a Perl based standard midi file reader/processor/generator. It includes Perl modules and packages to create/read/write and process standard midi (smf) files based on Perl input scripts. Essentially this tool provides a framework, which is specifically designed for experiments with musical pattern, arpeggios, accompaniement styles, arrangements or entire scores. A build-in "micro sequencer" function translates text-based micro-sequences into midi data.

Example smf output arrangement in score representation:
<img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/img3.png width="100%">

The score was entirely generated from the following example sourcecode:
<img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/img1.png width="100%">

To compile the midi file, just run:
<img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/img2.png width="100%">

The output is typically a smf midi file saved in your current working directory from where the script gets executed. In addition, the console output provides an overview and displays general song- and track-information of your arrangement. For more detailed result debug, individual event lists per track are saved as regular text files along with the smf.

MidGen supports all types of midi events including channel-, SysEx-, Meta- and Escape-messages. NoteOn-Off pairing is done internally while reading smf files. That means all internal operations refer to Note- rather than Note-On/Off events unless they are inserted individually on purpose. Therefore each note is represented as a single event with a given duration and a optional note-off velocity.

Timestamps and durations for functions (e.g. insert, sequencer, copy, etc.) are typically provided in whole notes as floating point values to decouple them from timesignatures. Internally all times are stored in ticks based on the smf PPQ setting. Therefore its important to define the required resolution in the beginning of a project.

Event data can be inserted and processed in various ways dependent on the event type.

Note data can be inserted by using a so called micro-sequencer Edit::Seq(). This specific function allows to insert small event sequences into the arrangement based on text string arguments.

Continous data such as controller, pitch bend, after touch, tempo changes, SysEx etc. can get inserted as event series allowing for smooth controller sweeps by using single commands.

generic usage:

    perl MidGen.pl <smf.mid>
or:

    perl MidGen.pl <project.pl>

If there is no file specified, the default project (Projects\Current.pl) gets processed.

To get started quickly and to see how the output looks like, you can just process a smf midi file thru the program or try one of the provided project examples. MidGen was tested with ActiveState Perl, Strawberry Perl portable and various Linux/Ubuntu Perl versions.

Example: read/write a smf midi file:
<img src=https://raw.githubusercontent.com/MrBMueller/MidGen/master/img/img0.png width="100%">
