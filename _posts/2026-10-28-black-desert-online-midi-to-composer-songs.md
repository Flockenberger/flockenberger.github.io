---
title: "Black Desert Online: Loading MIDI Files into BDO"
date: 2026-10-28
layout: splash
categories:
  - blog
tags:
  - bdo

excerpt: "How BDO stores its composer songs - and how to get any MIDI file into one"
header:
  overlay_color: "#252a34"
  overlay_image: assets/images/bdo_music/midi_converter.png
  actions:
    - label: "Download"
      url: "https://github.com/Flockenberger/BDMusic"
author_profile: false
toc: true
toc_label: "MIDI to BDO Music"
---
Hey there! Another writeup, another rabbit hole. This one is about music - or more precisely, about getting MIDI files into Black Desert Online's (BDO) in-game composer, and everything I learned about how the game stores its songs along the way.


# Backstory
BDO has a pretty nice music composer built into the game. You pick instruments, place notes on a piano roll and can then play your songs in game, alone or with other players. The catch: placing every note by hand takes forever. Meanwhile there are thousands of MIDI files out there for pretty much every song you can think of.

So a long time ago I wrote a little MIDI to BDO converter. And if I remember correctly, it never really worked. Looking at it again now, that memory was right. The old code:

- wrote every note as the same key at the same velocity,
- only read the first track of the MIDI and had a hard-coded instrument list,
- took the tempo from the last tempo change instead of the first

Needless to say it was essentially just a proof of concept.

So, time to do it properly: read the MIDI correctly, with all its velocities, pedals and settings, and write a song the game actually opens.

# How BDO stores a song
Songs are saved in your `Documents\Black Desert\Music` folder. Each song is one file without an extension, and the file name *is* the song's name in game.

The first thing you'll notice when you open one in a hex editor is... nothing. Apart from the first four bytes, it's all noise. Those four bytes are the file's version (currently 9), and everything after them is encrypted with **ICE** (Information Concealment Engine), an old 64-bit block cipher from the 90s, in its "thin" variant. Pearl Abyss uses the same cipher for a few different things so it was not something I had never seen before.

Once decrypted, the format is surprisingly simple. I worked it out from the client's own save and load code, plus the composer itself. Fun fact: the composer is actually a web page inside the game, HTML and JavaScript, that hands the finished song to the client as XML. The client then turns that into the binary file. The page also holds every instrument's note range and the names of the drum sounds, which came in handy.

As usual, here is the format as a (slightly trimmed) Kaitai Struct spec:

```yaml
seq:
  - id: version
    type: u4            # 9; the only part that isn't encrypted

  - id: user_no
    type: s8            # the author's user number

  - id: author
    type: strz
    size: 62            # UTF-16LE, 30 characters + 0

  - id: last_composer
    type: strz
    size: 62            # whoever saved the song last

  - id: bpm
    type: u2            # the game clamps it to 200 when loading

  - id: beat
    type: u2            # 3, 4 or 6 (3/4, 4/4, 6/8)

  - id: instruments
    type: strz
    size: 201
    encoding: ASCII     # the tracks' instruments, comma separated: "13,14,36"

  - id: num_tracks
    type: u2

  - id: tracks
    type: track
    repeat: expr
    repeat-expr: num_tracks
  
  - id: editable
    type: u1            # 1 for your own song, 0 for one from someone else

types:
  track:
    seq:
      - id: num_chunks
        type: u2
      
      - id: chunks
        type: chunk
        repeat: expr
        repeat-expr: num_chunks
 
 chunk:
    seq:
      - id: len_body
        type: u2        # 12 + 20 * num_notes
      
      - id: instrument
        type: u1
      
      - id: volume
        type: s1        # 0-100, 70 by default
      
      - id: effects
        size: 8         # reverb, delay and chorus sends plus the song-wide effect settings
      
      - id: num_notes
        type: u2
      
      - id: notes
        type: note
        repeat: expr
        repeat-expr: num_notes
  note:
    seq:
      - id: note_no
        type: u1        # the MIDI note number, 60 = middle C
      
      - id: articulation
        type: u1
      
      - id: on_velocity
        type: s1
      
      - id: off_velocity
        type: s1
      
      - id: start_ms
        type: f8
      
      - id: length_ms
        type: f8
```

