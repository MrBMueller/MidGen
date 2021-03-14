
# MidGen

MidGen is a perl based standard midi file generator. It includes Perl scripts and packages to create/read/write and modify standard midi (smf) files. The output is typically a smf midi file saved in your current working directory from where the script is executed. In addition, the console output displays general song- and individual track-information for a quick smf overview. For more detailed result debug, individual event lists per track are saved as regular text files along with the smf.

MidGen support all types of midi events including channel-, SysEx-, Meta- and Escape-messages. NoteOn-Off pairing is done internally while reading smf files. That means all internal operations refer only to Note- rather than Note-On/Off events unless they are inserted individually on purpose. Therefore each note is represented as a single event with a given duration and optionally a given note-off velocity.

Timestamps for functions (e.g. insert, sequencer, copy, etc.) are typically provided as whole notes (floating point) to decouple the time representation from Timesignature. Internally all times are stored as ticks based on the smf PPQ setting. Therefore its important to define the required resolution in the beginning of a project.

Continous data such as controller, pitch bend, after touch, tempo changes, SysEx etc. can get inserted as event series allowing for smooth controller ramps and sweeps by using single commands.

usage:

    perl MidGen.pl <smf.mid>
or:

    perl MidGen.pl <project.pl>

If there is no file specified, the default project (Projects\Current.pl) is used.

To get started quickly and to see how the output looks like, you can just run a smf midi file thru the program.

