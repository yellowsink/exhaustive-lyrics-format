# The Exhaustive Lyrics Format

The Exhaustive Lyrics Format is a text format for storing the lyrics to music.
It aims to support *everything* you would need it to, standardized, be easy to human write and easy to machine read, and be easy enough to implement.

**!!! This project is under construction, it is not yet usable in any form. !!!**

| Contents | [Motivation](#motivation) | [Features](#features) | [The Spec](#the-spec) |
|----------|---------------------------|-----------------------|-----------------------|

## Motivation

The current state of lyrics formats is quite unsatisfactory:

The standard from the past is the LRC file, the features of which have long since been surpassed by many providers.
There exists an extension for multiple singers, but that is limited in adoption and only allows two singers
(male and female, or duet, for both), which is far from ideal.
There also exists an extension for timed words, but both of these extensions suffer from the issue that they are
de-facto implementations by authors of software using LRC, not proper features of LRC that are widely supported.

Apple Music serves its lyrics to the client using the [W3C standard TTML](https://www.w3.org/TR/ttml2/),
with some proprietary extensions to make it more suitable for lyrics (TTML is designed for subtitling!).
My attempts to reverse engineer how Apple Music's word emphasis works have been unsuccessful so far, I can't find
any relevant information in the served TTML.
Due to using a format designed for rich subtitling, the format is very tedious to read and edit, and has high overhead.

Musixmatch serves a custom JSON ["Rich Sync" format](https://developer.musixmatch.com/documentation/api-reference/track-richsync-get).
This format is only really machine-readable, and lacks some features that Apple appears to have including emphasis and
multiple speakers. But, Musixmatch's format does time sync to individual characters, not just words!
This feature seems of low priority and of difficulty to do cleanly - indeed it directly results in hard to human-read
lyrics, but it is nice to have in a machine-written and machine-read file.

So each lyrics format has some flaw to it, be it limited feature set, over complication, lack of human usability, or
standardization.

Ideally the lyrics we want should be readable by a machine to give the rich synced lyrics with multiple parts and backings
that users of services like Apple Music are accustomed to,
and also should be openable in a text editor for the most basic of users to read along to without sophisticated tools.

It would be nice to use LRC as a guideline for how files may look at a glance, but mainly just by virtue of being the
main open text format for lyrics.

### Anti-Motivation

[![XKCD 927, How Standards Proliferate](https://imgs.xkcd.com/comics/standards.png)](https://xkcd.com/927/)

## Features

- Track metadata (performance etc)
- Lyric metadata (songwriting etc)
- Linking to an audio file, with audio offset
- Audio sync for lines
- Audio sync for words
- Audio sync for characters (not recommended for most usage)
- Multiple singers (no limit on number)
- Singer metadata
- One, *or, importantly, more* of those singers assigned to each line
- Backing vocals, linked to a singer or not
- Word/phrase emphasis
- Sections (chorus, verse, bridge, etc.)
- Instrumental breaks
- Overlapping lines
- Human-readable and writable, but not impractical to machine-read
- Open format in the public domain

## The Spec

The data should be represented, when in binary, as a UTF-8 text file.
(["plain text" is too vague](https://youtu.be/gd5uJ7Nlvvo))

Lines are separated with a newline `\n`. Trailing whitespace on a line, including `\r`, is ignored.
Empty lines are ignored<!-- *unless otherwise stated*-->.

When `//` is encountered, the rest of the line becomes a comment.

In `#`-delimited setup blocks, lines containing only `#` are allowed and ignored,
to allow adding space to breathe for your metadata/parts/etc.

Now, the definition of the elements of an Exhaustive Lyrics Format file, in order:

The file starts with the text `XLRC r1` on its own line.

### Metadata

Metadata tags are added at the top, with a `#meta` line, then in a series of lines following the format:
```
# tagname: tagvalue
```

A line may contain just a `#`, to allow leaving space in the tags section.

Metadata must all occur in one block.
The following example defines common tags to use:
```
#meta
#
# title: Finale
# artist: Madeon
# author: Hugo Leclerq, Nicholas Petricca
# album: Adventure (Deluxe)
# track: 2
# disc: 2
# isrc: FR9W11114947
#
# audio: CD2/02 Finale.opus
# oset: 232.3
# fileauthor: John Doe
# editor: Cool Lyric Authoring Software Name:tm:
```

The `audio` and `oset` tags are special:
the `audio` tag is a relative file path (UNIX style) to an audio file that these lyrics are associated with,
and the `oset` tag is a millisecond-offset from the start of the file to count times from, positive values moving the lyrics later in the audio and vice versa.

Other tags are informational only and while using common names and formats is recommended, it is not required.

All tags are optional, and the entire metadata block itself is optional,
but accurate tagging of basic info, such as title, artist, and author, is recommended.

### Parts Setup

A "part" generally corresponds to a singer, useful for duets and large groups. You can set up as many as you choose.
This section can be omitted, in which case any attempt to assign lines to a part later on in the file is an error.

Each part has an ID, extra metadata, can optionally be set as a backing part, and if a backing part,
can be optionally tied to another part.
The motivation for this is that in some cases a track may have backings from the same singers as the main lyrics,
or the backing vocals are clearly tied to one singer or another and should be designated as such.

The parts setup starts with the text `#parts` at the start of a line, then a series of lines following one of:
```
# id:
# id: singer's name
# id: [backing]
# id: [backing] singer's name
# id: [backing: otherid]
# id: [backing: otherid] singer's name
```

A hidden part is implicitly present in all lyric files, defined as
```
# UNNAMED:
```

It is the default part used at the start of a file before any part identifiers are used, including files that never
use any parts.
If your lyrics only uses one main part, it may be desirable to simply assign metadata to the UNNAMED part.

Some example parts setups may look like this:
```
#parts
# UNNAMED: Madeon
```
or with backings:
```
#parts
# NP: Nicholas Petricca
# Back: [backing: NP]
```
or for a larger group/band, using shorthand IDs to make them more convenient to use within the lyrics:
```
#parts
# Jin:
# Su: Suga
# JH: J-Hope
# RM:
# Jim: Jimin
# V:
# JK: Jungkook
```

### Song Sections

Sections of songs (chorus, verse, bridge, etc) are placed on their own line, in `()` parentheses.

For example:
```
(Chorus)
(Verse)
(Bridge)
```

Sections need not be predefined, they are implicitly created with usage.

If a lyrics file contains sections, but the start of the lyrics are not tagged with a section, they will be assigned
the section with an empty string as its name.

### Lyric Lines

A line of lyrics follows the following general format:
```
[hh:mm:ss.xxx] {part1,part2} some lyrics <{part3}> here <[hh:mm:ss.xx]> <xxx> more lyrics <! here !> <[hh:mm:ss.xx]>
```

The timestamp section in the square brackets `[]` is the absolute time in the track when this line starts.
The hours section `hh:` is optional. The minutes section `mm:` is also optional, but it is recommended to keep it,
and it is also recommended, but not required, to pad all sections with leading 0s.
The seconds and sub-second parts are required, but the required length/precision of the sub-second part is undefined.

The timestamp is optional. Omitting a timestamp will mean that that line is not synced.
How playback software chooses to handle a mixed synced/unsynced file is not defined here.

The part section in the curly braces `{}` contains the part IDs to use for this line, comma separated.
Optional, defaults to the parts used on the previous word.
The first line in the lyrics, if unspecified, uses `{UNNAMED}`, as previously described in the section on part setup.

The body of the lyrics line is the text of the lyrics.

Mid-line sync or part data is added using angle brackets `<>`.

Mid-line timestamps, using square brackets within angle brackets `<[]>` are relative to the line start.
A mid-line timestamp directly at the start of the line sets the start time of the first word relative to the line start,
and all other mid-line timestamps set the end time of the previous word.
The end time of a line is always the end time of the last word in it.
In mid-line timestamps, the seconds `ss.` may also be omitted,
in which case the sub-second part always counts milliseconds, instead of decimal values of a second.

If the end of a line is not specified with a timestamp, it is implicitly the start time of the next line.

Mid-line part changes, using curly braces within angle brackets `<{}>` replace the parts selection from the next word on.
This changed selection will carry on to the next line if the next line does not have parts defined.
For convenience, an empty mid-line part change `<{}>` resets to the parts chosen at the start of the line, useful if a
second singer sings just a couple words of a line along with the main singer.
The closest to an "empty" parts change possible is to switch to the unnamed part `<{UNNAMED}>`.

Small offsets to timing can be applied using angle brackets `<>` containing a number of milliseconds from the last sync point.
This is useful to add a purposeful gap between the end of one word and the start of the next,
or to sync individual characters.

Mid-line *offset* tags do not cause word boundaries: they may be placed between words, but also within words to sync characters.
This does not apply to part changes and timestamps, which should be on word boundaries, and will cause word boundaries
if placed within a word.

Emphasis can be applied to words or phrases within a line by surrounding the words with `<!` and `!>` tags.
These must stay within a line and occur on word boundaries.

### A final example

Here is a final example to leave off: a partial Exhaustive Lyrics Format transcription of Black Wedding by In This Moment:

```
XLRC r1

#meta
#
# title: Black Wedding
# artist: In This Moment feat. Rob Halford
# author: Chris Howorth, Maria Brink, Kevin Churko, Billy Idol, Scott Stevens
# album: Ritual
# fileauthor: Yellowsink
#
# oset: -156

#parts
# MB: Maria Brink
# RH: Rob Halford
# MBb: [backing: MB]

// This first verse demonstrates syncing in XLRC

(Verse)
[00:07.88] {MB} Priest, <[407]> are <[684]> you <[1.088]> there?
[00:09.60] Can <[217]> you <[441]> hear <[716]> my <[1.183]> voice?
[00:11.39] Will <[214]> you <[430]> hear <[665]> my <[1.031]> prayers?
[00:12.96] Are <[0.113]> you <[519]> out <[839]> there? <[1.491]>

// Manual word syncing is very tedious, I will skip a bit and just do more interesting syncs
[00:14.72] Forgive me, priest
[00:16.59] For I have sinned
[00:19.95] {MBb} I know not what I do
[00:21.77] {RH} Mother I am here, I can hear your song
[00:25.36] I can feel your fear, he's done you wrong
[00:28.63] but Temptation fed, his own desires
// Notice these overlap!
[00:33.96] In the ring of fire <[2.56]>
[00:35.84] {MBb} In the ring of fire <[2.98]>

// That's a very long "i"!
[00:38.49] {RH} Ton<341>i<2014>ght!

// Syncing is sufficiently demonstrated, so this chorus will demonstrate more complex multi-part lyrics

(Chorus)
{MB,RH} I would've loved you for a thousand years
{RH} I would've died for you
{MB,RH} I would've sacrificed it all my dear
{MB} I would've bled for you

{MB,RH} 'Til death do us part
{MB} You were unholy right from the start
{MB,RH} It's a nice night for a black wedding
Yeah, it's a nice night for a black wedding

(Verse)
```

![](https://github.com/catppuccin/catppuccin/raw/main/assets/footers/gray0_ctp_on_line.svg)

Oh hey, you reached the bottom! Hi!

Found an issue with the format? Feel free to open an issue pointing out what's missing / needs improvement / broken.