A few things in there that weren't obvious:

- **Notes are timed in milliseconds**, stored as doubles, not as positions on the beat grid. That's great news for a converter, because I don't have to squeeze anything onto a grid. A note can start exactly when it starts in the MIDI.

- **The chunks.** The game doesn't write a track's notes in one block. It splits them into chunks of at most 730 notes, repeats the track's settings in every chunk and always adds one empty chunk at the end. How many chunks there are follows a slightly odd formula straight from the client, and my writer copies it exactly:

```cpp
flib::UInt32 ChunkCount(size_t notes)
{
	if (notes == 0)
	{
		return 1;
	}

	return (flib::UInt32)std::ceil((flib::Float)(flib::Int32)(21 * notes) / 15329.0f) + 1;
}
```

- **Older versions differ.** The client still reads old songs: below version 5 there was no user number, below 7 no instrument list, and before version 8 there were no chunks at all, and notes were stored as floats.

- **Articulations are just numbers.** The instrument table in the game data lists which ones each instrument can play, but not their names. To show a name, the client builds a text key from the number and hashes it to find the name in the localization data. That's how I got the full list: Sustain, Short, Pizzicato, Tremolo, Mute, Harmonics, Slap, Sustain Pedal and so on, 29 in total.

# Reading MIDIs properly
With the BDO side done, the other half is the MIDI file. A MIDI file is basically a list of events: note on, note off, controller changes, tempo changes and so on, each with a time in ticks. Turning that into "this note starts at 1234.5 ms and lasts 300 ms" sounds simple, but there's a lot hiding in there:

- **Timing:** ticks become milliseconds through the full tempo map, so songs with tempo changes stay in time.
- **Pairing notes:** a note on is matched with its note off first in, first out per key, so overlapping notes on the same key stay separate.
- **Pedals:** the sustain pedal keeps released notes ringing until it lets go. The sostenuto pedal and mono mode change note lengths as well.
- **Velocity:** comes from the MIDI, scaled by the channel's volume and expression. A note takes the loudest level its channel reaches while it sounds, so fade-ins don't get lost.
- **Instruments:** every General MIDI program goes to the closest of the composer's 38 instruments (16 of them are synth variants). Drums go to the drum set, with bongos and congas on the hand drum.
- **Ranges:** every instrument has a playable range, and notes outside it move by octaves until they fit.
- **Merging:** MIDI parts that end up on the same instrument share one track. If two of them hit the same key at the same moment, that becomes one note.

