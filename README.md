# mediascripts
My assorted media manipulation scripts

## What's in here

  - addstats
    - You'd think remembering "--add-track-statistics-tags" wouldn't be that hard but I got so tired of forgetting that I made a wrapper
  - audiovise
    - Just transcodes audio, does not touch video
    - It's a 30 Rock joke
  - av1meup
    - A poorly maintained mod of hevcmeup2 for av1 instead of hevc
  - cropfinder
    - Tells you what ffmpeg thinks the crop should be for an input file
  - framedumper
    - Dumps specific frames as PNGs. Uses math and magnets to dump one frame per even time interval based on the number of frames you asked for.
    - Useful for doing comparisons between two versions of a file if you don't want to cherry pick specific frames because you're lazy
  - gifify
    - Makes a gif from a video file based on timecodes
    - It's simple and dumb, you have to edit the framerate and the output width by hand because I hate you and myself
  - hevcmeup2
    - This is the franchise. An extremely overcomplicated script to cajole ffmpeg into transcoding media files to HEVC via libx265
    - More on this below
  - stripcovers
    - Sometimes there are covers in mkv files and while some people think that's cool, they're wrong, it's dumb
    - They're specifically dumb because it's difficult to reliably keep ffmpeg from trying to encode them as video which really chaps my ass
    - This removes all covers with extreme prejudice
  - striptags
    - Sometimes there are tags in mkv files with data you'd rather not have in them
    - Specifically global file title metadata confuses the Kodi media scanner
    - Some tools/people like to stuff notes into the video track title, they can die in a fire
  - thecleaner
    - I'm a 'Murican, a high percentage of my media is from 'Murica in native 'Murican and I don't need other language audio tracks or subtitles
    - This _very specifically_ keeps _only_ english language tagged audio and subtitle tracks
    - It can also optionally change the title tag of the primary audio track
      - I specifically transcode most audio to opus cause it's the best so that's mostly what it's good at making names for
  - toolchecker
    - So. You're probably asking yourself why everything requires the toolchecker.
      1. So tired of trying to use these scripts in places and getting rando errors cause the fuckin' tools aren't installed, now I get intentional errors
      2. So tired of running these scripts inside a screen session or some kind of background subshell situation and it not having my god damned environment and thus missing my path and then getting rando errors, now I get very intentional errors
      3. Literally _any excuse_ *EVER* to make use of variable variables for confusion value
  - tracktitleeditor
    - You can do this with mkvpropedit by hand like a big bad matroska gangster but I can never remember the damn syntax
    - It only works on mkv files, natch.
    - I did my very best to take whatever text you type and stuff it into the metadata tag but if you try to paste some emoji nonsense in here it's probably not gonna work and I do not give a single shit.


