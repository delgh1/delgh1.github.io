---
layout: post
title: "A user's incomplete technical guide to Jellyfin and media streaming"
---
# Background knowledge

## What is streaming?

Simply put, the server sends a stream of video, audio, and optionally
subtitles, over the internet, to the client.

The difference between streaming and downloading a video file is that
streaming is real-time.  The user does not have to wait until the whole file
is downloaded.  The server divides the video file into many parts and serves a
bit of it at a time, and the client downloads it and plays it.  If the
internet connection allows, depending on the speed and latency of the
connection, the server serves more parts of the video file, and the client
downloads it and saves it locally, which is called buffering so that the video
can play more smoothly.

A large video is like a lake, and streaming is like a stream or river, the
quality and experience of streaming largely depend on the width of the stream
(the quality of said internet connection).

See more on [What is streaming? - How video streaming works -
Cloudflare](https://www.cloudflare.com/en-ca/learning/video/what-is-streaming/)

## What is Jellyfin

Jellyfin is a free (as in freedom) software media system, a free (also as in
free beer) media solution that consists of both server and client, uses
[HLS](https://en.wikipedia.org/wiki/HTTP_Live_Streaming) (HTTP Live Streaming
protocol) to stream media from server to client, and allows users to have
complete control of their media.

A Jellyfin server requires
[installation](https://jellyfin.org/docs/general/installation/) of software
packages on an appropriate OS/platform.  After the server installation, the
user is expected to create a library and to add/import media into the library.
Most people also set up a [NFS
share](https://jellyfin.org/docs/general/administration/storage) to share the
storage over network, as well as GPU [hardware
acceleration](https://jellyfin.org/docs/general/administration/hardware-acceleration/)
for real-time transcoding.

After properly setting up the server, the user can use a browser or [other
clients](https://jellyfin.org/downloads) to connect to the server and stream
media.

## What could possibly go wrong?

This is a non-exhausted list of what I have experienced:

- The server cannot access the media files (permission denied, need to
  properly set the file system permission)

- Green artifacts appear on the transcoded video when using AMD GPU (Radeon RX
  6600XT) with VAAPI hardware acceleration

- Messed up metadata

- Unresponsive browser window (Chrome's fault probably)

- Stuttering and low performance when playing 4K H265 10-bit HDR video (had to
  manually transcode to H264 8-bit SDR)

- When attempting to play anything at all, it displays a pop-up "Playback
  error - This client isn't compatible with the media and the server isn't
  sending a compatible media format." (solved by restarting the browser, still
  happening a few times a week as of May 2023)

- Chrome refuses to directly play 4K H265 10-bit HDR video even though it
  reported that [hardware support is
  enabled](https://github.com/StaZhu/enable-chromium-hevc-hardware-decoding#how-to-verify-hevc-hardware-support-is-enabled)

- Stuck at loading (while I am writing this)

The issues can be roughly divided into these categories: network, server, and
client. Network issues are very complicated, so let's focus on the server side
and the client side of the problem.

## Formats

Why can some videos "Direct Play" or "Direct Stream" but others require
"transcoding"? To answer this, one must know something called [codec
support](https://jellyfin.org/docs/general/clients/codec-support).

Many people are familiar with video files with the extensions `mp4`, `avi`,
`mov`, `mkv`, `m4v`, `webm`, `vob`, `3gp`, `flv`, `rmvb`, and people that have
used a computer long enough typically experienced the pop-up error
`unsupported video format`, which is usually shown when a video player does
not support such video file.  File extensions like `mp4` and `mkv` are merely
the name of the container of video, audio, and subtitles.  How video or audio
is encoded is another story. A raw video consists of frames of pictures like
analog films.  The pictures become a movie when they are moved/played at a
certain speed, typically about 24 frames per second.  In the digital age,
video formats are specific ways to code/compress video into something a
machine can understand, in other words, 1s and 0s.  Some containers can hold
almost all codecs while others may have limited support for certain formats.

[Codec is defined as software or hardware that encodes or decodes a data
stream or signal](https://en.wikipedia.org/wiki/Codec).  Here, sometimes the
term `codec` is used loosely referring to media formats.

### Video formats and codecs

Typically DVDs use `MPEG-2 Part 2` as the main video codec, a.k.a. `H.262`, or
simply `MPEG-2`.  This is decided by the [DVD
standard](https://en.wikipedia.org/wiki/DVD-Video).

On the other hand, Blu-rays use `MPEG-4 Part 10` or `H.264` or [AVC (Advanced
Video Coding)](https://en.wikipedia.org/wiki/Advanced_Video_Coding).  This is
by far the most popular video codec, and nearly all video players, (software,
or dedicated Blu-ray playback hardware) support it.

To support 4K UHD on Blu-ray, a newer and more efficient codec was developed,
named `MPEG-H Part 2` or `H.265` or [HEVC (High Efficiency Video
Coding)](https://en.wikipedia.org/wiki/High_Efficiency_Video_Coding).  It is
able to maintain the same quality while reducing the file size from 25% to 50%
compared to H.264, and it supports
[HDR](https://en.wikipedia.org/wiki/High-dynamic-range_television) (High
Dynamic Range video).  HDR has a much larger color range, in other words, it
can display much more color, brightness, and contrast than the old
[SDR](https://en.wikipedia.org/wiki/Standard-dynamic-range_video) (Standard
Dynamic Range), which was based on [CRT
display](https://en.wikipedia.org/wiki/Cathode-ray_tube).

However, the organization known as
[MPEG](https://en.wikipedia.org/wiki/Moving_Picture_Experts_Group) owns the
MPEG standards and their related patents.  Using these MPEG technologies
commercially requires paying royalties. That is why popular video streaming
platform like `YouTube` does not use them.  On the other hand, developers in
the open source community reverse-engineered and released the free tool `x264`
and `x265`, both are licensed under the terms of [GNU GPL v2.0 (or
later)](https://www.gnu.org/licenses/old-licenses/gpl-2.0.en.html), one of the
strongest copyleft licenses in the world.

The above video formats were developed specifically for their implementations
on physical media, like DVD or Blu-ray.  Microsoft published VC1, making it an
open but non-free format that was used on many Blu-ray discs in the earlier
days.

On the other hand, VP8 and VP9, released by Google as open and royalty-free
formats, which are analog to H.264 and H.265 in terms of efficiency, are two
of the three main formats used on YouTube after [it switched from H.264
following the call by Free Software Foundation in an open
letter](https://www.fsf.org/blogs/community/google-free-on2-vp8-for-youtube).
However, a Luxembourg-based company named Sisvel formed patent pools for VP9
(and AV1) laid claims on the patents that allegedly were used in VP9 and
AV1. [Some people suspect that Sisvel is a patent
troll.](https://www.mux.com/blog/did-sisvel-just-catch-aom-with-their-patents-down)

Alliance for Open Media (AOM), a non-profit organization that later released
the AV1 standard and its `libaom` codec under BSD 2-Clause License in 2018,
was formed to create royalty-free media standards to compete with MPEG.
Despite that the patent claims were made and patent licenses were sold, AV1
quickly gained popularity, soon Google enabled streaming AV1 coded contents on
YouTube, many hardware decoders were released by different vendors, and the
desktop GPU vendors Nvidia, AMD, and Intel released their desktop discrete
graphics cards with built-in AV1 hardware encode/decode acceleration, which is
anticipated to be a huge push to the wide adoption of live streaming in AV1
codec.

The members of AOM mainly consist of [multi-national
mega-corporations](https://en.wikipedia.org/wiki/Alliance_for_Open_Media#Operation_and_structure)
which are assumed that they want to maximize their profit by switching to open
media formats, but their efforts are met with patent claims by the companies
which are also assumed to maximize their profit. **Ironic.**

![Ironic](/assets/ironic.jpg)

*This is a still picture of the movie Star Wars: Revenge of the Sith (2005),
 where Chancellor Palpatine tallked to Anakin Skywalker at the senate. I do
 not claim its cpoyright.*

All the above video formats have complete or partial support from various
clients.  In terms of resource consumption when decoding/encoding these
formats, the newer ones are more resource intensive than the older ones.
Decoding should not be a problem on modern computers or phones.  Encoding, on
the other hand, consumes much more CPU resources when using software, less so
using GPU hardware acceleration, depending on the GPU vendor's
implementations.  Specifically, when encoding in AV1 using software, if the
highest efficiency option is used, a 30-second clip can take as long as 7
hours to encode, which is about hundreds of times slower than H.265.  However,
hardware encoding has its drawback. GPU is fast at doing repetitive work using
the same instructions with multiple data streams, but it trades off picture
quality and the flexibility of configurations for a fast encoding speed.
Software encoding which uses CPU, can offer flexibility and control over the
encoding process and have higher visual quality and reduction in artifacts.

### Audio formats and codecs

Audio on the other hand has a simpler story. Audio streams typically contain
less amount of data compared to video streams.

Pulse-code modulation (PCM), which was used in the days of telegraphy and
telephony before the digital age, is the standard that is most widely used.
Linear pulse-code modulation (LPCM) is a format used in audio files with the
extension `WAV` (Waveform Audio File Format).  The compact disc (CD) stores
stereo audio using 44.1 kHz sampling rate and 16-bit resolution audio. It is
also a part of DVD and Blu-ray standards.

However, (L)PCM is uncompressed, so the size of data is considered large and
uneconomic when being transported over network or physical media.

MP3 took over the world with its compact file size while maintaining
reasonable audio fidelity in the internet boom of the 1990s and
2000s. Unfortunately, it was soon associated with music copyright
infringement.

FLAC (Free Lossless Audio Codec), popular among enthusiasts, can compress
audio losslessly. YouTube uses Opus on the platform to reduce the bitrate
using lossy compression and maintain a certain level of audio quality.

DVD and Blu-ray audio mainly use Dolby Digital (AC3), DTS, or their
successors, like EAC3, Dolby TrueHD, Dolby Atmos, DTS-HD MA, DTS:X.

Audio encoding is not very resource intensive comparing to video encoding.

### Subtitles

For what matters in Jellyfin, the formats of subtitles can be roughly divided
into two categories: text-based and picture-based.

Text-based subtitles include the popular SubRip Text (SRT) and ASS/SSA. For
picture-based subtitles, VobSub is used on DVD, and PGSSUB is used on Blu-ray.

"Burned-in" subtitles, on the other hand, are permanently embedded into the
video, and they cannot be removed.  This is inconvenient for those who want to
turn off subtitles, thus separate subtitles are more popular, either
text-based or picture-based.

# Why do formats matter

Transcoding is the process to convert one format to another, and it consumes
more server resources than usual. [The goal is to Direct Play all
media.](https://jellyfin.org/docs/general/clients/codec-support) However,
being able to play the video file on one's PC without transcoding does not
mean it also plays well with streaming.  When the container, video, audio, and
subtitle are all compatible with the client, `Direct Play` will happen.  If
the audio, subtitle, or container is incompatible with the client, `Direct
Stream` will occur. Direct Stream is a process when only the video is directly
streamed without transcoding.  Audio may be transcoded to AAC format, or be
remuxed (remultiplexed) with video and subtitle to a new container TS, the
container used by DVD.  Transcoding happens when the video format is
incompatible with the client, or when the subtitle is "burned in".

Jellyfin has a [Codec
Table](https://jellyfin.org/docs/general/clients/codec-support) that lists the
codec compatibility in detail.  H.264 with 8-bit color depth is the most
supported video format across all platforms, and it is the format that the
media will be transcoded to by default if transcoding happens. Newer formats
like H.265 and AV1 are not well supported by clients.

As for audio, because transcoding audio is not nearly as resource-intensive,
it is not much of a concern.  Although FLAC is also supported on all clients
listed, Jellyfin transcodes incompatible audio formats to AAC by default.

Subtitles can lead to video transcoding too, if a picture-based subtitle track
is selected or the "force burn in all subtitles" option is enabled, the
subtitle track will be "burned in" to video.  This is the most
resource-intensive scenario because two transcodings happen at the same time,
while the subtitle layer is placed on the video.  PGS and Vob extracted from
Blu-ray and DVD are typically burned in when streaming.  This is the least
desirable situation, and users should choose SRT whenever available, but the
container should be mkv (Matroska, a free media container) for compatibility
if the user chooses not to place the SRT file as an [external
file](https://jellyfin.org/docs/general/server/media/external-files), so the
video, audio, and the SRT subtitle should be muxed (multiplexed) beforehand
using free (libre) tools like `mkvmerge` (CLI) and `MKVToolNix` (GUI).

Jellyfin uses a custom-built FFmpeg to transcode or remux. FFmpeg is a free
software, a collection of libraries and tools for processing media.  User can
supply their own FFmpeg binary by building from source code or using one of
the pre-built binaries from other developers, but this is not recommended by
Jellyfin, because Jellyfin requires certain build options and libraries to be
enabled for hardware acceleration.

# A note on hardware acceleration

Hardware acceleration is not very easy to [set
up](https://jellyfin.org/docs/general/administration/hardware-acceleration/). The
video quality and performance may not be satisfactory.  AMD GPUs are known to
have less satisfactory performance, and in my case, I can see massive chunks
of green artifacts when watching 4K HDR HEVC contents with HDR -> SDR tone
mapping enabled on Chrome, and the transcoding max out at 27 fps using AMD RX
6600XT.  Intel iGPUs may be better, but the latest Intel Arc dGPU has many
Linux driver issues as it does on Windows, as of mid 2023.

While Intel and AMD GPU can use the open source VAAPI, Nvidia GPU uses a
proprietary video codec API called NVENC/NVDEC, and Nvidia imposes an
artificial limitation of 3 or 5 simultaneous encoding sessions on their
consumer-grade graphics card, 3 or 5 sessions depending on the driver version
used.  This restriction can be bypassed using [an unofficial driver
patch](https://github.com/keylase/nvidia-patch).

GPU transcoding does not require much computational power as gaming does, so a
second-hand or mid-range graphics card with proper codec supports should be
fine if it is exclusively used by Jellyfin.
