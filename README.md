# The Exhaustive Lyrics Format

The Exhaustive Lyrics Format is a text format for storing the lyrics to music.
It aims to support *everything* you would need it to, standardized, be easy to human write and easy to machine read, and be easy enough to implement.

**!!! This project is under construction, it is not yet usable in any form. !!!**

| Contents | [Motivation](#motivation) | [Features](#features) | [Standard](#standard) |
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

## Features

- Track metadata (performance etc)
- Lyric metadata (songwriting etc)
- Linking to an audio file, with audio offset
- Audio sync for lines
- Audio sync for words
- Audio sync for characters (not recommended for most usage)
- Multiple singers (no limit on number)
- Singer metadata
- One, *or, importantly, more* of those singers on a single line
- Backing vocals, linked to a singer or not
- Word/phrase emphasis
- Sections (chorus, verse, bridge, etc.)
- Human-readable and writable, but not impractical to machine-read
- Open format in the public domain

## Standard

WIP.