## hevcmeup2
### What even is this
hevcmeup2 is truly the shiniest turd jewel in this shiny turd crown. I transcode a lot of video files into hevc for space saving reasons, specifically into yuv420p10le hevc (yes even SDR files, I do not care about your thoughts on transcoding 8-bit data into 10-bit pixels until you've read all of doom9 and since that will kill any human being, the problem will solve itself). I got very, very tired of hand-massaging ffmpeg command lines so I built my own wrapper to solve my specific problems. It started out quite simple, back before it had a `2` in the name. Now it's a complete bloody nightmare of bash nonsense, just absolute fucking chaos of bashisms. If you can read this whole script and not need to look up a single convoluted bash tricknique, your service to the secret fire has clearly been both lengthy and arduous.
### How do I use it
Look, and I'm being real here. Probably don't. But if you decide to use it, it has help, specifically this
```bash
Usage: ${0##*/} -i <INPUT FILE> -o <OUTPUT FILE>

-h|--help:     show this help message
-i|--input:    file name for input
-o|--output:   file name for output
--start:       start time for encode (default: unset)
--duration:    duration of encode (default: unset)
--crf:         crf value (default varies with input res, 1080:18, 720: 22, 480: 26)
--melton:      use Don Melton's trickeration for bitrate control
--mbitrate:    if using --melton, use this bitrate cap
--preset:      x265 preset value (default: medium)
--tune:        x265 tune value (default: unset)
--crop:        enable crop detection (default: no)
--debug:       output commands to be run, run nothing
--oprofile:    output profile [1080, 720, 480]
--vf:          custom videofilter input
--sdr:         best effort 2020->709 color fix
--yadif:       deinterlace with yadif
--yadifmode:   set yadif to specific mode (default: 1, send frame per field)
--mcdeint:     deinterlace with yadif and mcdeint (DEPRECATED)
--nnedi:       use the power of ai to deinterlace
--ivtc:        Inverse telecine, aka pullup (assumes NTSC DVD 29.97->23.976)
--pullup:      Alias for --ivtc
--fieldmatch:  Inverse telecine a different way using fieldmatch filter
--bitdepth:    Bitdepth for pixel format (8, 10 or 12, default: 10)
--audio:       transcode audio tracks to selected codec [flac, opus, aac]
--square:      convert anamorphic to square pixels which make sense
```

(seriously, if `${0##*/}` makes sense to you, honest to god you can lay your burden down and go into the west and remain <your name here>, you've done enough)

When I run it, which is literally constantly, this is my CPU time, this is typically what it looks like:
```bash
$ hevcmeup2 -i input_file.mkv -o output_file.mkv --crop --audio opus
```
See how easy it is? What, you're thinking "Gosh there must be a container ship full of defaults and nonsense to make that work"? Well you, friend, win the prize, that is exactly right, it's for me and it does what I want with minimal effort. Most every default is listed above in the usage, they're pretty self-explanatory.

### Confusing options explained
  - `--melton` and `--mbitrate`
    - is an implementation of a tricksy rate control hack as described here: ![video_transcode](https://github.com/donmelton/video_transcoding#how-my-simple-and-special-ratecontrol-systems-work) If you need to hit an ABR target for specific reasons, it's probably better than ABR, bit it's still worse than surfing CRFs to find one that makes an output file the right size. I'm not sayin' it's easy or fast to surf crfs but for two files of the same average bitrate, one using actual crf rate control will look better every single time. But I wanted to try it and see, so I tried it and saw and now it's here forever.

  - `--oprofile`
    - this was an idea I had that was ostensibly going to let me dial in defaults for specific output profiles. I started with profiles based on video dimensions, that's all I ever made, and all they actually do is set a scale filter and change the CRF to be appropriate for that video dimension. So, if you've got a 1080p file and you want it to be a 720p output, `--oprofile 720` will set up the scale and lower the CRF automatically.
  - `--square`
    - Boy howdy do anamorphic pixels blow. I'm not gonna explain them, but they make everything worse, they're a tired holdover from the analog era and I wish they were never made part of the DVD standard. But sadly they were, and sometimes you're trying to take content from a DVD that has dumbass anamorphic pixels and you'd really just like it to have a normal sqaure pixel resolution instead, great news! This does that.
  - `--mcdeint`
    - Prior to ffmpeg5, mcdeint (motion compensating deinterlacer) was, in my opinion, the hands-down best quality de-interlacer in ffmpeg. It was unbelievably, unbearably slow, but the output was superb. It's been deprected in ffmpeg5, so now, when I'm feeling very picky about deinterlacing, I use `--nnedi` which is also unbleliavably, unbearably slow, but it uses the power of _machine learning_ so you know it's good. Also it actually is good, it makes very good output, maybe even better than mcdeint.
  - `--audio`
    - This will also transcode the audio tracks. I like opus, I think it's the least doofy lossy codec, it also has the benefit of being unencumbered, so I transcode a lot of lossless audio to opus for space reasons. I _do not_ transcode _lossy_ audio, if I can help it. Also I try not to transcode between lossless formats cause why fucking bother. This section of the script tries its best to follow those rules. It chooses the opus bitrate based on the channel layout presented, if it understands the channel layout, or by the channel count if it doesn't. Based on the opus guidelines for bitrate I'm wildly exceeding the recommended bitrates at every channel count but we're talking about kilobits here and who fucking cares, it's still smaller than OG DTS. I would not call what this option does entirely intuitive, so if it does something to your audio you didn't expect, rest assured I probably did expect it so that's why it did that.
