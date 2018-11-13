`Event`

* `MetaEvent(Event)`
  * Defines subset of Events known as Meta events
  * Not meant to be used as a concrete class

[Note On / Note Off events](#Note-On-/-Note-Off-events)
[Aftertouch](#Aftertouch)
[Control Change](#Control-Change)
[Program Change](#Program-Change)
[Channel Aftertouch](#Channel-Aftertouch)
[Pitch Bend](#Pitch-Bend)
[System Exclusive Messages](#System-Exclusive-Messages)
[Sequence Number (Meta Event)](#Sequence-Number-(Meta-Event))
[Text (Meta Event)](#Text-(Meta-Event))
[Copyright (Meta Event)](#Copyright-(Meta-Event))
[Track Name (Meta Event)](#Track-Name-(Meta-Event))
[Instrument Name (Meta Event)](#Instrument-Name-(Meta-Event))
[Lyrics (Meta Event)](#Lyrics-(Meta-Event))
[Marker (Meta Event)](#Marker-(Meta-Event))
[Cue Point (Meta Event)](#Cue-Point-(Meta-Event))
[Channel Prefix (Meta Event)](#Channel-Prefix-(Meta-Event))
[Port (Meta Event)](#Port-(Meta-Event))
[Set Tempo (Meta Event)](#Set-Tempo-(Meta-Event))
[SMPTE Offset (Meta Event)](#SMPTE-Offset-(Meta-Event))
[Time Signature (Meta Event)](#Time-Signature-(Meta-Event))
[Key Signature](#Key-Signature)
[Sequencer-Specific (Meta Event)](#Sequencer-Specific-(Meta-Event))


### Note On / Note Off events

* Activation and release of same note are considered separate events
* When key pressed on keyboard, Note On message sent to MIDI OUT port
  * Keyboard can be transmitting on any of sixteen MIDI channels
  * Status byte indicates selected channel number
  * Status byte followed by two data bytes
    * Key number (which key pressed)
    * Velocity (how hard key was pressed)
* __Classes__:
  * `NoteEvent(Event)`
    * Not meant to be used as concrete class
    * Defines generalities of NoteOn, NoteOff events
    * `pitch`
    * `velocity`
    * Standard getters/setters
  * `NoteOnEvent(NoteEvent)`
    * `statusmsg = 0x90`
    * Chan 1 Note on
  * `NoteOffEvent(NoteEvent)`
    * `statusmsg = 0x80`
    * Chan 1 Note off

### Aftertouch

* Some keyboards sense amount of pressure applied to keys when depressed
* If keyboard has sensor, data sent in form of Polyphonic Key Pressure messages
  * Have separate bytes for key number, pressure amount
* __Classes:__
  * `AfterTouchEvent(Event)`
    * `statusmsg = 0xA0`
    * `length = 2`
    * `pitch`
    * `value`
    * Standard getters/setters

### Control Change

* Used to control wide variety of function in synth
* Only affect channel number indicated in status byte
* Status byte followed by one byte indicating "controller number"
  * Identifies which function of synth is to be controlled by message
* Second byte specifies "control value"
* __Classes:__
  * `ControlChangeEvent(Event)`
    * `statusmsg = 0xB0`
    * `length = 2`
    * `control`
    * `value`
    * Standard getters/setters

### Program Change

* Used to specify type of instrument which should be used to play sounds on given channel
* Needs only one byte specifying new program number
* _Usually third event in sound channels?_
* __Classes:__
  * `ProgramChangeEvent(Event)`
    * `statusmsg = 0xC0`
    * `length = 1`
    * `value`
    * Standard getters/setters

### Channel Aftertouch

* (Currently more common than note aftertouch)
* Keyboard senses single pressure level for entire keyboard
* Sent using Channel Pressure message, needs only one byte to specify pressure value
* __Classes:__
  * `ChannelAfterTouchEvent(Event)`
    * `statusmsg = 0xD0`
    * `length = 1`
    * `value`
    * Standard getters/setters

### Pitch Bend

* Responds to change in position of pitch bend wheel
* Used to modify pitch of sounds being played on given channel
* Includes two bytes to specify pitch bend value
* Two bytes required to allow fine enough resolution to make pitch changes resulting from movement of pitch bend wheel seem to occur in continuous manner
* __Classes:__
  * `PitchWheelEvent(Event)`
    * `statusmsg = 0xE0`
    * `length = 2`
    * `pitch`
    * Standard getters/setters

### System Exclusive Messages

* May be used to send data such as patch parameters or sample data between MIDI devices
* _Seems almost superfluous for our purposes_
* __Classes:__
  * `SysexEvent(Event)`
    * `statusmsg = 0xF0`
    * `length = 'varlen'`
    * `is_event` (attr and method)

### Sequence Number (Meta Event)

* Optional event, must occur at beginning of track (before any nonzero delta-times, before any transmittable MIDI events)
* Specifies number of a sequence
* Format 2 MIDI
  * Used to identify each pattern so a song sequence using cue message to refer to patterns
* Format 0/1 MIDI
* __Classes:__
  * `SequenceNumberMetaEvent(MetaEvent)`
    * `metacommand = 0x00`
    * `length = 2`

### Text (Meta Event)

* Any amount of text describing anything
* Good idea to put text at beginning of track
* __Classes:__
  * `MetaEventWithText(MetaEvent)`
    * `text`
  * `TextMetaEvent(MetaEventWithText)`
    * `metacommand = 0x01`
    * `length = 'varlen'`

### Copyright (Meta Event)

* Contains copyright notice as printable ASCII text
* __Classes:__
  * `CopyrightMetaEvent(MetaEventWithText)`
    * `metacommand = 0x02`
    * `length = 'varlen'`

### Track Name (Meta Event)

* Name of sequence/track
* __Classes:__
  * `TrackNameEvent(MetaEventWithText)`
    * `metacommand = 0x03`
    * `length = 'varlen'`

### Instrument Name (Meta Event)

* Description of type of instrumentation to be used in track
* __Classes:__
  * `InstrumentNameEvent(MetaEventWithText)`
    * `metacommand = 0x04`
    * `length = 'varlen'`

### Lyrics (Meta Event)

* Lyric to be sung
* Generally, each syllable will be separate lyric event which begins at event's time
* __Classes:__
  * `LyricsEvent(MetaEventWithText)`
    * `metacommand = 0x05`
    * `length = 'varlen'`

### Marker (Meta Event)

* Name of that point in sequence (rehearsal letter or section name)
* ex. "First Verse"
* __Classes:__
  * `MarkerEvent(MetaEventWithText)`
    * `metacommand = 0x06`
    * `length = 'varlen'`

### Cue Point (Meta Event)

* Description of something happening on film or video screen or stage at point in score
* ex. "Car crashes into house", "curtain opens", etc.
* __Classes:__
  * `CuePointEvent(MetaEventWithText)`
    * `metacommand = 0x07`
    * `length = 'varlen'`

### Channel Prefix (Meta Event)

* MIDI channel in event may be used to associate MIDI channel with all events which follow
  * Including sysex and meta events
* Effective until next normal MIDI event (which contains a channel) or next MIDI channel prefix meta event
* __Classes:__
  * `ChannelPrefixEvent(MetaEvent)`
    * `metacommand = 0x20`
    * `length = 1`

### Port (Meta Event)

* dunno
* __Classes:__
  * `PortEvent(MetaEvent)`
    * `metacommand = 0x21`

Skip `TrackLoopEvent`, `EndOfTrackEvent`

* Note for later: need to be able to choose how long to make track

### Set Tempo (Meta Event)

* Indicates tempo change
* Represented as time per beat (rather than beats per time)
* Ideally should only occur where MIDI clocks would be located
* __Classes:__
  * `SetTempoEvent(MetaEvent)`
    * `metacommand = 0x51`
    * `length = 3`
    * `bpm`
    * `mpqn` $\rightarrow$ microseconds per MIDI quarter note
    * Standard getters/setters

### SMPTE Offset (Meta Event)

* Optional
* Designated SMPTE (timecode) time at which track chunk is supposed to start
* Should be present at beginning of track (before any nonzero delta-times, before any transmittable MIDI events)
* __Classes:__
  * `SmpteOffsetEvent(MetaEvent)`
    * `metacommand = 0x54`

### Time Signature (Meta Event)

* Expressed as four numbers, $nn$, $dd$ represent numerator and denominator of time signature as it would be notated
* `FF 58 04 <nn> <dd> <cc> <bb>`
* Denominator is negative power of two 
* 2 represents quarter-note, 3 represents eighth-note, etc.
* $cc$ expresses number of MIDI clocks in metronome click

* $bb$ expresses notated 32nd-notes in MIDI quarter note
* __Classes:__
  * `TimeSignatureEvent(MetaEvent)`
    * `metacommand = 0x58`
    * `length = 4`
    * `numerator`
    * `denominator`
    * `metronome`
    * `thirtyseconds`
    * Standard getters/setters

### Key Signature

* `FF 59 02 <sf> <mi>`
* $sf$
  * -7: 7 flats
  * -1: 1 flat
  * 0: key of C
  * 1: 1 sharp
  * 7: 7 sharps
* $mi$
  * 0: major key
  * 1: minor key
* __Classes:__
  * `KeySignatureEvent(MetaEvent)`
    * `metacommand = 0x59`
    * `length = 2`
    * `alternatives`
    * `minor`
    * Standard getters/setters

### Sequencer-Specific (Meta Event)

* May be used when special requirements for particular sequencers are needed
* First byte/bytes is manufacturer ID