And then I went one step further: **every setting a MIDI carries that has a counterpart in BDO is used**, unless you override it. Time signatures (12/8 becomes the composer's 6/8), the reverb, delay and chorus sends, the song-wide effect settings that GS and XG files set through SysEx messages, master volume, per-channel tuning, key shifts, drum kits on other channels, note-off velocities...

## The one bug that wasn't mine (he says)
To test all of this, I generated a MIDI file that uses every one of these features and checked the converted song note by note. Most things worked right away, but a few didn't: drum kits selected by bank, slides and tuning changes all got lost.

The culprit turned out to be the MIDI library I use. When it sorts events by time, it uses a sort that isn't stable, and events on the same tick end up ordered by controller number. Unfortunately MIDI relies on that order in a few places. A tuning or bend-range change, for example, is a sequence of controller messages: first "which parameter", then "this value". Sorted by number, the value came *before* the parameter it belongs to, so the change went to the wrong place. So the converter now does its own stable merge of the tracks and builds its own tempo map. Not a fun one to track down, but at least I knew where to look thanks to the test file. :D

# Why everything sounded like Sustain (because it was)
After the first songs were working in game, the next problem showed up quickly: every single note was played as **Sustain**, and plain Sustain sounds pretty flat in BDO.

That isn't a converter bug, though. MIDI simply has no field for articulations. The best a converter can do is read what the file *implies*: a muted guitar or pizzicato strings program, a pitch bend of a semitone or more (Slide Up / Slide Down), modulation (Vibrato) or the sustain pedal (Sustain Pedal). Most MIDIs you find online have none of these, so there's simply nothing to read.

So now, if the MIDI says nothing, BDMusic picks what a player would most likely use. To get a feeling for that, I counted what hand-made songs actually use. The horn and clarinet, for example, play their loud SusForte about 96% of the time, and guitarists like Mute for short notes. The defaults ended up like this:

```cpp
// Nothing in the file says how: what a player would do with a note like this one
if (staccato)
{
	if (instrument.CanPlay(BDMArticulation::Short))
	{
		return BDMArticulation::Short;
	}

	// Palm-muted, as a driven guitar plays its short notes
	if ((instrument.Index == BDMInstruments::GuitarDrive || instrument.Index == BDMInstruments::GuitarDistortion)
		&& instrument.CanPlay(BDMArticulation::Mute))
	{
		return BDMArticulation::Mute;
	}
}

// The clarinet and horn sample each dynamic on its own; the velocity is the note's dynamic
if (instrument.CanPlay(BDMArticulation::SusPiano) && instrument.CanPlay(BDMArticulation::SusMezzoForte)
	&& instrument.CanPlay(BDMArticulation::SusForte))
{
	if (velocity < k_MezzoForteVelocity)
	{
		return BDMArticulation::SusPiano;
	}
	if (velocity < k_ForteVelocity)
	{
		return BDMArticulation::SusMezzoForte;
	}
	return BDMArticulation::SusForte;
}

// A piano part with no pedal at all, as notation programs write them: played with it held
if (instrument.Index == BDMInstruments::AdvancedPiano && !partPedals && options.UseSustain
	&& instrument.CanPlay(BDMArticulation::SustainPedal))
{
	return BDMArticulation::SustainPedal;
}

return instrument.DefaultArticulation();
```

- **Staccato** (held for an eighth note or less, and let go well before the part's next note) becomes **Short**, or **Mute** on the drive and distortion guitars.
- **Clarinet and horn** get their piano, mezzo-forte or forte samples depending on the note's velocity.
- **Piano parts the MIDI never pedals** get the **Sustain Pedal**, since notation programs often just leave the pedal out.

These rules are my best guess at what sounds good, so they can be switched off with `--plain-defaults`. And if you know better for a specific song, `--articulation` sets one articulation for a whole part.

# A few limits
- **BPM:** the composer only goes up to 200 bpm. Since notes are stored in milliseconds though, a faster song still plays at its real speed, the composer just shows 200.
- **Notes:** the composer lets you write up to 10,000 notes per track. The file itself can hold more, so BDMusic only warns you when a track goes over.
- **Re-saving:** if you open a converted song in the composer and save it again, the composer snaps every start time to 5 ms and every length to 1 ms. I didn't bother matching that - you won't hear the difference. (If you do, more power to you)
- **Song names:** the song is named after the MIDI file. At first I used the name stored inside the MIDI, but a lot of MIDIs "name" themselves after their first instrument, so I ended up with songs called "Xylophone" or "Guitar Lead 1". :D

# Using it
The result is **BDMusic**, a small console program. The easiest way to use it is to drag a `.mid` file onto `BDMusic.exe`. The song lands in your game's music folder under the MIDI's file name, ready to open in the composer.

From a command prompt you get more control:

```
BDMusic --parts "song.mid"                      lists the MIDI's parts and the instrument each one goes to
BDMusic --instruments                           lists the composer's instruments and their articulations
BDMusic "song.mid" --part 2=16 --part 5=none    part 2 on the Advanced Harp, drop part 5
BDMusic "song.mid" --articulation 3=5           part 3 as pizzicato
BDMusic "song.mid" --transpose -12 --trim --bpm 90
```

An existing song with the same name is never replaced unless you add `--force`. The README on GitHub has all the options.

BDMusic is open source under the **AGPL-3.0**: use it, change it, share it, but whatever you build on it has to stay open too, and keep the credit. The one thing missing from the source is, you guessed it, the encryption key. So if you want to build it yourself, you'll have to bring your own. The release on GitHub comes ready to use.

# Conclusion
What started as "let's quickly fix my old converter" turned into reading through the client's save code, a web page pretending to be a music program, and a bug in someone else's MIDI library. Classic.
