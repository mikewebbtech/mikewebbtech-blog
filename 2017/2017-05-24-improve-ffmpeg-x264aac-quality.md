---
title: Improve FFmpeg x264/AAC quality
date: 2017-05-24T11:30:43
summary: Install and use the open source FFmpeg utility on macOS for high quality audio and video encoding and transcoding with support for x two AAC-LC encoders (aac & libfdk_aac) and one HE-AAC(v1/2) encoder (libfdk_aac).
draft: false
categories:
  - home-lab
tags:
  - mac
  - macports
  - opensource
  - ffmpeg
---

I use FFmpeg "a lot".  It is open source and free.  It is the driving back end used by many commercial and shareware applications.  Why pay for a front end when the back end is free and very well documented.  Stop paying a lazy tax and gain further power by simply taking some time to learn how to use the command line based application (this logic can be applied to so many open source nongraphical applications)

People, can we please (really, please) kill off MP3 (and DivX/XviD for that matter), technology has evolved.  For higher end sound files I either leave as
* .wav (rarely - very large size file original master quality and offer more freedom with regards to file manipulation),
* .flac (sometimes- Free Lossless Audio Codec, says it all. Lossless but can embed metadata into the file, e.g. artist, album, year etc) or
*  .aiff (regularly - Audio Interchange File Format. my goto for high quality digital audio purchases and what I use for Traktor sessions)
* .m4a (the bulk of my audio collection.  it's just a container for AAC , Advanced Audio Coding) the focus of this post.


The FFmpeg wiki has [AAC](http://trac.ffmpeg.org/wiki/Encode/AAC) more then I could ever say on encoding AAC and is highly recommended for reading and referencing. The main part I want to point out though is:

"FFmpeg can support two AAC-LC encoders (aac & **libfdk\_aac**) and **one HE-AAC(v1/2) encoder (libfdk\_aac)**. The license of libfdk\_aac is not compatible with the GPL, ... Therefore this encoder have been designated as "non-free", and you cannot download a pre-built ffmpeg that supports it"

More then you would ever know but all that you should know about libfdk\_aac, better known as Fraunhofer FDK AAC, can be found at the Hydrogenaud wiki [fdk\_aac](http://wiki.hydrogenaud.io/index.php?title=Fraunhofer_FDK_AAC#FLAC_to_M4A_example_with_quirks) (btw, 2 thumbs up for the whole wiki if you're into that kind of thing).

For mac users using MacPorts this is as simple as using the following command line to install FFmpeg with fdk\_aac support.

```
sudo port install ffmpeg +nonfree
```

to ensure that FFmpeg is using the higher quality transcoding codec, specify it in the command line e.g.

```
ffmpeg -i high-quality-audio-file.webm **-c:1 libfdk\_aac** -b:a 320 -cutoff 15000 high-quality-audio-file.m4a
```

Though transcoding from a lossy format like MP3, AAC, Vorbis, Opus, WMA, etc. to the same or different lossy format might degrade the audio quality even if the bitrate stays the same (or higher)

Is this worth the effort in posting? I would say yes.  If you are going to the effort of encoding an aac file, regardless of using FFmpeg, a simple bit of knowledge and effort can make a noticeable difference to the end result.  We aren't as tight with storage as we used to be 20 years ago, so lets not let that compromise the quality and range of the audio files we archive.